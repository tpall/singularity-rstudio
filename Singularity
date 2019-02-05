BootStrap: shub
From: tpall/singularity-r@78ab1978abb2755fb253be6bd2d19b4d2446cdd3

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
  && wget -O libssl1.0.0.deb http://ftp.debian.org/debian/pool/main/o/openssl/libssl1.0.0_1.0.1t-1+deb8u8_amd64.deb \
  && dpkg -i libssl1.0.0.deb \
  && rm libssl1.0.0.deb \
  && wget -q https://s3.amazonaws.com/rstudio-ide-build/server/debian9/x86_64/rstudio-server-${RSTUDIO_VERSION}-amd64.deb \
  && dpkg -i rstudio-server-${RSTUDIO_VERSION}-amd64.deb \
  && rm rstudio-server-*-amd64.deb
  
  ## Clean up
  apt-get clean \
  && rm -rf /var/lib/apt/lists/
  
  ## Add support for LDAP authentication
  wget -q https://bootstrap.pypa.io/get-pip.py \
  && python3 get-pip.py \
  && rm get-pip.py \
  && pip3 install --upgrade pip \
  && pip3 install ldap3
