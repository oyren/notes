# A guid to:
Run a headless node on a debian vps as a systemd service.

# Prerequisites
- VPS server with Debian OS (required for this guide)  
- 2GB of Ram minimum , 4GB of ram recommended
- 10GB of free disk space (ssd preferred).

# Setup
## Installation of required software
``` sh
su -
apt-get -y update
apt-get -y install openjdk-8-jre
apt-get -y install wget
```

## Download lastes iri from releases on github
http://github.com/iotaledger/iri/releases
``` sh
mkdir -p /opt/iota && cd /opt/iota
wget -O IRI.jar https://github.com/iotaledger/iri/releases/download/v1.2.2/iri-1.2.2.jar
```
Note: I renamed the iri-1.2.2.jar file to IRI.jar.
## setup iota.ini
``` sh
vim /opt/iota/iota.ini
```
and insert the following:
``` sh
[IRI]
PORT = 14700
UDP_RECEIVER_PORT = 14600
TCP_RECEIVER_PORT = 14265
NEIGHBORS = udp://neighbour1.com:14600 udp://neighbour2:14600 udp://iota.neighbour3:14800
IXI_DIR = ixi
HEADLESS = true
DEBUG = true
TESTNET = false
DB_PATH = db
```
## setup the iota systemd service
``` sh
vim /etc/systemd/system/iota.service
```
and again insert the following:
``` sh
[Unit]
Description=IOTA node
After=network.target

[Service]
WorkingDirectory=/opt/iota
ExecStart=/usr/bin/java -jar IRI.jar -c iota.ini 
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure

[Install]
WantedBy=multi-user.target
Alias=iota.service
```

## run the node
systemctl daemon-reload && systemctl restart iota
systemctl status iota

# add new neighbors
just add the address in /opt/iota/iota.ini
and restart the iota service
``` sh
systemctl daemon-reload && systemctl restart iota
systemctl status iota
```
# Credits
https://forum.iota.org/t/setting-up-a-headless-node-on-a-ubuntu-iri-version-1-2-1/1332