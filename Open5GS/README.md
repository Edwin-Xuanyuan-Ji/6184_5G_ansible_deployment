# 1.Install Open5GS

Open5GS is installed on the local virtual machine of Linux Ubantu.

```shell
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository ppa:open5gs/latest
sudo apt update
sudo apt install open5gs
```

# 2. Setup Open5GS

 As the solution part has discussed, the 5G core and gNodeB will be running on separate servers, the NGAP bind address of the AMFand the GTPU bind address of the UPF need to be configured. 

amf config is shown below

```shell
# amf config file locates in /etc/open5gs/amf.yaml
# ngap addr configured to 5g core server ip
amf:
    sbi:
      - addr: 127.0.0.5
        port: 7777
    ngap:
      - addr: xxx.xxx.xxx.xxx # your core network server, we also call it the host server
    guami:
      - plmn_id:
          mcc: 901
          mnc: 70
        amf_id:
          region: 2
          set: 1
    tai:
      - plmn_id:
          mcc: 901
          mnc: 70
        tac: 1
    plmn_support:
      - plmn_id:
          mcc: 901
          mnc: 70
        s_nssai:
          - sst: 1
    security:
        integrity_order : [ NIA2, NIA1, NIA0 ]
        ciphering_order : [ NEA0, NEA1, NEA2 ]
    network_name:
        full: Open5GS
    amf_name: open5gs-amf0
```



``` sh
# restart amf services
sudo systemctl restart open5gs-amfd
# amf logs can be found in /var/log/open5gs/amf.log
sudo tail -f /var/log/open5gs/amf.log
```

upf config is shown below

```shell
# upf config file locates in /etc/open5gs/upf.yaml
# gtpu addr configured to 5g core server ip
upf:
    pfcp:
      - addr: 127.0.0.7
    gtpu:
      - addr: xxx.xxx.xxx.xxx # your core network server, we also call it the host server
    subnet:
      - addr: 10.45.0.1/16
      - addr: 2001:db8:cafe::1/48

```

``` shell
# restart upf services
sudo systemctl restart open5gs-upfd

---

# upf logs can be found in /var/log/open5gs/upf.log
sudo tail -f /var/log/open5gs/upf.log
```

# 3.NAT Port Forwarding

NAT port forwarding: this step is desinged to build the conncetion between Internet and 5G core UPF.

``` shell
# nat port forwarding 
sudo sysctl -w net.ipv4.ip_forward=1
sudo iptables -t nat -A POSTROUTING -o ens33 -j MASQUERADE
sudo systemctl stop ufw
sudo iptables -I FORWARD 1 -j ACCEPT
```

# 4. Register UE Device

``` shell
# install nodejs
sudo apt update
sudo apt install curl
curl -fsSL https://deb.nodesource.com/setup_14.x | sudo -E bash -
sudo apt install nodejs
sudo npm install
# clone webui
git clone https://github.com/open5gs/open5gs.git

# run webui with npm
cd webui
npm run dev --host 0.0.0.0

# the web interface will start on
http://localhost:3000

# webui login credentials
username - admin
password - 1423

# add new subscriber
# the default device information can be found in open5gs config on UERANSIM
IMSI: 901700000000001
Subscriber Key: 465B5CE8B199B49FAA5F0A2EE238A6BC
USIM Type: OPc
Operator Key: E8ED289DEBA952E4283B54E88E6183CA
```

# 5.Install UERANSIM

The  UERANSIM is installed on `server2`. The installation done with `make` file available in UERANSIM repository. Following is the way to install UERANSIM.

```shell

# install cmake
# UERANSIM does not work with the apt version of cmake, that's why we need to install snap and the snap version of cmake:
sudo apt update 
sudo apt upgrade 
sudo apt install make g++ libsctp-dev lksctp-tools 
iproute2 sudo snap install cmake --classic

# install ueransim
git clone https://github.com/aligungr/UERANSIM
cd UERANSIM
make
```

# 6.Setup gNodeB

UERANSIM contains two parts gNodeB and UE. The gNodeB config files which related to Open5GS located in `UERANSIM/config/open5gs-gnb.yaml`. The ip address of `linkIp, ngapIp, gtpIp` and `amfConfigs: address` in the config file need to be configured. The ip address of `linkIp, ngapIp, gtpIp` are setup with `client server` (UERANSIM running server). The `amfConfigs: address` is IP address he `host server` ( the Open5GC running server)t. The gNB can be started with the `UERANSIM/build/nr-gnb` script by using the `UERANSIM/config/open5gs-gnb.yaml` config file.

```shell
# configure with IP address
linkIp: xxx.xxx.xxx.xxx  # #your client server ip address
ngapIp: xxx.xxx.xxx.xxx  # your client server ip address
gtpIp:  xxx.xxx.xxx.xxx   #your client server ip address

# list of AMF address information
# configure with host server IP
amfConfigs:
  - address: xxx.xxx.xxx #  the host server ip address
    port: 38412
```

```shell
# start gnb with open5gc-gnb.yaml config file
sudo ./nr-gnb -c ../config/open5gs-gnb.yaml
```

# 7.Setup UE

The UERANSIM UE config files which related to Open5GS located in `UERANSIM/config/open5gs-ue.yaml`. The `gnbSearchList` with the IP address of the client server (IP address of UERANSIM running server) need to be configured. The UE can be started with the `UERANSIM/build/nr-ue` script by using the `UERANSIM/config/open5gs-ue.yaml` config file. And the `open5gs-ue.yaml` defines the UE device configurations. The configurations should be the same  when registering the subscriber in the Open5GC 5G Core via the WebUI.

```shell
# ue device config
# IMSI number of the UE. IMSI = [MCC|MNC|MSISDN] (In total 15 digits)
supi: 'imsi-901700000000001'
# Mobile Country Code value of HPLMN
mcc: '901'
# Mobile Network Code value of HPLMN (2 or 3 digits)
mnc: '70'

# Permanent subscription key
key: '465B5CE8B199B49FAA5F0A2EE238A6BC'
# Operator code (OP or OPC) of the UE
op: 'E8ED289DEBA952E4283B54E88E6183CA'
# This value specifies the OP type and it can be either 'OP' or 'OPC'
opType: 'OPC'
# Authentication Management Field (AMF) value
amf: '8000'
# IMEI number of the device. It is used if no SUPI is provided
imei: '356938035643803'
# IMEISV number of the device. It is used if no SUPI and IMEI is provided
imeiSv: '4370816125816151'
# IMEISV number of the device. It is used if no SUPI and IMEI is provided
imeiSv: '4370816125816151'

---

# List of gNB IP addresses for Radio Link Simulation
# configure with client server ip address
gnbSearchList: 
  - xxx.xxx.xxx.xx #your client server ip address
```

 

```shell
# start gnb with open5gc-ue.yaml config file
sudo  ./nr-ue -c ../config/open5gs-ue.yaml
```

# 8.Test 5G network

The test is done by the client server. While keeping the terminal of gNB and UE both under running, open another termimal

window. The `ping` command could be used to test the network.

```shell
# ping command bind direcly to uesimtun0
ping -I uesimtun0 google.com
```

