# A guide to:
Run a headless IOTA node on a Debian based VPS as a systemd service.

# Prerequisites
- VPS server with Debian based OS, eg Ubunutu (required for this guide)  
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

## Download lastes IRI from Github (Current version 1.2.2)
https://github.com/iotaledger/iri/releases
``` sh
mkdir -p /opt/iota && cd /opt/iota
wget -O IRI.jar https://github.com/iotaledger/iri/releases/download/v1.2.2/iri-1.2.2.jar
```
## Setup iota.ini
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
Note: you have to find and add neighbours, best way to do it, is to subscribe to the IOTA slack channel (slack.iota.org) and join the #nodesharing channel.

## Setup the IOTA systemd service
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
ExecStart=/usr/bin/java -jar  iri-1.2.2.jar -c iota.ini 
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure

[Install]
WantedBy=multi-user.target
Alias=iota.service
```

## Run the node
systemctl daemon-reload && systemctl restart iota
systemctl status iota

# Add new neighbors
just add the address in /opt/iota/iota.ini
and restart the iota service
``` sh
systemctl daemon-reload && systemctl restart iota
systemctl status iota
```
# Credits
https://forum.iota.org/t/setting-up-a-headless-node-on-a-ubuntu-iri-version-1-2-1/1332
https://knarz.github.io/notes/iota-node-do/
http://www.iotasupport.com/gettingstarted.shtml
