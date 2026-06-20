# Dhcp Using Switch

## 🖼️ Network Topology
![Topology](dhcp using switch-topology.png)

```markdown
## 📄 Overview: This deployment establishes VLAN-based network segmentation and dynamic IP address assignment for connected endpoints.

## 🛠️ Core Capabilities:
*   **VLAN Segmentation** (VLAN 1, 10, 20)
*   **IEEE 802.1Q Trunking**
*   **Dynamic Host Configuration Protocol (DHCP) client binding awareness**
*   **Layer 2 Switching**

## 📊 Addressing Matrix:
| Device / Interface | IP Address | VLAN / Role |
|---|---|---|
| VLAN 10 Network | 192.168.10.0/24 | IT Department |
| VLAN 20 Network | 172.20.0.0/24 | HR Department |
| Switch | *No IP configured* | Layer 2 Core |

## ⚙️ Infrastructure Blueprint (CLI):
```cisco
vlan 10
 name it
vlan 20
 name HR

interface FastEthernet0/1
 switchport mode trunk
 switchport trunk encapsulation dot1q

interface FastEthernet0/2
 switchport mode trunk
 switchport trunk encapsulation dot1q
```

## 🧪 Verification Metrics:
```cisco
Switch#show ip route
% IP routing is not enabled
```

```bash
PC_VLAN10# ping 172.20.0.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 172.20.0.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/2/4 ms
```