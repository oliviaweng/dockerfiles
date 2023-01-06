# Docker environment for hls4ml + Vivado HLS 2020.1

## Setup

1. First, download the [VivadoHLx 2020.1: All OS installer Single-File Download](https://www.xilinx.com/member/forms/download/xef.html?filename=Xilinx_Unified_2020.1_0602_1208.tar.gz) into a clone of this repo.
2. Run `./docker-build.sh` in the directory containing your clone of hls4ml---this Docker image is for developing hls4ml. This will take several minutes because the docker image is ~109GB. Make sure you have enough space in `/var/lib/docker`. This script will automatically spin up a container of this image. To manually spin up an image yourself, run `docker-compose up -d`.
3. Gain access to the container by running `docker exec -it hls4ml bash`. `hls4ml` is the name of the container, which we defined in `docker-compose.yml`. You may choose to change it if you like. 
4. Once in the docker container, run `./install-hls4ml.sh` and `./install-qkeras.sh` to install an editable version of hls4ml and to get the right version of qkeras.
