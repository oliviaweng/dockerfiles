FROM centos:centos7 AS hls4ml-test-base
MAINTAINER Vladimir Loncar
RUN yum groupinstall -y "X Window System" "Development Tools" && \
    yum install -y sudo nano mc wget curl grep sed && \
    # TINI_VERSION=`curl https://github.com/krallin/tini/releases/latest | grep -o "/v.*\"" | sed 's:^..\(.*\).$:\1:'` && \
    curl -L "https://github.com/krallin/tini/releases/download/v0.19.0/tini_0.19.0.rpm" > tini.rpm && \
    rpm -i tini.rpm && \
    rm tini.rpm
# Setup default user
ARG USER_ID=1000
ARG GROUP_ID=1000
RUN groupadd -g ${GROUP_ID} hls4ml && \
    useradd -u ${USER_ID} -g ${GROUP_ID} -m hls4ml && \
    echo "hls4ml ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/hls4ml && \
    chmod 0440 /etc/sudoers.d/hls4ml
USER hls4ml
ENV HOME /home/hls4ml
WORKDIR /home/hls4ml
RUN cp /etc/skel/.bashrc .bashrc 
COPY environment.yml /tmp/environment.yml
RUN wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda.sh && \
    bash ~/miniconda.sh -b -p $HOME/miniconda && \
    rm ~/miniconda.sh && \
    sudo ln -s $HOME/miniconda/etc/profile.d/conda.sh /etc/profile.d/conda.sh && \
    cp /etc/skel/.bashrc .bashrc && \
    echo ". $HOME/miniconda/etc/profile.d/conda.sh" >> ~/.bashrc && \
    echo "conda activate base" >> ~/.bashrc && \
    eval "$($HOME/miniconda/bin/conda shell.bash hook)" && \
    conda init && \
    conda env create -f /tmp/environment.yml
ENTRYPOINT [ "/usr/bin/tini", "--" ]
CMD [ "/bin/bash" ]

FROM hls4ml-test-base AS hls4ml-test-vivado-2019.2
ENV XILINXD_LICENSE_FILE 2112@licenxilinx
ENV LM_LICENSE_FILE 2112@licenxilinx
USER root
COPY vivado_cfg.txt /tmp/vivado_cfg.txt
# RUN wget http://188.185.125.174:8000/vivado.tar.gz > /dev/null 2&>1 && \
COPY Xilinx_Vivado_2019.2_1106_2127.tar.gz vivado.tar.gz
RUN tar -xzf vivado.tar.gz && \
    cd Xilinx_Vivado_2019.2_1106_2127 && \
    ./xsetup --agree XilinxEULA,3rdPartyEULA,WebTalkTerms --batch Install --config /tmp/vivado_cfg.txt && \
    cd .. && \
    rm -r Xilinx_Vivado_2019.2_1106_2127 && \
    rm vivado.tar.gz && \
    rm /tmp/vivado_cfg.txt
USER hls4ml
