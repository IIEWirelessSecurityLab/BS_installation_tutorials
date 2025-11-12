# BS_installation_tutorials

## Table of Contents

- [Install docker](#install-docker)
- [DCI recoder](#dci-recoder)
  - [LTESniffer](#ltesniffer)
  - [sni5gect](#sni5gect)
- [Core Network and Base Station](#core-network-and-base-station)
  - [srsRAN_4G](#srsran_4g)
  - [open5gs](#open5gs)
- [Other](#other)
  - [Some issues](#some-issues)
  - [INSTALL usrp driver](#install-usrp-driver)
## [Install docker](https://docs.docker.com/engine/install/ubuntu/)

```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

## DCI recoder
### [LTESniffer](https://github.com/SysSec-KAIST/LTESniffer)

1. Install LTESniffer from docker:
```bash
cd LTESniffer
docker compose build ltesniffer
docker compose up -d ltesniffer
docker exec -it ltesniffer bash
```

2. Start sniffing...
```bash
LTESniffer -A 1 -W 8 -f 1870e6 -C -m 0 -a "num_recv_frames=512" -D record.csv
```

### [sni5gect](https://github.com/asset-group/Sni5Gect-5GNR-sniffing-and-exploitation)
```bash
wget https://zenodo.org/records/15601773/files/sni5gect-artifacts-docker.tar.gz
docker load < sni5gect-artifacts-docker.tar.gz

wget https://zenodo.org/records/15601773/files/sni5gect-evaluation-results.zip
unzip sni5gect-evaluation-results.zip


wget https://zenodo.org/records/15601773/files/Sni5Gect-source-code.zip
unzip Sni5Gect-source-code.zip
cd Sni5Gect-5GNR-sniffing-and-exploitation-main

docker compose up -d
docker exec -it artifacts bash


cd /root/sni5gect
vi configs/config-srsran-n78-20MHz.conf

./build/shadower/shadower configs/config-srsran-n78-20MHz.conf
```

## Core Network and Base Station
## [srsRAN_4G]()

1. Configure srsRAN_4G
```bash
cd srsLTE/config

######### gedit enb.conf ###########
# mcc = 001
# mnc = 01
# n_prb = 50
# dl_earfcn = 3350
# tx_gain = 80
# rx_gain = 40


######### gedit epc.conf ###########
# mcc = 001
# mnc = 01
# apn = srsapn

######### gedit user_db.csv ###########
# Register UE:
# ue2,mil,001010123456780,00112233445566778899aabbccddeeff,opc,63bfa50ee6523365ff14c1f45f88737d,8000,000000001234,7,dynamic
```


2. INSTALL srsRAN_4G


```bash
cd srsLTE
sysctl -w net.ipv4.ip_forward=1
docker compose build srslte
docker compose up -d srslte
```

3. Start BS
```bash
docker exec -it srslte bash
# 4G Core Network
srsepc

# start new bash
docker exec -it srslte bash
# srsRAN eNB using SDR (OTA)
srsenb
```

## [open5gs](https://github.com/herlesupreeth/docker_open5gs.git)
1. INSTALL open5gs
```bash
docker pull ghcr.io/herlesupreeth/docker_open5gs:master
docker tag ghcr.io/herlesupreeth/docker_open5gs:master docker_open5gs

docker pull ghcr.io/herlesupreeth/docker_grafana:master
docker tag ghcr.io/herlesupreeth/docker_grafana:master docker_grafana

docker pull ghcr.io/herlesupreeth/docker_metrics:master
docker tag ghcr.io/herlesupreeth/docker_metrics:master docker_metrics
```

2. INSTALL srsgnb
```bash
docker pull ghcr.io/herlesupreeth/docker_srsran:master
docker tag ghcr.io/herlesupreeth/docker_srsran:master docker_srsran
```

3. INSTALL docker_open5gs
```bash
git clone https://github.com/herlesupreeth/docker_open5gs.git
cd docker_open5gs
set -a
source .env
set +a
sudo ufw disable
sudo sysctl -w net.ipv4.ip_forward=1
sudo cpupower frequency-set -g performance
gedit .env
# MCC=001
# MNC=01
# DOCKER_HOST_IP=YOUR_HOST_IP_ADDR
cd srsran
gedit gnb.yaml
# device_args: type=b200,num_recv_frames=64,num_send_frames=64
# dl_arfcn: 632628   
# band: 78         
# channel_bandwidth_MHz: 20  
# common_scs: 30      
```
4. RUN
```bash
# 5G Core Network
docker compose -f sa-deploy.yaml up

# srsRAN gNB using SDR (OTA)
docker compose -f srsgnb.yaml up -d && docker container attach srsgnb
```


5. Configure
```bash
# Open `http://<YOUR_HOST_IP_ADDR>:9999` in a web browser. Login with following credentials
Username : admin
Password : 1423

IMSI : <SIM_IMSI> (e.g. 001010123456790)
MSISDN : <DESIRED_MSISDN> (e.g. 9076543210)
K : <SIM_K> (e.g. 00000000000000000000000000000000)
OPC : <SIM_OPC> (e.g. 00000000000000000000000000000000)

APN Configuration:
---------------------------------------------------------------------------------------------------------------------
| APN      | Type | QCI | ARP | Capability | Vulnerablility | MBR DL/UL(Kbps)     | GBR DL/UL(Kbps) | PGW IP        |
---------------------------------------------------------------------------------------------------------------------
| internet | IPv4 | 9   | 8   | Disabled   | Disabled       | unlimited/unlimited |                 |               |
|          |      | 1   | 2   | Enabled    | Enabled        | 128/128             | 128/128         |               |
|          |      | 2   | 4   | Enabled    | Enabled        | 128/128             | 128/128         |               |
---------------------------------------------------------------------------------------------------------------------
| ims      | IPv4 | 5   | 1   | Disabled   | Disabled       | 3850/1530           |                 |               |
|          |      | 1   | 2   | Enabled    | Enabled        | 128/128             | 128/128         |               |
|          |      | 2   | 4   | Enabled    | Enabled        | 128/128             | 128/128         |               |
---------------------------------------------------------------------------------------------------------------------
```

## Other 

### Some issues

### [INSTALL usrp driver](https://blog.csdn.net/qq_36666115/article/details/144943242)

```bash
# Run on physical machines without docker
sudo add-apt-repository ppa:ettusresearch/uhd
sudo apt-get update
sudo apt-get install libuhd-dev uhd-host
sudo uhd_images_downloader
```
