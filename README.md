# SDN-IP Environment
## Requirements
1. VPN Server (NCHC)
2. VPN Client installs Quagga, OpenVPN, and Docker
3. ONOS docker
4. Open vSwitch

## Architecture
Simple case (only Quagga in the peer site)
```
                       |
 exchanged routes      |                                                         +--------+
 20.20.20.0/24         |      tap0: 10.73.75.1                                   | SDNIP  |
+----------------+     |     +----------------+         +---------------+        +--------+
|                |     |     |                |         |               |  MGMT  |        |
| OpenVPN client +-----------+ OpenVPN server +---------+  OpenFlow sw  +--------+  ONOS  |
|  with Quagga   |tap0 |     |                |         |               |        |        |
|                |10.73.75.x |                |         +-------+-------+        +--------+
+-------+--------+     |     +----------------+                 |
        | eth1         |                                        |
        |              |                                  +-----+------+ exchanged routes
   +----+---+          |                                  |            | 10.10.10.0/24
   |  Host  |          |                                  |   Quagga   |
   +--------+          |                                  |            | IP:10.73.75.8
    IP:20.20.20.2      |                                  +-----+------+
                       |                                        |
                       |                                    +---+----+
                       |                                    |  Host  |
  (Peer site)          |          (NCHC site)               +--------+
                       |                                     IP:10.10.10.2
                       |

```
Complex case (install SDN-IP in the peer site)
```




                                                    |
+------+                                            |                                               +--------+
|SDNIP |                         tap0:10.73.75.x    |     tap0:10.73.75.1                           | SDNIP  |
+------+    +------------+     +----------------+   |   +----------------+   +---------------+      +--------+
|      |    |OpenFlow sw |     |                |   |   |                |   |               | MGMT |        |
| ONOS +----+     or     +-----+ OpenVPN client +-------+ OpenVPN server +---+  OpenFlow sw  +------+  ONOS  |
|      |    |    OVS     |     |                |   |   |                |   |               |      |        |
+------+    +-----+------+     +----------------+   |   +----------------+   +-------+-------+      +--------+
                  |                                 |                                |
                  |                                 |                                |
             +----+----+ exchanged routes           |                          +-----+------+ exchanged routes
             |         | 20.20.20.0/24              |                          |            | 10.10.10.0/24
             | Quagga  |                            |                          |   Quagga   |
             |         | IP:10.73.75.y              |                          |            | IP:10.73.75.8
             +---+-----+                            |                          +-----+------+
                 |                                  |                                |
              +--+---+                              |                            +---+----+
              | Host |                              |                            |  Host  |
              +------+           (Peer site)        |        (NCHC site)         +--------+
            IP:20.20.20.2                           |                             IP:10.10.10.2
                                                    |
                                               |
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
3. Docker and OvS
* Install OvS in Ubuntu 16.04
```
sudo apt-get install qemu-kvm libvirt-bin ubuntu-vm-builder bridge-utils 
sudo apt-get install openvswitch-switch
```
* Configure Docker container to attach the port of OvS (Refer to: https://medium.com/@joatmon08/making-it-easier-docker-containers-on-open-vswitch-4ed757619af9)
```
sudo docker run -d --name=onos --net=none onosproject/onos
sudo ovs-docker add-port br0 eth0 onos --ipaddress=10.73.75.22/24 --gateway=10.73.75.2
```
