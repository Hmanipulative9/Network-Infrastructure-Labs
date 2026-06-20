# Dhcp Using Switch

## 🖼️ Network Topology
![Topology](dhcp using switch-topology.png)

```markdown
## 📄 Overview: This configuration establishes a multi-VLAN switching infrastructure with integrated DHCP services for network segmentation.

## 🛠️ Core Capabilities:
*   **802.1Q Trunking**
*   **VLAN Segmentation (VLAN 1, 10, 20)**
*   **DHCP Services**
*   **Ethernet Switching**

## 📊 Addressing Matrix:
| Device / Interface | IP Address | VLAN / Role |
| :----------------- | :--------- | :---------- |
| **Switch: Fa0/1** | N/A | **Trunk (VLANs 1, 10, 20)** |
| **Switch: Fa0/2** | N/A | **Trunk (VLANs 1, 10, 20)** |
| **Switch: Fa0/3-24, Gig0/1-2** | N/A | **Access (VLAN 1)** |
| **Client (VLAN 10)** | 192.168.10.2 | **IT Network Host** |
| **Client (VLAN 10)** | 192.168.10.3 | **IT Network Host** |
| **Client (VLAN 20)** | 172.20.0.1 | **HR Network Host** |
| **Client (VLAN 20)** | 172.20.0.2 | **HR Network Host** |
| **VLAN 10 Network** | 192.168.10.0/24 | **IT Subnet** |
| **VLAN 20 Network** | 172.20.0.0/24 | **HR Subnet** |

## ⚙️ Infrastructure Blueprint (CLI):
```cisco
vlan 10
 name it
vlan 20
 name HR
!
interface FastEthernet0/1
 switchport mode trunk
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 1,10,20
!
interface FastEthernet0/2
 switchport mode trunk
 switchport trunk encapsulation dot1q
 switchport trunk allowed vlan 1,10,20
!
interface FastEthernet0/3
 switchport mode access
 switchport access vlan 1
! (Additional access ports Fa0/4-24, Gig0/1-2 are also in VLAN 1)
```

## 🧪 Verification Metrics:
```cisco
Switch#show ip route
% Default gateway is not set
```
```cisco
Switch#ping 172.20.0.1 source 192.168.10.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 172.20.0.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
```
```
```