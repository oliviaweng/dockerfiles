FROM ubuntu:bionic

# avoid stuck build due to user prompt
ENV DEBIAN_FRONTEND noninteractive

ARG INSTALL_FILE="Xilinx_Unified_2020.1_0602_1208.tar.gz"

RUN \
  sed -i -e "s%http://[^ ]\+%http://ftp.jaist.ac.jp/pub/Linux/ubuntu/%g" /etc/apt/sources.list && \
  apt-get update -y && \
  apt-get upgrade -y && \
  apt-get -y --no-install-recommends install \
    ca-certificates curl sudo xorg dbus dbus-x11 ubuntu-gnome-default-settings gtk2-engines \
    ttf-ubuntu-font-family fonts-ubuntu-font-family-console fonts-droid-fallback lxappearance && \
  apt-get autoclean && \
  apt-get autoremove && \
  rm -rf /var/lib/apt/lists/* && \
  echo "%sudo ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

ARG GOSU_VERSION=1.14
RUN set -eux; \
# save list of currently installed packages for later so we can clean up
	savedAptMark="$(apt-mark showmanual)"; \
	apt-get update; \
	apt-get install -y --no-install-recommends ca-certificates wget; \
	if ! command -v gpg; then \
		apt-get install -y --no-install-recommends gnupg2 dirmngr; \
	elif gpg --version | grep -q '^gpg (GnuPG) 1\.'; then \
# "This package provides support for HKPS keyservers." (GnuPG 1.x only)
		apt-get install -y --no-install-recommends gnupg-curl; \
	fi; \
	rm -rf /var/lib/apt/lists/*; \
	\
	dpkgArch="$(dpkg --print-architecture | awk -F- '{ print $NF }')"; \
	wget -O /usr/local/bin/gosu "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch"; \
	wget -O /usr/local/bin/gosu.asc "https://github.com/tianon/gosu/releases/download/$GOSU_VERSION/gosu-$dpkgArch.asc"; \
	\
# verify the signature
	export GNUPGHOME="$(mktemp -d)"; \
	gpg --batch --keyserver hkps://keys.openpgp.org --recv-keys B42F6819007F00F88E364FD4036A9C25BF357DD4; \
	gpg --batch --verify /usr/local/bin/gosu.asc /usr/local/bin/gosu; \
	command -v gpgconf && gpgconf --kill all || :; \
	rm -rf "$GNUPGHOME" /usr/local/bin/gosu.asc; \
	\
# clean up fetch dependencies
	apt-mark auto '.*' > /dev/null; \
	[ -z "$savedAptMark" ] || apt-mark manual $savedAptMark; \
	apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false; \
	\
	chmod +x /usr/local/bin/gosu; \
# verify that the binary works
	gosu nobody true

# vidao
RUN \
  dpkg --add-architecture i386 && \
  apt-get update && \
  apt-get -y --no-install-recommends install \
    build-essential git gcc-multilib libc6-dev:i386 ocl-icd-opencl-dev libjpeg62-dev && \
  apt-get -y -f install && \
  apt-get -y install python && \
  apt-get autoclean && \
  apt-get autoremove && \
  rm -rf /var/lib/apt/lists/*

COPY install_config.txt /vivado-installer/
COPY ${INSTALL_FILE} /vivado-installer/

RUN \
  cat /vivado-installer/${INSTALL_FILE} | tar zx --strip-components=1 -C /vivado-installer && \
  /vivado-installer/xsetup \
    --agree 3rdPartyEULA,WebTalkTerms,XilinxEULA \
    --batch Install \
    --config /vivado-installer/install_config.txt && \
  rm -rf /vivado-installer

# install python3.9 & other dependencies
# Based on: https://luis-sena.medium.com/creating-the-perfect-python-dockerfile-51bdec41f1c8
RUN \
  apt-get update && apt-get -y install \
  graphviz vim tmux software-properties-common && \
  add-apt-repository -y ppa:deadsnakes/ppa && \
  apt-get update && apt-get -y --no-install-recommends install \
  python3.9 python3.9-venv python3-pip python3-wheel build-essential && \
  apt-get clean && rm -rf /var/lib/apt/lists/*


# COPY entrypoint.sh /usr/local/bin/entrypoint.sh
# RUN chmod +x /usr/local/bin/entrypoint.sh
# ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]

WORKDIR /app
# Creates a non-root user with an explicit UID and adds permission to access the /app folder
# For more info, please refer to https://aka.ms/vscode-docker-python-configure-containers
RUN adduser -u 1043 --disabled-password --gecos "" appuser && chown -R appuser /app/
USER appuser

# create and activate virtual environment as appuser
# using final folder name to avoid path issues with packages
ENV VIRTUAL_ENV=/app/venv
RUN python3.9 -m pip install --user --upgrade pip
RUN python3.9 -m pip install --user virtualenv
RUN python3.9 -m venv $VIRTUAL_ENV
ENV PATH="$VIRTUAL_ENV/bin:$PATH"

COPY requirements.txt /app/requirements.txt
RUN python3 -m pip install --no-cache-dir wheel
RUN python3 -m pip install -r /app/requirements.txt

# make sure all messages always reach console
ENV PYTHONUNBUFFERED=1

ENV PATH="/tools/Xilinx/Vivado/2020.1/bin:$PATH"

CMD ["/bin/bash", "-c", "source /tools/Xilinx/Vivado/2020.1/settings64.sh; cd /app; sleep infinity"]
