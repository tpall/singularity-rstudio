BootStrap: docker
From: debian:stretch

%labels
  Maintainer tpall
  RStudio_Version 1.2.1256

%help
  This will run RStudio Server

%apprun rserver
  exec rserver "${@}"

%runscript
  exec rserver "${@}"

%environment
  export PATH=/usr/lib/rstudio-server/bin:${PATH}
  LC_ALL=en_US.UTF-8
  LANG=en_US.UTF-8
  TERM=xterm
  export LC_ALL LANG TERM

%setup
  install -Dv \
    rstudio_auth.sh \
    ${SINGULARITY_ROOTFS}/usr/lib/rstudio-server/bin/rstudio_auth
  install -Dv \
    ldap_auth.py \
    ${SINGULARITY_ROOTFS}/usr/lib/rstudio-server/bin/ldap_auth

%post
  # Software versions
  export RSTUDIO_VERSION=${RSTUDIO_VERSION:-1.2.1256}
  export R_VERSION=${R_VERSION:-3.5.3}
  
  # Get dependencies
  apt-get update \
  && apt-get install -y --no-install-recommends \
    locales

  # Configure default locale
  echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
  locale-gen en_US.utf8
  /usr/sbin/update-locale LANG=en_US.UTF-8
  export LC_ALL=en_US.UTF-8
  export LANG=en_US.UTF-8
  
  # Configure term
  export TERM=xterm

  # Install R
  apt-get update \
  && apt-get install -y --no-install-recommends \
    bash-completion \
    ca-certificates \
    file \
    fonts-texgyre \
    g++ \
    gfortran \
    gsfonts \
    libblas-dev \
    libbz2-1.0 \
    libcurl3 \
    libicu57 \
    libjpeg62-turbo \
    libopenblas-dev \
    libpangocairo-1.0-0 \
    libpcre3 \
    libpng16-16 \
    libreadline7 \
    libtiff5 \
    liblzma5 \
    make \
    unzip \
    zip \
    zlib1g \
  && BUILDDEPS="curl \
    default-jdk \
    libbz2-dev \
    libcairo2-dev \
    libcurl4-openssl-dev \
    libpango1.0-dev \
    libjpeg-dev \
    libicu-dev \
    libpcre3-dev \
    libpng-dev \
    libreadline-dev \
    libtiff5-dev \
    liblzma-dev \
    libx11-dev \
    libxt-dev \
    unixodbc-dev
    perl \
    tcl8.6-dev \
    tk8.6-dev \
    texinfo \
    texlive-extra-utils \
    texlive-fonts-recommended \
    texlive-fonts-extra \
    texlive-latex-recommended \
    x11proto-core-dev \
    xauth \
    xfonts-base \
    xvfb \
    zlib1g-dev" \
  && apt-get install -y --no-install-recommends $BUILDDEPS \
  && cd tmp/
  
  ## Download source code
  curl -O https://cran.r-project.org/src/base/R-3/R-${R_VERSION}.tar.gz
  
  ## Extract source code
  tar -xf R-${R_VERSION}.tar.gz \
  && cd R-${R_VERSION}
  
  ## Set compiler flags
  R_PAPERSIZE=letter \
    R_BATCHSAVE="--no-save --no-restore" \
    R_BROWSER=xdg-open \
    PAGER=/usr/bin/pager \
    PERL=/usr/bin/perl \
    R_UNZIPCMD=/usr/bin/unzip \
    R_ZIPCMD=/usr/bin/zip \
    R_PRINTCMD=/usr/bin/lpr \
    LIBnn=lib \
    AWK=/usr/bin/awk \
    CFLAGS="-g -O2 -fstack-protector-strong -Wformat -Werror=format-security -Wdate-time -D_FORTIFY_SOURCE=2 -g" \
    CXXFLAGS="-g -O2 -fstack-protector-strong -Wformat -Werror=format-security -Wdate-time -D_FORTIFY_SOURCE=2 -g"
  
  ## Configure options
  ./configure --enable-R-shlib \
               --enable-memory-profiling \
               --with-readline \
               --with-blas \
               --with-tcltk \
               --disable-nls \
               --with-recommended-packages
  
  ## Build and install
  make \
  && make install
  
  ## Add a library directory (for user-installed packages)
  mkdir -p /usr/local/lib/R/site-library
  
  ## Set site library path
  echo "R_LIBS_SITE='/usr/local/lib/R/site-library'" >> /usr/local/lib/R/etc/Renviron
  
  ## Install packages from date-locked MRAN snapshot of CRAN
  [ -z "$BUILD_DATE" ] && BUILD_DATE=$(TZ="America/Los_Angeles" date -I) || true \
  && MRAN=https://mran.microsoft.com/snapshot/${BUILD_DATE} \
  && echo MRAN=$MRAN >> /etc/environment \
  && export MRAN=$MRAN \
  && echo "options(repos = c(CRAN = '$MRAN'), download.file.method = 'libcurl')" >> /usr/local/lib/R/etc/Rprofile.site
  
  ## Download and install RStudio server & dependencies
  ## Attempts to get detect latest version, otherwise falls back to version given in $VER
  apt-get update \
  && apt-get install -y --no-install-recommends \
    file \
    git \
    libapparmor1 \
    libcurl4-openssl-dev \
    libedit2 \
    libssl-dev \
    libclang-dev \
    lsb-release \
    psmisc \
    procps \
    python-setuptools \
    sudo \
    wget \
  && wget -O libssl1.0.2.deb http://ftp.br.debian.org/debian/pool/main/o/openssl1.0/libssl1.0.2_1.0.2l-2+deb9u3_amd64.deb \
  && dpkg -i libssl1.0.2.deb \
  && rm libssl1.0.2.deb \
  && wget -q https://s3.amazonaws.com/rstudio-ide-build/server/debian9/x86_64/rstudio-server-${RSTUDIO_VERSION}-amd64.deb \
  && dpkg -i rstudio-server-${RSTUDIO_VERSION}-amd64.deb \
  && rm rstudio-server-*-amd64.deb
  
  ## Clean up from R source install
  cd / \
  && rm -rf /tmp/* \
  && apt-get remove --purge -y $BUILDDEPS \
  && apt-get autoremove -y \
  && apt-get autoclean -y \
  && rm -rf /var/lib/apt/lists/*
  
  ## Add support for LDAP authentication
  wget -q https://bootstrap.pypa.io/get-pip.py \
  && python3 get-pip.py \
  && rm get-pip.py \
  && pip3 install ldap3
