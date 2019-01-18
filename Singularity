BootStrap: shub
From: nickjer/singularity-r:3.5.1

%labels
  Maintainer tpall
  RStudio_Version 1.2.1226

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
  export RSTUDIO_VERSION=1.2.1226

  # Install RStudio Server
  apt-get update
  apt-get install -y \
    ca-certificates \
    wget \
    gdebi-core \
    git
  wget \
    --no-verbose \
    -O rstudio-server.deb \
    "https://s3.amazonaws.com/rstudio-ide-build/server/trusty/amd64/rstudio-server-${RSTUDIO_VERSION}-amd64.deb"
  gdebi -n rstudio-server.deb
  rm -f rstudio-server.deb
  
  # Install libgfortran.so.4 for data.table 
  add-apt-repository ppa:jonathonf/gcc-7.1
  apt-get update
  apt-get install -y \
    gcc-7 \
    g++-7 \
    gfortran-7
  
  # Add support for LDAP authentication
  wget \
    --no-verbose \
    -O get-pip.py \
    "https://bootstrap.pypa.io/get-pip.py"
  python3 get-pip.py
  rm -f get-pip.py
  pip3 install ldap3

  # Clean up
  rm -rf /var/lib/apt/lists/*
