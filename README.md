# SDN-IP Environment
## Requirements
1. VPN Server (NCHC site)
2. VPN Client (Peer site) installs OpenVPN and Docker
3. Open vSwitch
4. Quagga (real host or docker)
5. ONOS (docker)

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
* ```$ sudo apt-get install openvpn```
* Go to the directory contains the *.ovpn file (ex: my.ovpn) and execute ```sudo openvpn my.ovpn```.
* Use ```ifconfig``` to check whether the interface (ex: tap0) contains a private ip address assigned by OpenVPN server. 

2. Install OvS
```
$ sudo apt-get install qemu-kvm libvirt-bin ubuntu-vm-builder bridge-utils 
$ sudo apt-get install openvswitch-switch
```
Create a bridge br0 and attach tap0 to it
```
$ sudo ovs-vsctl add-br br0
$ sudo ovs-vsctl add-port br0 tap0
```
Move the ipaddress from tap0 to br0
```
$ sudo ifconfig tap0 0
$ sudo ifconfig br0 10.73.75.x netmask 255.255.255.0
```

3. Install Quagga (Real host)
* ```sudo apt-get install quagga```
* Refer to the files ([daemons](https://github.com/hujw/SDN-IP/blob/master/daemons), [bgpd.conf](https://github.com/hujw/SDN-IP/blob/master/network-cfg.json), and [zebra.conf](https://github.com/hujw/SDN-IP/blob/master/zebra.conf.example)) and replace to your information. Then, execute ```sudo /etc/init.d/quagga restart``` to start the quagga service.
* Use ```route -n``` to check whether the system receives the exchanged routes from your peers.

4. Install Quagga (Docker)
```
$ sudo docker pull cumulusnetworks/quagga
$ docker run -t -d --privileged --name Quagga cumulusnetworks/quagga:latest
```
Add port (docker quagga) and assign IP (for VPN) into OvS
```
$ sudo ovs-docker add-port br0 tap0 Quagga --ipaddress=10.73.75.55/24
```
Check the ip address of Quagga container
```
root@c42638db5085:/# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
84: tap0@if85: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 62:8c:a8:68:79:ee brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.73.75.55/24 scope global tap0
       valid_lft forever preferred_lft forever
```
Check the connection between the host and Quagga container
```
$ ip addr | grep br0
67: br0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default qlen 1000
    inet 10.73.75.2/24 brd 10.73.75.255 scope global br0

$ ping 10.73.75.55
PING 10.73.75.55 (10.73.75.55) 56(84) bytes of data.
64 bytes from 10.73.75.55: icmp_seq=1 ttl=64 time=0.623 ms
64 bytes from 10.73.75.55: icmp_seq=2 ttl=64 time=0.071 ms
```
Configure the files including daemon, zebra.conf, and bgpd.conf. Refer to the files ([daemons](https://github.com/hujw/SDN-IP/blob/master/daemons), [bgpd.conf](https://github.com/hujw/SDN-IP/blob/master/network-cfg.json), and [zebra.conf](https://github.com/hujw/SDN-IP/blob/master/zebra.conf.example)).
```
$ sudo docker cp daemons Quagga:/etc/quagga/daemons
$ sudo docker cp bgpd.conf Quagga:/etc/quagga/bgpd.conf
$ sudo docker cp zebra.conf Quagga:/etc/quagga/zebra.conf
$ docker exec -it Quagga /bin/bash
root@c42638db5085:/# /usr/lib/quagga/quagga restart
```
Check whether the bgp routes are exchanged
```
$ route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
10.73.75.0      0.0.0.0         255.255.255.0   U     0      0        0 br0
30.10.10.0      10.73.75.55     255.255.255.0   UG    0      0        0 br0
```

5. Install and configure ONOS (Docker)
```
$ sudo docker run -d --name=onos --net=none onosproject/onos
$ ssh -p 8101 karaf@172.17.0.2 # password is karaf
onos> app activate org.onosproject.openflow
onos> app activate org.onosproject.sdnip
```
