# Router On A Stick

## 🖼️ Network Topology
![Topology](router on a stick-topology.png)

## 📄 Overview

This configuration outlines a foundational Layer 2 network segmentation strategy for a single access switch, designed to isolate different departmental traffic. It leverages Virtual Local Area Networks (VLANs) to logically separate HR, IT, and CEO users, enhancing security, reducing broadcast domains, and improving overall network performance.

The core business problem addressed is the need for secure and efficient departmental communication without intermixing traffic. By assigning specific switch ports to their respective VLANs, sensitive information is kept separate, and network resources are more effectively utilized, providing a secure and organized environment for various business functions.

## 🛠️ Technical Stack

*   **VLANs (Virtual Local Area Networks):** Logical segmentation of the network.
*   **PVST (Per-VLAN Spanning Tree):** Prevents Layer 2 loops within each VLAN.

## 📊 Addressing & Topology

This device functions as a Layer 2 switch, primarily concerned with MAC address forwarding and VLAN assignment. IP addressing for client devices within these VLANs would typically be handled by an upstream Layer 3 device (router or Layer 3 switch).

| VLAN ID | VLAN Name | Interface(s) Assigned | Notes                                      |
| :------ | :-------- | :-------------------- | :----------------------------------------- |
| 1       | default   | Fa0/7-24, Gig0/1-2    | Default VLAN, typically for unassigned ports |
| 10      | HR        | Fa0/1, Fa0/2          | Human Resources department                 |
| 20      | IT        | Fa0/3, Fa0/4          | Information Technology department          |
| 30      | CEO       | Fa0/6                 | Executive office                           |

## ⚙️ Core Configurations

**1. Hostname Configuration:**
*Sets the device's administrative name.*
```cisco
hostname Switch
```

**2. Spanning Tree Protocol (STP) Configuration:**
*Enables Per-VLAN Spanning Tree Protocol to prevent loops within each VLAN.*
```cisco
spanning-tree mode pvst
spanning-tree extend system-id
```

**3. VLAN Creation and Naming:**
*Defines the logical segments and assigns descriptive names.*
```cisco
vlan 10
 name HR
vlan 20
 name IT
vlan 30
 name CEO
```

**4. Access Port Configuration (Example for HR VLAN):**
*Configures FastEthernet0/1 as an access port, assigning it to the HR VLAN.*
```cisco
interface FastEthernet0/1
 switchport access vlan 10
 switchport mode access
```

## 🧪 Verification

**1. Verify VLAN Assignments:**
*Confirms that VLANs are configured and ports are correctly assigned to them.*
```cisco
Switch# show vlan brief
```
**Simulated Output:**
```cisco
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
```

**2. Verify Spanning Tree Protocol Status:**
*Checks the spanning-tree mode and status for critical VLANs.*
```cisco
Switch# show spanning-tree summary
```
**Simulated Output:**
```cisco
Root internal
Total PVST instances                      : 4 (for Vlans 1, 10, 20, 30)
Root bridge for: VLAN0001, VLAN0010, VLAN0020, VLAN0030
[...]
Mode                                    PVST
```