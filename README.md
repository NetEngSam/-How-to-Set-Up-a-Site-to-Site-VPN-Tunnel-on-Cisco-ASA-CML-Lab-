#  How to Set Up a Site-to-Site VPN Tunnel on Cisco ASA (Cisco CML-Lab)
Establish a secure IPSec VPN tunnel between two sites using Cisco ASA firewalls in a simulated environment (Cisco Modeling Labs).

## 🖥️ Lab Topology
  ![image](https://github.com/user-attachments/assets/4dbc2b67-1e26-4bb1-ac45-d40dd9dab599)

- ASA1 = Site 1 Firewall  
- ASA2 = Site 2 Firewall  
- PC1/PC2 used for traffic generation and testing  
- G0/0 = Outside  
- G0/1 = Inside
---
## 🧰 Tools Used
- Cisco Modeling Labs (CML)
- Cisco ASA (Virtual ASA Nodes)
- Linux PCs
---
## ⚙️ Configuration Steps

### 1. Interface Configuration (IP addresses)

#### ASA1
```bash
conf t

hostname asa1

int g0/0 
ip address 172.16.1.1 255.255.255.0
nameif outside
no shutdown

int g0/1
ip address 192.168.10.1 255.255.255.0
nameif inside 
no shutdown
exit
```
### ASA2
```bash
conf t

hostname asa2

int g0/0
ip address 172.16.1.2 255.255.255.0
nameif outside 
no shutdown 


int g0/1
ip address 192.168.20.1 255.255.255.0 
nameif inside 
no shutdown
exit
```
##
### 2. Add Default Route 

### ASA1
```bash
route outside 192.168.20.0 255.255.255.0 172.16.1.2
```
### ASA2
```bash
route outside 192.168.10.0 255.255.255.0 172.16.1.1
```

##
### 3. Create Access List for "Interesting Traffic"
### ASA1
```bash
access-list VPN10 extended permit ip 192.168.10.0 255.255.255.0 192.168.20.0 255.255.255.0
```
### ASA2
```bash
access-list VPN20 extended permit ip 192.168.20.0 255.255.255.0 192.168.10.0 255.255.255.0
```
##
### 4. Define Tunnel Group and Pre-Shared Key (IKEv2)
### ASA1
```bash
tunnel-group 172.16.1.2 type ipsec-l2l
tunnel-group 172.16.1.2 ipsec-attributes
ikev2 remote-authentication pre-shared key cisco
ikev2 local-authentication pre-shared key cisco
```
### ASA2
```bash
tunnel-group 172.16.1.1 type ipsec-l2l
tunnel-group 172.16.1.1 ipsec-attributes
ikev2 remote-authentication pre-shared-key cisco
ikev2 local-authentication pre-shared-key cisco
```
##













