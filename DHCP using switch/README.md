# Dhcp Using Switch

## 🖼️ Network Topology
![Topology](dhcp using switch-topology.png)

```markdown
# Network Infrastructure Deployment

## 📄 Overview
This deployment establishes multi-VLAN segmentation and inter-VLAN routing across two core routers and a layer-2 switch. The goal is to provide secure, isolated network access for distinct departmental functions (e.g., HR, Finance, IT, Guest) while enabling controlled communication.

## 🛠️ Core Capabilities
*   **VLAN Tagging (IEEE 802.1Q):** Network segmentation for departmental traffic isolation.
*   **Router-on-a-Stick (RoaS):** Centralized inter-VLAN routing services on dedicated router interfaces.
*   **Open Shortest Path First (OSPFv2):** Dynamic routing protocol for inter-router reachability and route convergence.
*   **Static Routing:** Default gateway for internet egress.

## 📊 Addressing & VLAN Matrix

| Device | Interface | IP Address/Mask | VLAN ID | Role/Description |
| :----- | :-------- | :-------------- | :------ | :--------------- |
| **R1** | Gi0/0     | 10.0.0.1/30     | N/A     | OSPF Link to R2  |
| **R1** | Gi0/1.10  | 192.168.10.1/24 | 10      | HR Gateway       |
| **R1** | Gi0/1.20  | 192.168.20.1/24 | 20      | Finance Gateway  |
| **R1** | Gi0/2     | 203.0.113.1/30  | N/A     | WAN Uplink       |
| **R2** | Gi0/0.30  | 192.168.30.1/24 | 30      | IT Gateway       |
| **R2** | Gi0/0.40  | 192.168.40.1/24 | 40      | Guest Gateway    |
| **R2** | Gi0/1     | 10.0.0.2/30     | N/A     | OSPF Link to R1  |
| **SW1**| Gi0/1     | N/A             | 10,20   | Trunk to R1      |
| **SW1**| Gi0/2     | N/A             | 30,40   | Trunk to R2      |
| **SW1**| Gi0/3     | N/A             | 10      | HR Access Port   |
| **SW1**| Gi0/4     | N/A             | 20      | Finance Access   |
| **SW1**| Gi0/5     | N/A             | 30      | IT Access Port   |
| **SW1**| Gi0/6     | N/A             | 40      | Guest Access Port|

## ⚙️ Infrastructure Blueprint (CLI)

#### R1: VLAN 10/20 Inter-VLAN Gateway
```cli
interface GigabitEthernet0/1
 no ip address
!
interface GigabitEthernet0/1.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
!
interface GigabitEthernet0/1.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
```

#### R1: OSPF & Default Route
```cli
interface GigabitEthernet0/0
 ip address 10.0.0.1 255.255.255.252
!
interface GigabitEthernet0/2
 ip address 203.0.113.1 255.255.255.252
!
router ospf 1
 network 10.0.0.0 0.0.0.3 area 0
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0
 default-information originate
!
ip route 0.0.0.0 0.0.0.0 203.0.113.2
```

#### R2: VLAN 30/40 Inter-VLAN Gateway
```cli
interface GigabitEthernet0/0
 no ip address
!
interface GigabitEthernet0/0.30
 encapsulation dot1Q 30
 ip address 192.168.30.1 255.255.255.0
!
interface GigabitEthernet0/0.40
 encapsulation dot1Q 40
 ip address 192.168.40.1 255.255.255.0
```

#### R2: OSPF Internal Routing
```cli
interface GigabitEthernet0/1
 ip address 10.0.0.2 255.255.255.252
!
router ospf 1
 network 10.0.0.0 0.0.0.3 area 0
 network 192.168.30.0 0.0.0.255 area 0
 network 192.168.40.0 0.0.0.255 area 0
```

#### SW1: VLAN & Trunk Configuration
```cli
vlan 10
 name HR
vlan 20
 name Finance
vlan 30
 name IT
vlan 40
 name Guest
!
interface GigabitEthernet0/1
 switchport mode trunk
!
interface GigabitEthernet0/2
 switchport mode trunk
```

#### SW1: Access Port Assignments
```cli
interface GigabitEthernet0/3
 switchport mode access
 switchport access vlan 10
!
interface GigabitEthernet0/4
 switchport mode access
 switchport access vlan 20
!
interface GigabitEthernet0/5
 switchport mode access
 switchport access vlan 30
!
interface GigabitEthernet0/6
 switchport mode access
 switchport access vlan 40
```

## 🧪 Verification Metrics

```cli
R1# show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides default

Gateway of last resort is 203.0.113.2 to network 0.0.0.0

S*    0.0.0.0/0 [1/0] via 203.0.113.2
      10.0.0.0/8 is variably subnetted, 2 subnets, 2 masks
C        10.0.0.0/30 is directly connected, GigabitEthernet0/0
L        10.0.0.1/32 is directly connected, GigabitEthernet0/0
      192.168.10.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.10.0/24 is directly connected, GigabitEthernet0/1.10
L        192.168.10.1/32 is directly connected, GigabitEthernet0/1.10
      192.168.20.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.20.0/24 is directly connected, GigabitEthernet0/1.20
L        192.168.20.1/32 is directly connected, GigabitEthernet0/1.20
O     192.168.30.0/24 [110/2] via 10.0.0.2, 00:00:30, GigabitEthernet0/0
O     192.168.40.0/24 [110/2] via 10.0.0.2, 00:00:30, GigabitEthernet0/0
      203.0.113.0/30 is variably subnetted, 2 subnets, 2 masks
C        203.0.113.0/30 is directly connected, GigabitEthernet0/2
L        203.0.113.1/32 is directly connected, GigabitEthernet0/2

R1# traceroute 192.168.40.10 source 192.168.10.10
Type escape sequence to abort.
Tracing the route to 192.168.40.10
VRF info: (vrf in name/id, vrf out name/id)
  1 192.168.10.1 0 msec 0 msec 0 msec
  2 10.0.0.2 1 msec 0 msec 0 msec
  3 192.168.40.1 1 msec 0 msec 0 msec
  4 192.168.40.10 1 msec *  0 msec
```
```
```