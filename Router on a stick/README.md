# Router On A Stick

## 🖼️ Network Topology
![Topology](router on a stick-topology.png)

## 📄 Overview: This deployment establishes a robust, OSPF-driven routed network supporting multiple VLAN-segmented user domains.

## 🛠️ Core Capabilities:
*   **OSPFv2**: Dynamic Interior Gateway Protocol for route advertisement.
*   **VLANs**: Layer 2 network segmentation for different user groups.
*   **Inter-VLAN Routing**: Layer 3 forwarding between segmented VLANs.
*   **Routed Interfaces**: Direct Layer 3 links between network devices.
*   **Loopback Interfaces**: Stable router identification and reachability testing.

## 📊 Addressing Matrix:

| Device / Interface | IP Address      | VLAN / Role      |
| :----------------- | :-------------- | :--------------- |
| **R1**             |                 |                  |
| Loopback0          | 1.1.1.1/32      | Router ID        |
| Gi0/0              | 10.0.0.1/30     | Link to R2       |
| Gi0/1              | 10.0.1.1/30     | Link to SW1      |
| **R2**             |                 |                  |
| Loopback0          | 2.2.2.2/32      | Router ID        |
| Gi0/0              | 10.0.0.2/30     | Link to R1       |
| **SW1**            |                 |                  |
| VLAN 10            | 192.168.10.1/24 | User Segment     |
| VLAN 20            | 192.168.20.1/24 | Server Segment   |
| Gi0/1              | 10.0.1.2/30     | Link to R1       |

## ⚙️ Infrastructure Blueprint (CLI):

```cisco
! Router R1 Configuration
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
!
interface GigabitEthernet0/0
 ip address 10.0.0.1 255.255.255.252
 no shutdown
!
interface GigabitEthernet0/1
 ip address 10.0.1.1 255.255.255.252
 no shutdown
!
router ospf 1
 router-id 1.1.1.1
 network 1.1.1.1 0.0.0.0 area 0
 network 10.0.0.0 0.0.0.3 area 0
 network 10.0.1.0 0.0.0.3 area 0
```

```cisco
! Router R2 Configuration
interface Loopback0
 ip address 2.2.2.2 255.255.255.255
!
interface GigabitEthernet0/0
 ip address 10.0.0.2 255.255.255.252
 no shutdown
!
router ospf 1
 router-id 2.2.2.2
 network 2.2.2.2 0.0.0.0 area 0
 network 10.0.0.0 0.0.0.3 area 0
```

```cisco
! Layer 3 Switch SW1 Configuration
vlan 10
 name Users_VLAN10
!
vlan 20
 name Servers_VLAN20
!
interface GigabitEthernet0/1
 no switchport
 ip address 10.0.1.2 255.255.255.252
 no shutdown
!
interface GigabitEthernet0/2
 switchport mode access
 switchport access vlan 10
!
interface GigabitEthernet0/3
 switchport mode access
 switchport access vlan 20
!
interface Vlan10
 ip address 192.168.10.1 255.255.255.0
 no shutdown
!
interface Vlan20
 ip address 192.168.20.1 255.255.255.0
 no shutdown
!
router ospf 1
 router-id 3.3.3.3
 network 10.0.1.0 0.0.0.3 area 0
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0
 passive-interface Vlan10
 passive-interface Vlan20
```

## 🧪 Verification Metrics:

```
SW1# show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides default

Gateway of last resort is not set

      1.0.0.0/32 is subnetted, 1 subnets
O     1.1.1.1 [110/2] via 10.0.1.1, 00:00:25, GigabitEthernet0/1
      2.0.0.0/32 is subnetted, 1 subnets
O     2.2.2.2 [110/3] via 10.0.1.1, 00:00:25, GigabitEthernet0/1
      10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
C        10.0.1.0/30 is directly connected, GigabitEthernet0/1
L        10.0.1.2/32 is directly connected, GigabitEthernet0/1
O        10.0.0.0/30 [110/2] via 10.0.1.1, 00:00:25, GigabitEthernet0/1
      192.168.10.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.10.0/24 is directly connected, Vlan10
L        192.168.10.1/32 is directly connected, Vlan10
      192.168.20.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.20.0/24 is directly connected, Vlan20
L        192.168.20.1/32 is directly connected, Vlan20
```

```
SW1# traceroute 2.2.2.2 source Vlan10
Type escape sequence to abort.
Tracing the route to 2.2.2.2
VRF info: (vrf in name/id, vrf out name/id)
  1 10.0.1.1 1 msec 1 msec 0 msec
  2 10.0.0.2 1 msec 0 msec 1 msec
  3 2.2.2.2 0 msec *  1 msec
```