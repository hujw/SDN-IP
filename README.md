# SDN-IP Environment
## Requirements
1. VPN Server (NCHC)
2. VPN Client installs Quagga, OpenVPN, and Docker
3. ONOS docker
4. Open vSwitch

## Architecture
```
+--------------+
|              |
|     ONOS     |
|              |
+------+-------+
       |
       |
       |         +-----------------+
       +-------->+  Open vSwitch   |
                 +-----------------+
                 +-----------------+             +-------------------+
                 |                 |             |                   |
                 | Open VPN Client +-------------+  Open VPN Server  |
                 |  with Quagga    |             |    with Quagga    |
                 |                 |             |                   |
                 +-----------------+             +-------------------+
```

## Installation
In VPN client
1. Install OpenVPN
* ```sudo apt-get install openvpn```
* Go to the directory contains the *.ovpn file (ex: my.ovpn) and execute ```sudo openvpn my.ovpn```.
* Use ```ifconfig``` to check whether the interface (ex: tap0) contains a private ip address assigned by OpenVPN server. 
2. Install Quagga
* ```sudo apt-get install quagga```
* Refer to the files ([daemons](https://github.com/hujw/SDN-IP/blob/master/daemons), [bgpd.conf](https://github.com/hujw/SDN-IP/blob/master/network-cfg.json), and [zebra.conf](https://github.com/hujw/SDN-IP/blob/master/zebra.conf.example)) and replace to your information. Then, execute ```sudo /etc/init.d/quagga restart``` to start the quagga service.
* Use ```route -n``` to check whether the system receives the exchanged routes from your peers.
