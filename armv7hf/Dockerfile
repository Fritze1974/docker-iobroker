FROM debian:stretch

LABEL maintainer="Andre Germann" \ 
      url="<https://buanet.de>"

ENV DEBIAN_FRONTEND noninteractive

# Install prerequisites (as listed in iobroker installer.sh)
RUN apt-get update && apt-get install -y \
    acl \
    apt-utils \
    build-essential \
    curl \
    git \
    gnupg2 \
    gosu \
    libavahi-compat-libdnssd-dev \
    libcap2-bin \
    libpam0g-dev \
    libudev-dev \
    locales \
    nano \
    pkg-config \
    procps \
    python \
    python-dev \
    sudo \
    udev \
    unzip \
    wget \
    && rm -rf /var/lib/apt/lists/*

# Install node10
RUN curl -sL https://deb.nodesource.com/setup_10.x | bash \
    && apt-get update && apt-get install -y nodejs \
    && rm -rf /var/lib/apt/lists/*

# Generating locales
RUN sed -i 's/^# *\(de_DE.UTF-8\)/\1/' /etc/locale.gen \
    && sed -i 's/^# *\(en_US.UTF-8\)/\1/' /etc/locale.gen \
    && locale-gen

# Install openzwave
RUN cd /tmp \
	&& git clone https://github.com/OpenZWave/open-zwave.git \
	&& cd /tmp/open-zwave \
	&& make install \
	&& ln -s /usr/local/lib64/libopenzwave.so /usr/local/lib/libopenzwave.so \
	&& ln -s /usr/local/lib64/libopenzwave.so.1.6 /usr/local/lib/libopenzwave.so.1.6 \
	&& ldconfig

# Install Redis Client

RUN npm install -h redis-cli

# Create scripts directory and copy scripts
RUN mkdir -p /opt/scripts/ \
    && chmod 777 /opt/scripts/
WORKDIR /opt/scripts/
COPY scripts/iobroker_startup.sh iobroker_startup.sh
COPY scripts/setup_avahi.sh setup_avahi.sh
COPY scripts/setup_packages.sh setup_packages.sh
COPY scripts/setup_zwave.sh setup_zwave.sh
RUN chmod +x iobroker_startup.sh \
    && chmod +x setup_avahi.sh \
    && chmod +x setup_packages.sh

# Install ioBroker
WORKDIR /
RUN apt-get update \
    && curl -sL https://raw.githubusercontent.com/ioBroker/ioBroker/stable-installer/installer.sh | bash - \
    && echo $(hostname) > /opt/iobroker/.install_host \
    && echo $(hostname) > /opt/.firstrun \
    && rm -rf /var/lib/apt/lists/*

# Install node-gyp
WORKDIR /opt/iobroker/
RUN npm install -g node-gyp

# Backup initial ioBroker-folder
RUN tar -cf /opt/initial_iobroker.tar /opt/iobroker

# Setting up iobroker-user (shell and home directory)
RUN chsh -s /bin/bash iobroker \
    && usermod --home /opt/iobroker iobroker

# Setting up ENVs
ENV ADMINPORT=8081 \
    AVAHI="false" \
    DEBIAN_FRONTEND="teletype" \
    LANG="de_DE.UTF-8" \
    LANGUAGE="de_DE:de" \
    LC_ALL="de_DE.UTF-8" \
    PACKAGES="vi" \
    REDIS="false" \
    SETGID=1000 \
    SETUID=1000 \
    TZ="Europe/Berlin" \
    USBDEVICES="none" \
    ZWAVE="true"
	
# Run startup-script
ENTRYPOINT ["/opt/scripts/iobroker_startup.sh"]
