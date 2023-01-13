FROM ubuntu:focal

# avoid stuck build due to user prompt
ENV DEBIAN_FRONTEND noninteractive

ARG VERILOG_ENV="oss-cad-suite-linux-x64-20230113.tgz"

COPY ${VERILOG_ENV} /verilog-setup/

RUN \
    apt-get update -y && \
    apt-get upgrade -y && \
    apt-get -y --no-install-recommends install \
        ca-certificates curl sudo && \
    apt-get autoclean && \
    apt-get autoremove && \
    echo "%sudo ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

RUN \
  dpkg --add-architecture i386 && \
  apt-get update && \
  apt-get -y --no-install-recommends install \
    build-essential git gcc-multilib libc6-dev:i386 ocl-icd-opencl-dev libjpeg62-dev && \
  apt-get -y -f install && \
  apt-get autoclean && \
  apt-get autoremove && \
  rm -rf /var/lib/apt/lists/*

RUN \
    apt-get update && \
    apt-get -y --no-install-recommends install \
        vim tmux htop make g++

RUN \
    tar -zxf /verilog-setup/${VERILOG_ENV} -C /verilog-setup && \
    rm -rf /verilog-setup/${VERILOG_ENV}

RUN \
    mkdir -p touch /etc/udev/rules.d/ && \
    touch /etc/udev/rules.d/50-lattice-ftdi.rules && \
    echo "ACTION==\"add\", ATTR{idVendor}==\"0403\", ATTR{idProduct}==\"6010\", MODE:=\"666\"" > /etc/udev/rules.d/50-lattice-ftdi.rules

WORKDIR /app

ENV PATH="/verilog-setup/oss-cad-suite/bin:$PATH"

# Creates a non-root user with an explicit UID and adds permission to access the /app folder
# For more info, please refer to https://aka.ms/vscode-docker-python-configure-containers
RUN adduser -u 1000 --disabled-password --gecos "" appuser && chown -R appuser /app/
USER appuser

CMD ["/bin/bash", "-c", "cd /app; sleep infinity"]