# Router on a stick

## 1. Network Topology
![Topology](topology.png)

# Cisco Layer 2 VLAN Segmentation Project

## Project Overview

This project demonstrates foundational network segmentation using Virtual Local Area Networks (VLANs) on a Cisco Layer 2 switch. The primary goal is to logically divide a single physical network into multiple isolated broadcast domains, improving security, enhancing network performance, and simplifying network management for different organizational departments.

In a real-world scenario, this configuration is crucial for businesses looking to isolate sensitive traffic (e.g., HR data) from general user traffic (e.g., IT department), or to contain broadcast storms within smaller segments. By segmenting the network, unauthorized access between departments is restricted at Layer 2, and the overall network efficiency is boosted by reducing the size of broadcast domains. This setup forms the critical underlay for more advanced network architectures, providing a robust and secure foundation for various business applications and user groups.

## Technical Skills Demonstrated

*   **VLANs (Virtual Local Area Networks):** Logical segmentation of a Layer 2 network.
*   **Access Ports:** Assigning switch ports to specific VLANs for end-device connectivity.
*   * **Cisco IOS CLI:** Configuration and verification of network devices.
*   **Network Segmentation:** Designing and implementing isolated network segments.
*   **Broadcast Domain Management:** Reducing the size of broadcast domains for improved network performance.
*   **Basic Switch Configuration:** Setting hostname and essential operational parameters.

## Network Specifications & Addressing Table

The configuration focuses on Layer 2 segmentation using VLANs. No Layer 3 IP addressing is configured directly on the switch interfaces (Switch Virtual Interfaces - SVIs), as this switch is operating purely as a Layer 2 device. Inter-VLAN routing would typically be handled by a connected Layer 3 device (router or Layer 3 switch).

**Device:** Ros (hostname: Switch)

| Interface      | VLAN ID | VLAN Name | Mode   | Description                 |
| :------------- | :------ | :-------- | :----- | :-------------------------- |
| Fa0/1, Fa0/2   | 10      | HR        | Access | HR Department Workstations  |
| Fa0/3, Fa0/4   | 20      | IT        | Access | IT Department Workstations  |
| Fa0/6          | 30      | CEO       | Access | CEO Office                  |
| Fa0/7 - Fa0/24 | 1       | default   | Access | Default/General Use         |
| Gig0/1, Gig0/2 | 1       | default   | Access | Default/General Use         |

## Key Configuration Breakdown

This section highlights the most critical configuration blocks and explains their purpose.

### Global Configuration

```cisco
hostname Switch
!
spanning-tree mode pvst
spanning-tree extend system-id
```

*   `hostname Switch`: Assigns a descriptive name to the device, making it easier to identify in a network.
*   `spanning-tree mode pvst`: Enables Per-VLAN Spanning Tree (PVST), a Cisco proprietary STP mode that runs a separate instance of STP for each VLAN, allowing for load balancing across redundant links for different VLANs.
*   `spanning-tree extend system-id`: Automatically adds the VLAN ID to the bridge ID, preventing potential issues with bridge ID uniqueness in complex PVST+ environments.

### VLAN Definitions

The VLANs are defined by their assignment to interfaces, as no explicit `vlan <ID> name <NAME>` commands are shown in the provided running-config snippet. The `show vlan brief` output confirms the active VLANs and their names.

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

*   **VLAN 10 (HR):** Dedicated for the Human Resources department.
*   **VLAN 20 (IT):** Dedicated for the Information Technology department.
*   **VLAN 30 (CEO):** Dedicated for the Chief Executive Officer's office.
*   **VLAN 1 (default):** Catches all other unassigned ports.

These VLANs ensure that devices connected to ports within the same VLAN can communicate, while devices in different VLANs are isolated at Layer 2, enhancing security and reducing broadcast traffic.

### Access Port Configuration

```cisco
interface FastEthernet0/1
 switchport access vlan 10
 switchport mode access
```

*   `interface FastEthernet0/1`: Selects the specific FastEthernet port for configuration.
*   `switchport mode access`: Configures the port as an access port, meaning it will carry traffic for only a single VLAN and connect to an end device (e.g., a workstation, printer).
*   `switchport access vlan 10`: Assigns FastEthernet0/1 to VLAN 10 (HR). This command ensures that any device connected to this port will only be able to communicate within the HR VLAN. Similar commands would be applied to Fa0/2 for HR, Fa0/3-4 for IT, and Fa0/6 for CEO.

## Verification & Testing

To confirm the correct operation of the VLAN segmentation, a network engineer would execute the following commands.

### 1. Verify VLAN Database and Port Assignments

This command shows all defined VLANs, their names, and which ports are assigned to them, confirming that the logical segmentation is in place.

```cisco
Switch# show vlan brief
```

**Simulated Successful Output:**

```
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
1005 trnet-default                    active    
```

### 2. Verify Individual Access Port Configuration

This command provides detailed information about a specific port's switchport mode and access VLAN assignment, confirming that it is correctly configured for an end device.

```cisco
Switch# show interface FastEthernet0/1 switchport
```

**Simulated Successful Output:**

```
Name: Fa0/1
Switchport: Enabled
Administrative Mode: static access
Operational Mode: static access
Administrative Trunking Encapsulation: dot1q
Operational Trunking Encapsulation: native
Negotiation of Trunking: Off
Access Mode VLAN: 10 (HR)
Trunking Native Mode VLAN: 1 (default)
Administrative Native VLAN tagging: enabled
Voice VLAN: none
[...]
```

 and the spanning-tree protocol is actively preventing loops across the network segments.
