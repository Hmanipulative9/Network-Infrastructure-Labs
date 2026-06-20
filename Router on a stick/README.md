# Router On A Stick

## 🖼️ Network Topology
![Topology](router-on-a-stick-topology.png)

## 📄 Overview
Implemented a switched network infrastructure providing VLAN-based network segmentation for distinct departments.

## 🎯 Project Objective
To segment network traffic for different departments into dedicated broadcast domains, enhancing security and manageability within a local area network.

## 💡 Skills Demonstrated
- VLAN Configuration
- Spanning Tree Protocol (PVST)
- Switchport Access Mode Configuration
- Basic Cisco IOS Configuration

## 🛠️ Core Capabilities
- VLANs (IEEE 802.1Q)
- Per-VLAN Spanning Tree (PVST)
- Ethernet Switching

## 📊 Addressing Matrix

| Device / Interface | IP Address | VLAN / Role |
|-------------------|-----------|------------|
| Switch / Fa0/1 | - | VLAN 10 (HR) |
| Switch / Fa0/2 | - | VLAN 10 (HR) |
| Switch / Fa0/3 | - | VLAN 20 (IT) |
| Switch / Fa0/4 | - | VLAN 20 (IT) |
| Switch / Fa0/6 | - | VLAN 30 (CEO) |
| Switch / Fa0/7-24 | - | VLAN 1 (Default) |
| Switch / Gig0/1-2 | - | VLAN 1 (Default) |

## ⚙️ Infrastructure Blueprint

```cisco
hostname Switch
!
spanning-tree mode pvst
spanning-tree extend system-id
!
vlan 10
 name HR
vlan 20
 name IT
vlan 30
 name CEO
!
interface FastEthernet0/1
 switchport access vlan 10
 switchport mode access
```

## 🧪 Verification Metrics

```text
Switch#show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/7, Fa0/8, Fa0/9, Fa0/10
                                                Fa0/11, Fa0/12, Fa0/13, Fa0/14
                                                Fa0/15, Fa0/16, Fa0/17, Fa0/18
                                                Fa0/19, Fa0/20, Fa0/21, Fa0/22
                                                Fa0/23, Fa0/24, Gig0/1, Gig0/2
10   HR                               active    Fa0/1, Fa0/2
20   IT                               active    Fa0/3, Fa0/4
30   CEO                              active    Fa0/6
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default               active    
```