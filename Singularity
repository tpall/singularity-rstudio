BootStrap: shub
From: tpall/singularity-r@78ab1978abb2755fb253be6bd2d19b4d2446cdd3

%labels
  Maintainer tpall
  RStudio_Version 1.2.1237

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
  export S6_VERSION=${S6_VERSION:-v1.21.7.0}
  export S6_BEHAVIOUR_IF_STAGE2_FAILS=2
  export PATH=/usr/lib/rstudio-server/bin:$PATH
  
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
  
  ## RStudio wants an /etc/R, will populate from $R_HOME/etc
  mkdir -p /etc/R
  
  ## Write config files in $R_HOME/etc
  echo '\n\
    \n# Configure httr to perform out-of-band authentication if HTTR_LOCALHOST \
    \n# is not set since a redirect to localhost may not work depending upon \
    \n# where this container is running. \
    \nif(is.na(Sys.getenv("HTTR_LOCALHOST", unset=NA))) { \
    \n  options(httr_oob_default = TRUE) \
    \n}' >> /usr/local/lib/R/etc/Rprofile.site \
  && echo "PATH=${PATH}" >> /usr/local/lib/R/etc/Renviron
  
  ## Prevent rstudio from deciding to use /usr/bin/R if a user apt-get installs a package
  echo 'rsession-which-r=/usr/local/bin/R' >> /etc/rstudio/rserver.conf
  
  ## Use more robust file locking to avoid errors when using shared volumes:
  echo 'lock-type=advisory' >> /etc/rstudio/file-locks
  
  ## Configure git not to request password each time
  git config --system credential.helper 'cache --timeout=3600' \
  && git config --system push.default simple
  
  ## Set up S6 init system
  wget -P /tmp/ https://github.com/just-containers/s6-overlay/releases/download/${S6_VERSION}/s6-overlay-amd64.tar.gz \
  && tar xzf /tmp/s6-overlay-amd64.tar.gz -C / \
  && mkdir -p /etc/services.d/rstudio \
  && echo '#!/usr/bin/with-contenv bash \
          \n## load /etc/environment vars first: \
  		  \n for line in $( cat /etc/environment ) ; do export $line ; done \
          \n exec /usr/lib/rstudio-server/bin/rserver --server-daemonize 0' \
          > /etc/services.d/rstudio/run \
  && echo '#!/bin/bash \
          \n rstudio-server stop' \
          > /etc/services.d/rstudio/finish \
  && mkdir -p /home/rstudio/.rstudio/monitored/user-settings \
  && echo 'alwaysSaveHistory="0" \
          \nloadRData="0" \
          \nsaveAction="0"' \
          > /home/rstudio/.rstudio/monitored/user-settings/user-settings \
  && chown -R rstudio:rstudio /home/rstudio/.rstudio
  
  ## Add support for LDAP authentication
  wget \
    --no-verbose \
    -O get-pip.py \
    "https://bootstrap.pypa.io/get-pip.py"
  python3 get-pip.py
  rm -f get-pip.py
  pip3 install ldap3
