# Dhcp Using Switch

## 🖼️ Network Topology
![Topology](dhcp using switch-topology.png)

```markdown
# Network Infrastructure Overview

## 📄 Overview
This document outlines the configuration of a core Cisco Layer 2 access switch, designed to provide segmented network connectivity for various departmental users. The architecture leverages VLANs to logically separate traffic, enhancing security, manageability, and network performance by reducing broadcast domains.

The primary objective of this setup is to ensure reliable and organized network access for the default user group, IT department, and HR department. The switch acts as an aggregation point, trunking segregated traffic upstream to a Layer 3 device responsible for inter-VLAN routing and DHCP services, thereby supporting diverse user requirements across the organization.

## 🛠️ Technical Stack
*   **VLANs (Virtual Local Area Networks):** Logical network segmentation.
*   **802.1Q Trunking:** Standard for carrying multiple VLANs over a single link.
*   **DHCP (Dynamic Host Configuration Protocol):** For automatic IP address assignment to network clients (leveraged from an external server).

## 📊 Addressing & Topology
The following table details the interfaces, their assigned IP addresses (where applicable), associated subnets, and VLAN memberships based on the provided configuration snippets.

| Device/Interface | IP Address      | Subnet Mask (Inferred) | VLAN | Role/Description               |
| :--------------- | :-------------- | :--------------------- | :--- | :----------------------------- |
| **Switch**       |                 |                        |      |                                |
| Fa0/1            | N/A             | N/A                    | 1,10,20 (Trunk) | Uplink/Trunk to L3 device      |
| Fa0/2            | N/A             | N/A                    | 1,10,20 (Trunk) | Uplink/Trunk to L3 device      |
| Fa0/3-24, Gig0/1-2 | N/A             | N/A                    | 1 (Access)   | Default/General Access Ports   |
| **Learned DHCP Clients** |             |                        |      |                                |
| IT Client 1      | 192.168.10.2    | 255.255.255.0          | 10   | IT Department End Device       |
| IT Client 2      | 192.168.10.3    | 255.255.255.0          | 10   | IT Department End Device       |
| HR Client 1      | 172.20.0.1      | 255.255.255.0          | 20   | HR Department End Device       |
| HR Client 2      | 172.20.0.2      | 255.255.255.0          | 20   | HR Department End Device       |

## ⚙️ Core Configurations
Critical configuration snippets are provided below, illustrating the setup for VLANs and trunking.

**VLAN Definitions:**
```cisco
vlan 10
 name it
vlan 20
 name HR
```
*   Creates VLAN 10 for the IT department and VLAN 20 for the HR department.

**Trunk Port Configuration:**
```cisco
interface FastEthernet0/1
 switchport mode trunk
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 1
 switchport trunk allowed vlan 1,10,20
!
interface FastEthernet0/2
 switchport mode trunk
 switchport trunk encapsulation dot1q
 switchport trunk native vlan 1
 switchport trunk allowed vlan 1,10,20
```
*   Configures FastEthernet0/1 and Fa0/2 as 802.1Q trunks, allowing VLANs 1, 10, and 20 to traverse. Native VLAN is set to 1.

**Access Port Configuration (Sample):**
```cisco
interface FastEthernet0/3
 switchport mode access
 switchport access vlan 1
```
*   Configures FastEthernet0/3 as an access port, assigning it to VLAN 1 (default). This pattern applies to Fa0/3-24 and Gig0/1-2.

## 🧪 Verification
Standard diagnostic commands to confirm the network's operational status.

**1. Verify VLAN Configuration:**
```cisco
Switch# show vlan brief

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

**2. Verify Trunk Port Status:**
```cisco
Switch# show interfaces trunk

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

**3. Verify IP Interface Status:**
```cisco
Switch# show ip interface brief

Interface              IP-Address      OK? Method Status                Protocol 
FastEthernet0/1        unassigned      YES unset  up                    up 
FastEthernet0/2        unassigned      YES unset  up                    up 
FastEthernet0/3        unassigned      YES unset  down                  down 
<... output truncated for brevity ...>
FastEthernet0/23       unassigned      YES unset  down                  down 
FastEthernet0/24       unassigned      YES unset  down                  down 
GigabitEthernet0/1     unassigned      YES unset  down                  down 
GigabitEthernet0/2     unassigned      YES unset  down                  down 
```

**4. Verify DHCP Bindings (Learned from External Server):**
```cisco
Switch# show ip dhcp binding

IP address       Client-ID/              Lease expiration        Type
                 Hardware address
192.168.10.3     0002.1772.8421           --                     Automatic
192.168.10.2     0001.43EE.698B           --                     Automatic
172.20.0.2       00E0.F96C.524A           --                     Automatic
172.20.0.1       00D0.D3D1.9D7E           --                     Automatic
```
```