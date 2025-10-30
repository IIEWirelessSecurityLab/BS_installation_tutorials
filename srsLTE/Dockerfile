FROM ubuntu:22.04 AS base
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ="Asia/Singapore"

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# General utilities
RUN apt update && apt -y upgrade
RUN apt-get install -y build-essential git cmake iproute2 libfftw3-dev libmbedtls-dev libboost-program-options-dev libconfig++-dev libsctp-dev

FROM base AS uhd

RUN apt-get update && apt-get install -y software-properties-common
RUN add-apt-repository ppa:ettusresearch/uhd &&  apt-get install -y libuhd-dev uhd-host
RUN uhd_images_downloader


FROM uhd AS srslte
RUN git config --global http.proxy "http://10.42.0.1:7897"
RUN git config --global https.proxy "https://10.42.0.1:7897"

RUN git clone https://github.com/srsRAN/srsRAN_4G.git /root/srsRAN_4G
WORKDIR /root/srsRAN_4G
RUN mkdir build && \
    cd build && \
    cmake ../ && \
    make -j$(nproc) && \
    make install && \
    srsran_install_configs.sh user

RUN ldconfig

RUN apt install -y iptables
