# Dhcp Using Switch

## 🖼️ Network Topology
![Topology](dhcp-using-switch-topology.png)

## 📄 Overview
Provides Layer 2 network segmentation and automated IP address assignment for connected devices across multiple VLANs.

## 🎯 Project Objective
Establish distinct logical network segments for departmental separation and streamline IP address management for network clients.

## 💡 Skills Demonstrated
- VLAN Configuration
- Trunking (IEEE 802.1Q)
- DHCP Services
- Layer 2 Switching

## 🛠️ Core Capabilities
- VLAN
- IEEE 802.1Q
- DHCP

## 📊 Addressing Matrix

| Device / Interface | IP Address | VLAN / Role |
|-------------------|-----------|------------|
| Switch / Fa0/1 | unassigned | Trunk (802.1q), Native 1, Active VLANs 1,10,20 |
| Switch / Fa0/2 | unassigned | Trunk (802.1q), Native 1, Active VLANs 1,10,20 |
| Switch / Fa0/3-Fa0/24, Gig0/1-Gig0/2 | unassigned | Access (VLAN 1) |
| Client 1 | 192.168.10.2 | VLAN 10 (it) |
| Client 2 | 192.168.10.3 | VLAN 10 (it) |
| Client 3 | 172.20.0.1 | VLAN 20 (HR) |
| Client 4 | 172.20.0.2 | VLAN 20 (HR) |

## ⚙️ Infrastructure Blueprint

```cisco
! VLAN Definitions
vlan 10
 name it
vlan 20
 name HR
!
! Trunk Port Configuration
interface FastEthernet0/1
 switchport mode trunk
!
interface FastEthernet0/2
 switchport mode trunk
!
! Access Port Configuration (Sample)
interface FastEthernet0/3
 switchport access vlan 1
 switchport mode access
!
```

## 🧪 Verification Metrics
- Routing table output: Not available from provided configuration.
- ICMP ping trace: Not available from provided configuration.