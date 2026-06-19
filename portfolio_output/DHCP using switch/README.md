# Dhcp Using Switch

## 🖼️ Network Topology
![Topology](dhcp using switch-topology.png)

## 📄 Overview
This document outlines the configuration of a Cisco Layer 2 access switch, designed to provide segmented network connectivity for various departments within an organization. The switch leverages VLANs to logically separate traffic, enhancing security and manageability across different user groups (e.g., IT, HR, and a default segment).

The architecture facilitates efficient network resource utilization by providing trunk links to an upstream Layer 3 device (likely a router-on-a-stick), which handles inter-VLAN routing and dynamic IP address assignment via DHCP for the segmented networks. This design ensures that user devices receive appropriate network configurations and that traffic between different departmental VLANs is routed securely and effectively.

## 🛠️ Technical Stack
*   **VLANs (IEEE 802.1Q):** For logical network segmentation.
*   **Trunking:** To carry multiple VLANs over a single link between switches and/or routers.
*   **DHCP (Dynamic Host Configuration Protocol):** Implied, for automatic IP address assignment by an external server.

## 📊 Addressing & Topology
The following table details the VLANs, their inferred subnets (managed by an external router), and the interfaces on this switch that are part of these segments or serve as trunk links.

| VLAN ID | VLAN Name | Subnet (Inferred/External) | Interface(s)                              | Role                                   |
| :------ | :-------- | :------------------------- | :---------------------------------------- | :------------------------------------- |
| 1       | default   | N/A (Switch Management)    | Fa0/3-24, Gig0/1-2 (Access Ports)         | Native VLAN, Default User/Management   |
| 10      | it        | 192.168.10.0/24            | (Configured, no active access ports)      | IT Department Segment                  |
| 20      | HR        | 172.20.0.0/24              | (Configured, no active access ports)      | HR Department Segment                  |
| N/A     | Trunk     | N/A                        | Fa0/1, Fa0/2                              | Uplink to Router/Other Switches        |

## ⚙️ Core Configurations
Critical configuration snippets to achieve the described network segmentation and connectivity.

**VLAN Creation:**
```cisco
vlan 10
 name it
vlan 20
 name HR
```
*   Configures VLANs for departmental segmentation.

**Trunk Port Configuration:**
```cisco
interface FastEthernet0/1
 switchport mode trunk
 switchport trunk encapsulation dot1q
interface FastEthernet0/2
 switchport mode trunk
 switchport trunk encapsulation dot1q
```
*   Configures FastEthernet0/1 and FastEthernet0/2 as 802.1Q trunk ports to carry multiple VLANs.

**Access Port Configuration (Example for VLAN 1):**
```cisco
interface FastEthernet0/3
 switchport mode access
 switchport access vlan 1
! ... (similar configuration for Fa0/4-24, Gig0/1-2)
```
*   Assigns end-device ports to the respective VLANs (e.g., Fa0/3 to VLAN 1).

## 🧪 Verification
Standard diagnostic commands to confirm the network state and configuration.

**Verify VLANs and Port Assignments:**
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
1005 trnet-default                    active    
```

**Verify Trunk Port Status:**
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

**Verify DHCP Bindings (from external DHCP server via switch):**
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