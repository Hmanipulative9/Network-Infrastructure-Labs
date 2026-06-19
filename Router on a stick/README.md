# Router On A Stick

## 🖼️ Network Topology
![Topology](router on a stick-topology.png)

```markdown
## 📄 Overview:
This deployment establishes inter-VLAN routing services between distinct network segments. Its business use-case enables secure logical separation and communication for various departmental networks on shared physical infrastructure.

## 🛠️ Core Capabilities:
*   Inter-VLAN Routing (Router-on-a-Stick)
*   IEEE 802.1Q VLAN Tagging
*   Dynamic Host Configuration Protocol (DHCP)
*   Ethernet Trunking

## 📊 Addressing & VLAN Matrix:

| Device | Interface           | IP Address/Mask       | VLAN ID   | Role                 |
| :----- | :------------------ | :-------------------- | :-------- | :------------------- |
| R1     | GigabitEthernet0/0.10 | 192.168.10.1/24       | 10        | Default Gateway      |
| R1     | GigabitEthernet0/0.20 | 192.168.20.1/24       | 20        | Default Gateway      |
| R1     | GigabitEthernet0/0  | No IP (Parent)        | (Trunk)   | Inter-VLAN L3 Trunk  |
| SW1    | GigabitEthernet0/1  | (N/A)                 | (Trunk)   | Router-Link Trunk    |
| SW1    | GigabitEthernet0/2  | (N/A)                 | 10        | VLAN 10 Access Port  |
| SW1    | GigabitEthernet0/3  | (N/A)                 | 20        | VLAN 20 Access Port  |

## ⚙️ Infrastructure Blueprint (CLI):

**Router Sub-Interface VLAN Tagging**
```cli
interface GigabitEthernet0/0
 no ip address

interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0

interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
```

**Router DHCP Pool Configuration**
```cli
ip dhcp pool VLAN10_POOL
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
```

**Switch Trunk Port Configuration**
```cli
interface GigabitEthernet0/1
 switchport mode trunk
```

**Switch Access Port VLAN Assignment**
```cli
interface GigabitEthernet0/2
 switchport mode access
 switchport access vlan 10

interface GigabitEthernet0/3
 switchport mode access
 switchport access vlan 20
```

## 🧪 Verification Metrics:

```cli
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
```

```cli
PC-VLAN10> traceroute 192.168.20.10

Tracing route to 192.168.20.10 over a maximum of 30 hops:

  1    <1 ms    <1 ms    <1 ms  192.168.10.1
  2     1 ms     1 ms     1 ms  192.168.20.10

Trace complete.
```