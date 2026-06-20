# Dhcp Using Switch

## 🖼️ Network Topology
![Topology](dhcp-using-switch-topology.png)

## 📄 Overview:
This configuration deploys a multi-VLAN Layer 2 network segmentation with integrated DHCP services.

## 🛠️ Core Capabilities:
*   VLAN Segmentation (802.1Q)
*   Static Trunking
*   DHCP Server (implied by bindings)

## 📊 Addressing Matrix:

| Device / Interface | IP Address | VLAN / Role |
|--------------------|------------|-------------|
| **Switch** (Management) | - | Layer 2 Switch |
| FastEthernet0/1    | -          | Trunk (VLANs 1, 10, 20) |
| FastEthernet0/2    | -          | Trunk (VLANs 1, 10, 20) |
| FastEthernet0/3-24, Gig0/1-2 | -          | Access (VLAN 1) |
| VLAN 1 (default)   | -          | Default Data Network |
| VLAN 10 (it)       | -          | IT Services Network |
| VLAN 20 (HR)       | -          | HR Services Network |

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
interface range FastEthernet0/3-24, GigabitEthernet0/1-2
 switchport mode access
 switchport access vlan 1
```

## 🧪 Verification Metrics:

**Routing Table:**
This device is configured as a Layer 2 switch with no IP routing enabled or Layer 3 interfaces.

```cisco
Switch#show ip route
% IP routing not enabled
```

**End-to-End ICMP Ping Trace:**
This device is configured as a Layer 2 switch and does not have an IP address for management or any Layer 3 interface. Consequently, it cannot originate or terminate ICMP ping requests to demonstrate end-to-end Layer 3 reachability from its own CLI. Layer 3 verification would need to be performed from connected end devices or a connected router.