# A guide to:
Run a headless IOTA node on a Debian based VPS as a systemd service.
(For more detailed instructions on setting up a VPS with Digital Ocean see: https://knarz.github.io/notes/iota-node-do/)

# Requirements
- VPS server with Debian based OS (that support systemd)  
- 2GB of Ram minimum , 4GB of ram recommended
- 10GB of free disk space (ssd preferred).
- a static ip (Most VPS service include static IP)

# Setup
## Installation of required software
``` sh
su -
apt-get -y update
apt-get -y install openjdk-8-jre
apt-get -y install wget
```

## Download lastes IRI from Github (Current version 1.4.0)
https://github.com/iotaledger/iri/releases
``` sh
mkdir -p /opt/iota && cd /opt/iota
wget -O IRI.jar https://github.com/iotaledger/iri/releases/download/v1.4.1.1/iri-1.4.1.1.jar
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
NEIGHBORS = udp://example.neighbor1.com:14600 udp://example.neighbor2.com:14600 udp://iota.neighbour3:14800
IXI_DIR = ixi
HEADLESS = true
DEBUG = true
TESTNET = false
DB_PATH = db
```
Note: You will need to find and add neighbors. Best way to do it is to subscribe to the IOTA slack channel (slack.iota.org) and join the #nodesharing channel. Then insert the neighbors IP into iota.ini file above (remove example neighbors and add your own neighbors) 

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
ExecStart=/usr/bin/java -Xmx4g -jar IRI.jar -c iota.ini --remote
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure

[Install]
WantedBy=multi-user.target
Alias=iota.service
```
Note: You probably have to change the "-Xmx4g" parameter to the amount of RAM the node should use (4g = 4 GB of RAM).
**For those who start a new node and synchronize the first time:  
Please add the following line to the iota.ini configuration file:  
P_REMOVE_REQUEST = 0.0  
The default for this value in IRI is 0.03. That means that 3% of requests for missing transactions are constantly removed from the request queue.  
I have the suspicion that P_REMOVE_REQUEST = 0.03 contributes to the difficulties to get a node *solid*.
Once your node is synched, you can remove this line and restart IRI.**
## Run the node
``` sh 
systemctl daemon-reload && systemctl restart iota
systemctl status iota
```

# Add new neighbors
Just add the address in /opt/iota/iota.ini
and restart the iota service
``` sh
systemctl daemon-reload && systemctl restart iota && systemctl status iota
```

# You have now succesfully set-up a headless 24/7 IOTA node! 

# Maintaining the node
To see the logs of the node process:
``` sh
journalctl -u iota -f
```

To check the systemd status
``` sh
systemctl status iota
```


To check the connection
``` sh
curl http://localhost:14700 -X POST -H 'Content-Type:application/json' -d '{"command":"getNeighbors"}' | python -m json.tool
or
curl http://localhost:14700 -X POST -H 'Content-Type: application/json' -d '{"command": "getNodeInfo"}' | python -m json.tool
```
Note: You have to change the port to match yours.  

Updating the node to a newer version:
- Just replace the old IRI.jar with the new one and restart the iota service.

``` sh
cd /opt/iota
wget -O IRI.jar https://github.com/iotaledger/iri/releases/download/v1.x.x/iri-1.x.x.jar
```

``` sh
systemctl daemon-reload && systemctl restart iota
```
# Some Errors

Connection refused or Connection reset by peer:
- make sure you are communicating through the right port (iota.ini or cURL/Python commands)

# Credits and useful links
https://forum.iota.org/t/setting-up-a-headless-node-on-a-ubuntu-iri-version-1-2-1/1332  
https://knarz.github.io/notes/iota-node-do/  
http://www.iotasupport.com/gettingstarted.shtml  
https://iota.readme.io/docs/  
