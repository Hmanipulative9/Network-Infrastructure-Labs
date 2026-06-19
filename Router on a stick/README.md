# Router On A Stick

## 🖼️ Network Topology
![Topology](router on a stick-topology.png)

```markdown
# 🚀 Production Lab Environment - Network Documentation

## 📄 Overview
This document outlines the network design and configuration for a critical production lab environment, engineered to provide robust, segmented network services. The architecture focuses on simulating a typical small branch office setup, ensuring secure and efficient communication paths for distinct user groups and services, while also providing connectivity to an "external" network segment.

The core infrastructure problem addressed is the need for traffic isolation and centralized routing within a shared physical network, alongside dynamic IP address allocation. By implementing VLANs for logical segmentation, a central router for inter-VLAN routing and DHCP services, and a dynamic routing protocol for reachability, the design ensures scalability, simplified management, and reliable layer 3 connectivity across all segments.

## 🛠️ Technical Stack
*   **Router-on-a-stick**: Centralized inter-VLAN routing on a single physical interface.
*   **802.1Q Trunking**: Industry-standard encapsulation for carrying multiple VLANs over a single link.
*   **OSPF (Open Shortest Path First)**: Dynamic Interior Gateway Protocol for efficient route advertisement.
*   **DHCP Services**: Centralized dynamic IP address allocation for end devices.
*   **VLANs (Virtual Local Area Networks)**: Layer 2 segmentation for traffic isolation.

## 📊 Addressing & Topology

| Device | Interface | IP Address/Subnet | VLAN | Role / Description |
| :----- | :-------- | :---------------- | :--- | :----------------- |
| **R1** | Gi0/0.10  | 192.168.10.1/24   | 10   | Users VLAN Gateway |
| **R1** | Gi0/0.20  | 192.168.20.1/24   | 20   | Servers VLAN Gateway |
| **R1** | Gi0/1     | 10.0.0.1/30       | N/A  | Uplink to R2 (Transit) |
| **SW1** | Vlan10    | N/A               | 10   | User Access VLAN |
| **SW1** | Vlan20    | N/A               | 20   | Server Access VLAN |
| **SW1** | Fa0/1     | N/A (Trunk)       | 10,20 | Trunk to R1 |
| **SW1** | Fa0/2-23  | N/A (Access)      | 10   | Access Ports for Users |
| **SW1** | Fa0/24    | N/A (Access)      | 20   | Access Port for Servers |
| **R2** | Gi0/0     | 10.0.0.2/30       | N/A  | Downlink to R1 (Transit) |
| **R2** | Loopback0 | 172.16.1.1/32     | N/A  | Simulated External Network ID |
| **PC1** | NIC       | 192.168.10.100/24 | 10   | User Workstation |
| **SVR1**| NIC       | 192.168.20.100/24 | 20   | Lab Server |

## ⚙️ Core Configurations

### Router (R1) - Inter-VLAN Routing & DHCP
```cisco
interface GigabitEthernet0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
 description --- Users VLAN Gateway ---

interface GigabitEthernet0/0.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
 description --- Servers VLAN Gateway ---

interface GigabitEthernet0/1
 ip address 10.0.0.1 255.255.255.252
 description --- Uplink to R2 ---

ip dhcp pool VLAN10_USERS
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8

ip dhcp pool VLAN20_SERVERS
 network 192.168.20.0 255.255.255.0
 default-router 192.168.20.1
 dns-server 8.8.8.8

router ospf 1
 network 10.0.0.0 0.0.0.3 area 0
 network 192.168.10.0 0.0.0.255 area 0
 network 192.168.20.0 0.0.0.255 area 0
```
*   **Sub-interfaces**: Configured for `dot1Q` encapsulation to route traffic for VLANs 10 and 20.
*   **Uplink Interface**: Directly connects to R2 for transit routing.
*   **DHCP Pools**: Provides dynamic IP addresses, default gateways, and DNS servers for VLAN 10 (Users) and VLAN 20 (Servers).
*   **OSPF**: Advertises connected networks, establishing dynamic routing with R2.

### Switch (SW1) - VLANs & Trunking
```cisco
vlan 10
 name USERS_VLAN
vlan 20
 name SERVERS_VLAN

interface FastEthernet0/1
 switchport trunk encapsulation dot1q
 switchport mode trunk
 description --- Trunk to R1 ---

interface FastEthernet0/2
 switchport mode access
 switchport access vlan 10
 description --- User Access Port ---

interface FastEthernet0/24
 switchport mode access
 switchport access vlan 20
 description --- Server Access Port ---
```
*   **VLAN Definitions**: Creates VLANs 10 and 20 for logical segmentation.
*   **Trunk Port**: Configures Fa0/1 to carry multiple VLANs to R1 using 802.1Q.
*   **Access Ports**: Assigns Fa0/2 for user devices in VLAN 10 and Fa0/24 for server devices in VLAN 20.

### Router (R2) - Edge Connectivity
```cisco
interface GigabitEthernet0/0
 ip address 10.0.0.2 255.255.255.252
 description --- Downlink to R1 ---

interface Loopback0
 ip address 172.16.1.1 255.255.255.255

router ospf 1
 network 10.0.0.0 0.0.0.3 area 0
 network 172.16.1.1 0.0.0.0 area 0
```
*   **Downlink Interface**: Connects to R1 for transit routing.
*   **Loopback Interface**: Represents a stable, simulated external network identifier.
*   **OSPF**: Advertises connected networks and the loopback interface, establishing dynamic routing with R1.

## 🧪 Connectivity & Diagnostics Verification

### Standard Verification Commands
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

Gateway of last resort is not set

     10.0.0.0/30 is subnetted, 1 subnets
C       10.0.0.0 is directly connected, GigabitEthernet0/1
     172.16.0.0/32 is subnetted, 1 subnets
O       172.16.1.1 [110/2] via 10.0.0.2, 00:00:25, GigabitEthernet0/1
     192.168.10.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.10.0/24 is directly connected, GigabitEthernet0/0.10
L       192.168.10.1/32 is directly connected, GigabitEthernet0/0.10
     192.168.20.0/24 is variably subnetted, 2 subnets, 2 masks
C       192.168.20.0/24 is directly connected, GigabitEthernet0/0.20
L       192.168.20.1/32 is directly connected, GigabitEthernet0/0.20

SW1# show vlan brief

VLAN Name                             Status    Ports
---- -------------------------------- --------- -------------------------------
1    default                          active    Fa0/3, Fa0/4, Fa0/5, Fa0/6
                                                Fa0/7, Fa0/8, Fa0/9, Fa0/10
                                                Fa0/11, Fa0/12, Fa0/13, Fa0/14
                                                Fa0/15, Fa0/16, Fa0/17, Fa0/18
                                                Fa0/19, Fa0/20, Fa0/21, Fa0/22
                                                Gi0/1, Gi0/2
10   USERS_VLAN                       active    Fa0/2
20   SERVERS_VLAN                     active    Fa0/24
1002 fddi-default                     act/unsup
1003 token-ring-default               act/unsup
1004 fddinet-default                  act/unsup
1005 trnet-default                    act/unsup

R1# show ip dhcp binding
IP address       Hardware address        Lease expiration        Type
192.168.10.100   001b.21a5.f040          Mar 01 2024 07:12 PM    Automatic
192.168.20.100   001a.33b2.e150          Mar 01 2024 07:13 PM    Automatic
```

### ICMP Ping Verification
```text
PC1 (192.168.10.100)> ping 192.168.20.100
Pinging 192.168.20.100 with 32 bytes of data:
Reply from 192.168.20.100: bytes=32 time=5ms TTL=127
Reply from 192.168.20.100: bytes=32 time=5ms TTL=127
Reply from 192.168.20.100: bytes=32 time=6ms TTL=127
Reply from 192.168.20.100: bytes=32 time=5ms TTL=127

Ping statistics for 192.168.20.100:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 5ms, Maximum = 6ms, Average = 5ms

PC1 (192.168.10.100)> ping 172.16.1.1
Pinging 172.16.1.1 with 32 bytes of data:
Reply from 172.16.1.1: bytes=32 time=9ms TTL=254
Reply from 172.16.1.1: bytes=32 time=9ms TTL=254
Reply from 172.16.1.1: bytes=32 time=10ms TTL=254
Reply from 172.16.1.1: bytes=32 time=9ms TTL=254

Ping statistics for 172.16.1.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 9ms, Maximum = 10ms, Average = 9ms
```
*   **Inter-VLAN Reachability**: A user PC in VLAN 10 successfully pings a server in VLAN 20, demonstrating successful inter-VLAN routing via R1.
*   **End-to-End External Reachability**: The user PC in VLAN 10 successfully pings the R2 Loopback interface (simulated external network), confirming full Layer 3 connectivity across the entire lab environment.
```
