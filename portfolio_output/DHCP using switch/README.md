# Dhcp Using Switch

## 🖼️ Network Topology
![Topology](dhcp using switch-topology.png)

## 📄 Overview
This document details the configuration of a Cisco Layer 2 access switch, designed to segment a local area network (LAN) into distinct broadcast domains using Virtual Local Area Networks (VLANs). The primary purpose is to enhance network security, improve performance, and simplify network management by logically separating different user groups.

The switch acts as an access layer device, forwarding traffic based on VLAN assignments and trunking aggregated VLAN traffic to an upstream Layer 3 device for inter-VLAN routing. DHCP services are handled by an external server, providing IP addresses to clients within their respective VLANs, facilitating efficient resource allocation and network scalability.

## 🛠️ Technical Stack
*   **Ethernet:** Physical Layer connectivity.
*   **VLANs (802.1Q):** Logical network segmentation.
*   **Trunking:** Carrying multiple VLANs over a single physical link.
*   **DHCP:** Dynamic Host Configuration Protocol for automatic IP address assignment (client-side observed).

## 📊 Addressing & Topology
| VLAN ID | VLAN Name | Subnet (Derived from DHCP) | Interfaces (Access)                      | Interfaces (Trunk)    | Purpose / Notes                                        |
| :------ | :-------- | :------------------------- | :--------------------------------------- | :-------------------- | :----------------------------------------------------- |
| 1       | default   | N/A                        | Fa0/3-24, Gig0/1-2                       | Fa0/1, Fa0/2          | Default VLAN, likely for general-purpose or management. |
| 10      | it        | 192.168.10.0/24            | None (active on trunk)                   | Fa0/1, Fa0/2          | Dedicated for IT personnel, ready for port assignment. |
| 20      | HR        | 172.20.0.0/24              | None (active on trunk)                   | Fa0/1, Fa0/2          | Dedicated for HR personnel, ready for port assignment. |

## ⚙️ Core Configurations
The following snippets represent the critical commands for the switch's operation:

**VLAN Creation:**
Creating VLANs to segment user groups.
```cisco
vlan 10
 name it
vlan 20
 name HR
```

**Trunk Port Configuration:**
Configuring interfaces to carry multiple VLANs to an upstream router or switch, using 802.1Q encapsulation.
```cisco
interface FastEthernet0/1
 switchport mode trunk
 switchport trunk encapsulation dot1q
interface FastEthernet0/2
 switchport mode trunk
 switchport trunk encapsulation dot1q
```

**Access Port Configuration (Example for VLAN 1):**
Assigning specific ports to VLAN 1. Note: VLAN 10 and 20 currently show no access ports explicitly assigned in the latest `show vlan` output, but their existence on trunks and DHCP bindings indicate they are active.
```cisco
interface FastEthernet0/3
 switchport mode access
 switchport access vlan 1
! ... (similar configuration for Fa0/4-24, Gig0/1-2 for VLAN 1)
```

## 🧪 Verification
Standard diagnostic commands to confirm the network state and functionality.

**Verify VLANs and Port Assignments:**
Check the status of created VLANs and their assigned ports.
```cisco
show vlan brief
```
```cisco
VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/3, Fa0/4, Fa0/5, Fa0/6
                                                Fa0/7, Fa0/8, Fa0/9, Fa0/10
                                                Fa0/11, Fa0/12, Fa0/13, Fa0/14
                                                Fa0/15, Fa0/16, Fa0/17, Fa0/18
                                                Fa0/19, Fa0/20, Fa0/21, Fa0/22
                                                Fa0/23, Fa0/24, Gig0/1, Gig0/2
10   it                               active    
20   HR                               active    
1002 fddi-default                     active    
1003 token-ring-default               active    
1004 fddinet-default                  active    
1005 trnet-default                  active    
```

**Verify Trunk Port Status:**
Confirm which interfaces are operating as trunks and the VLANs they are carrying.
```cisco
show interfaces trunk
```
```cisco
Port        Mode         Encapsulation  Status        Native vlan
Fa0/1       on           802.1q         trunking      1
Fa0/2       on           802.1q         trunking      1

Port        Vlans allowed on trunk
Fa0/1       1-1005
Fa0/2       1-1005

Port        Vlans allowed and active in management domain
Fa0/1       1,10,20
Fa0/2       1,10,20

Port        Vlans in spanning tree forwarding state and not pruned
Fa0/1       1,10,20
Fa0/2       1,10,20
```

**Verify DHCP Bindings:**
Review active DHCP leases from clients, indicating successful IP assignment across VLANs.
```cisco
show ip dhcp binding
```
```cisco
IP address       Client-ID/              Lease expiration        Type
                 Hardware address
192.168.10.3     0002.1772.8421           --                     Automatic
192.168.10.2     0001.43EE.698B           --                     Automatic
172.20.0.2       00E0.F96C.524A           --                     Automatic
172.20.0.1       00D0.D3D1.9D7E           --                     Automatic
```