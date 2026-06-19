# Router On A Stick

## 🖼️ Network Topology
![Topology](router on a stick-topology.png)

```markdown
# Network Segmentation & Inter-VLAN Routing Deployment

## 📄 Overview
This deployment establishes multi-VLAN network segmentation and inter-VLAN routing on a single infrastructure stack. It facilitates secure communication between distinct broadcast domains for various user and service groups.

## 🛠️ Core Capabilities
*   IEEE 802.1Q VLAN Trunking
*   Router-on-a-stick Inter-VLAN Routing
*   DHCP Address Assignment
*   Spanning Tree Protocol (PVST+)
*   Standard IP Routing

## 📊 Addressing & VLAN Matrix
| Device | Interface | IP Address/Mask | VLANs/Encapsulation | Description |
|---|---|---|---|---|
| **R1** | Gi0/0.10 | 192.168.10.1/24 | dot1Q 10 | User VLAN Gateway |
| | Gi0/0.20 | 192.168.20.1/24 | dot1Q 20 | Server VLAN Gateway |
| | Gi0/0.99 | 192.168.99.1/24 | dot1Q 99 | Management VLAN Gateway |
| | Gi0/0 | N/A | Trunk | Physical Uplink Interface |
| **SW1** | Gi0/1 | N/A | Trunk (allowed 10,20,99) | Router Uplink |
| | Gi0/2 | N/A | Access VLAN 10 | User Access Port |
| | Gi0/3 | N/A | Access VLAN 20 | Server Access Port |
| | Vlan99 | 192.168.99.2/24 | SVI Management | Switch Management IP |

## ⚙️ Infrastructure Blueprint (CLI)

#### Router Sub-interfaces (R1)
```cisco
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
interface GigabitEthernet0/0.99
 encapsulation dot1Q 99
 ip address 192.168.99.1 255.255.255.0
```

#### DHCP Address Pools (R1)
```cisco
ip dhcp pool VLAN10_USERS
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8 8.8.4.4
ip dhcp pool VLAN20_SERVERS
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 8.8.8.8 8.8.4.4
```

#### Switch Trunk & Access Ports (SW1)
```cisco
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,99
interface GigabitEthernet0/2
 switchport mode access
 switchport access vlan 10
interface GigabitEthernet0/3
 switchport mode access
 switchport access vlan 20
```

#### Switch Management SVI (SW1)
```cisco
interface Vlan99
 ip address 192.168.99.2 255.255.255.0
 no shutdown
```

## 🧪 Verification Metrics

#### Router Routing Table (R1)
```
R1#show ip route
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       a - application route
       + - replicated route, % - next hop override, p - overrides from PfR

Gateway of last resort is not set

      192.168.10.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.10.0/24 is directly connected, GigabitEthernet0/0.10
L        192.168.10.1/32 is directly connected, GigabitEthernet0/0.10
      192.168.20.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.20.0/24 is directly connected, GigabitEthernet0/0.20
L        192.168.20.1/32 is directly connected, GigabitEthernet0/0.20
      192.168.99.0/24 is variably subnetted, 2 subnets, 2 masks
C        192.168.99.0/24 is directly connected, GigabitEthernet0/0.99
L        192.168.99.1/32 is directly connected, GigabitEthernet0/0.99
```

#### End-to-End ICMP Trace (Client in VLAN 10 to Server in VLAN 20)
```
C:\>tracert 192.168.20.20

Tracing route to 192.168.20.20 over a maximum of 30 hops:

  1    <1 ms    <1 ms    <1 ms  192.168.10.1
  2    <1 ms    <1 ms    <1 ms  192.168.20.20

Trace complete.
```