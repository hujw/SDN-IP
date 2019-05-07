# SDN-IP Environment
## Requirements
1. VPN Server (NCHC)
2. VPN Client installs Quagga, OpenVPN, and Docker
3. Pull ONOS from docker

## Architecture
```
+-------------+
|    ONOS     |
+-------------+
|   Docker    |
+-------------+           +------------+
|  VPN Client |           |            |
|    with     +-----------+ VPN Server |
|   Quagga    |           |            |
+-------------+           +------------+
```
