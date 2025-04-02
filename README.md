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
ikev2 remote-authentication pre-shared-key cisco
ikev2 local-authentication pre-shared-key cisco
exit
```
### ASA2
```bash
tunnel-group 172.16.1.1 type ipsec-l2l
tunnel-group 172.16.1.1 ipsec-attributes
ikev2 remote-authentication pre-shared-key cisco
ikev2 local-authentication pre-shared-key cisco
exit
```
##
### 5. Create IKEv2 Policy 
### ASA1 & ASA2
```bash
crypto ikev2 policy 10 
encryption aes-256
integrity sha256
group 14
prf sha256
lifetime seconds 86400
exit
crypto ikev2 enable outside
```
##
### 6. Create Ipsec preposal 
### ASA1 & ASA2
```bash
crypto IPsec ikev2 ipsec-proposal IPSEC-PRO
protocol esp encryption aes-256
protocol esp integrity sha-256
exit
```
##
### 7. Create and Apply Crypto Map
### ASA1
```bash
crypto map 10-20 10 match address VPN10
crypto map 10-20 10 set peer 172.16.1.2
crypto map 10-20 10 set ikev2 ipsec-proposal IPSEC-PRO
crypto map 10-20 interface outside 
```
### ASA2
```bash
crypto map 20-10 10 match address VPN20
crypto map 20-10 10 set peer 172.16.1.1
crypto map 20-10 10 set ikev2 ipsec-proposal IPSEC-PRO
crypto map 20-10 interface outside
```
##
### 9. PC1 & PC2 Ip address configurations 
### Pc1 
```bash
sudo ip addr add 192.168.10.10/24 dev eth0
ip route add default via 192.168.10.1
```
### Pc2
```bash
sudo ip addr add 192.168.20.20/24 dev eth0
ip route add default via 192.168.20.1
```
##
### 10. Verification & Testing
### From PC1, test connectivity:
```bash
ping 192.168.20.20
```
![image](https://github.com/user-attachments/assets/21e414e9-d449-42a3-8777-18f2342bda1d)

### From PC2, test connectivity:
```bash
ping 192.168.10.10
```
![image](https://github.com/user-attachments/assets/2c12a8af-e51c-4ca6-9b47-9145b0ea3fa0)

### On ASA1 and ASA2, verify the VPN status:
```bash
show crypto ikev2 sa
show crypto ipsec sa
```
![image](https://github.com/user-attachments/assets/f61a2ecc-7266-4f94-a20f-d9d9dfba02ae)
![image](https://github.com/user-attachments/assets/037d1c1d-b8a2-4e48-90d4-ee8dfeab4a71)
##

![image](https://github.com/user-attachments/assets/61c87644-f80c-492d-aaae-f418c25ec102)


