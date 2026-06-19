# Dhcp Using Switch

## 🖼️ Network Topology
![Topology](dhcp using switch-topology.png)

## 📄 Overview: This deployment establishes a multi-VLAN routed network utilizing OSPF for dynamic routing and inter-VLAN communication, providing segmented network access and connectivity between internal and external segments.

## 🛠️ Core Capabilities:
*   **OSPFv2** for dynamic routing
*   **Inter-VLAN Routing (Router-on-a-Stick)** on a core router
*   **VLAN Trunking (802.1Q)** for multi-VLAN transport
*   **Static Default Routing** for internet edge connectivity

## 📊 Addressing Matrix:
| Device / Interface | IP Address     | VLAN / Role       |
| :----------------- | :------------- | :---------------- |
| **R1**             |                |                   |
| Loopback0          | 1.1.1.1        | OSPF Router ID    |
| Gi0/0              | 192.168.12.1   | P2P Link to R2    |
| Gi0/1.10           | 10.0.10.1      | VLAN 10 (Users)   |
| Gi0/1.20           | 10.0.20.1      | VLAN 20 (Servers) |
| **R2**             |                |                   |
| Loopback0          | 2.2.2.2        | OSPF Router ID    |
| Gi0/0              | 192.168.12.2   | P2P Link to R1    |
| Gi0/1              | 10.0.30.1      | VLAN 30 (DMZ)     |
| **DSW1**           |                |                   |
| VLAN 1             | 10.0.1.1       | Management SVI    |
| Gi0/1              | -              | Trunk to R1       |
| Gi0/2              | -              | Access VLAN 10    |
| Gi0/3              | -              | Access VLAN 20    |

## ⚙️ Infrastructure Blueprint (CLI):

```cisco
! R1 Configuration
hostname R1
!
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
!
interface GigabitEthernet0/0
 ip address 192.168.12.1 255.255.255.252
 no shutdown
!
interface GigabitEthernet0/1
 no ip address
 no shutdown
!
interface GigabitEthernet0/1.10
 encapsulation dot1Q 10
 ip address 10.0.10.1 255.255.255.0
!
interface GigabitEthernet0/1.20
 encapsulation dot1Q 20
 ip address 10.0.20.1 255.255.255.0
!
router ospf 1
 router-id 1.1.1.1
 network 1.1.1.1 0.0.0.0 area 0
 network 10.0.10.0 0.0.0.255 area 0
 network 10.0.20.0 0.0.0.255 area 0
 network 192.168.12.0 0.0.0.3 area 0
!
ip route 0.0.0.0 0.0.0.0 192.168.12.2
```

```cisco
! R2 Configuration
hostname R2
!
interface Loopback0
 ip address 2.2.2.2 255.255.255.255
!
interface GigabitEthernet0/0
 ip address 192.168.12.2 255.255.255.252
 no shutdown
!
interface GigabitEthernet0/1
 ip address 10.0.30.1 255.255.255.0
 no shutdown
!
router ospf 1
 router-id 2.2.2.2
 network 2.2.2.2 0.0.0.0 area 0
 network 10.0.30.0 0.0.0.255 area 0
 network 192.168.12.0 0.0.0.3 area 0
!
ip route 0.0.0.0 0.0.0.0 192.168.12.1
```

```cisco
! DSW1 Configuration
hostname DSW1
!
vlan 10
 name USERS
vlan 20
 name SERVERS
!
interface GigabitEthernet0/1
 switchport mode trunk
!
interface GigabitEthernet0/2
 switchport mode access
 switchport access vlan 10
!
interface GigabitEthernet0/3
 switchport mode access
 switchport access vlan 20
!
interface Vlan1
 ip address 10.0.1.1 255.255.255.0
 no shutdown
!
ip default-gateway 10.0.10.1
```

## 🧪 Verification Metrics:

```cisco
R1# show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides from PfR

Gateway of last resort is 192.168.12.2 to network 0.0.0.0

      1.0.0.0/32 is subnetted, 1 subnets
C        1.1.1.1 is directly connected, Loopback0
      10.0.0.0/8 is variably subnetted, 3 subnets, 2 masks
C        10.0.10.0/24 is directly connected, GigabitEthernet0/1.10
C        10.0.20.0/24 is directly connected, GigabitEthernet0/1.20
O        10.0.30.0/24 [110/2] via 192.168.12.2, 00:00:15, GigabitEthernet0/0
      192.168.12.0/24 is variably subnetted, 1 subnets, 1 masks
C        192.168.12.0/30 is directly connected, GigabitEthernet0/0
S*    0.0.0.0/0 [1/0] via 192.168.12.2
```

```cisco
R1# ping 10.0.30.1 source 10.0.10.1
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.0.30.1, timeout is 2 seconds:
Packet sent with a source address of 10.0.10.1
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms

R1# traceroute 10.0.30.1 source 10.0.10.1
Type escape sequence to abort.
Tracing the route to 10.0.30.1
VRF info: (vrf in global routing table)
  1 192.168.12.2 1 msec 1 msec 1 msec
  2 10.0.30.1 1 msec *  1 msec
```