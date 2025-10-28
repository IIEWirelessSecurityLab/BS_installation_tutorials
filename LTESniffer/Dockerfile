FROM ubuntu:22.04 AS base
ENV DEBIAN_FRONTEND=noninteractive
ENV TZ="Asia/Singapore"

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# General utilities
RUN apt update && apt -y upgrade
RUN apt-get install -y build-essential git cmake libfftw3-dev libmbedtls-dev libboost-program-options-dev libconfig++-dev libsctp-dev

FROM base AS uhd

# RUN git config --global http.proxy "http://10.42.0.1:7897"
# RUN git config --global https.proxy "https://10.42.0.1:7897"

RUN apt-get update && apt-get install -y software-properties-common
RUN add-apt-repository ppa:ettusresearch/uhd &&  apt-get install -y libuhd-dev uhd-host
RUN uhd_images_downloader

FROM uhd AS ltesniffer
RUN apt-get install -y libglib2.0-dev libudev-dev libcurl4-gnutls-dev libboost-all-dev qtdeclarative5-dev libqt5charts5-dev

RUN git clone https://github.com/SysSec-KAIST/LTESniffer.git /root/LTESniffer
WORKDIR /root/LTESniffer
RUN sed -i '205s|//||' /root/LTESniffer/src/src/SubframeWorker.cc
RUN mkdir build && \
    cd build && \
    cmake ../ && \
    make -j$(nproc) 

RUN ln -s /root/LTESniffer/build/src/LTESniffer /usr/bin/LTESniffer