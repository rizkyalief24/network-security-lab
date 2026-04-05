🔐 Network Simulation & Security Lab
Cisco Packet Tracer | VLAN Segmentation | Inter-VLAN Routing | ACL | DHCP | OSPF | ASA Firewall
---
📋 Project Overview
This project simulates a small-to-medium enterprise network environment built entirely in Cisco Packet Tracer. The topology replicates real-world corporate network architecture, implementing layered security through network segmentation, access control policies, and perimeter firewall deployment.
The primary objective is to demonstrate practical competency in network design, device configuration, and security policy enforcement — skills directly applicable to Network Engineer and IT Infrastructure roles.
---
🗺️ Network Topology
```
                    \[ ASA 5505 Firewall ]
                            |
                    \[ Cisco 2911 Router ]
                     (Inter-VLAN Routing)
                            |
                    \[ Cisco 2960 Switch ]
                   /         |          \\
          VLAN 10        VLAN 20       VLAN 30
        Perkantoran     Server Farm   Guest/DMZ
      192.168.10.0/24  192.168.20.0/24  192.168.30.0/24
```
---
🖥️ Device Inventory
Device	Model	Role
Firewall	Cisco ASA 5505	Perimeter security, traffic filtering
Router	Cisco 2911	Inter-VLAN routing (Router-on-a-Stick)
Switch	Cisco 2960-24TT	VLAN segmentation, trunk uplink
PC Kantor	PC-PT	End device — VLAN 10
Server Farm	Server-PT	Internal server — VLAN 20
PC Guest/DMZ	PC-PT	Untrusted end device — VLAN 30
---
📐 IP Addressing Plan
VLAN	Name	Network	Gateway	DHCP Range
VLAN 10	Perkantoran	192.168.10.0/24	192.168.10.1	192.168.10.2 – .254
VLAN 20	Server Farm	192.168.20.0/24	192.168.20.1	Static (192.168.20.2)
VLAN 30	Guest/DMZ	192.168.30.0/24	192.168.30.1	192.168.30.2 – .254
—	ASA ↔ Router	10.0.0.0/30	—	Static
---
⚙️ Features Implemented
1. VLAN Segmentation
Network traffic is isolated into three logical segments based on user/device classification. Each VLAN operates as an independent broadcast domain, reducing attack surface and improving traffic management.
```
Switch(config)# vlan 10
Switch(config-vlan)# name Kantor
Switch(config)# interface FastEthernet0/1
Switch(config-if)# switchport mode access
Switch(config-if)# switchport access vlan 10
```
2. Inter-VLAN Routing (Router-on-a-Stick)
A single physical uplink from the switch to the router is configured as a trunk port, carrying traffic from all VLANs. The router uses sub-interfaces with IEEE 802.1Q encapsulation to route traffic between VLANs.
```
Router(config)# interface GigabitEthernet0/1.10
Router(config-subif)# encapsulation dot1Q 10
Router(config-subif)# ip address 192.168.10.1 255.255.255.0
```
3. DHCP Server (per VLAN)
The router acts as a DHCP server, providing automatic IP assignment to each VLAN segment. Gateway addresses are excluded from the DHCP pool to prevent address conflicts.
```
Router(config)# ip dhcp pool VLAN10
Router(dhcp-config)# network 192.168.10.0 255.255.255.0
Router(dhcp-config)# default-router 192.168.10.1
Router(dhcp-config)# dns-server 8.8.8.8
Router(config)# ip dhcp excluded-address 192.168.10.1
```
4. Access Control List (ACL)
Extended named ACL applied inbound on the VLAN 30 sub-interface to enforce network segmentation policy. Guest/DMZ traffic is denied access to internal corporate and server networks.
```
Router(config)# ip access-list extended BLOCK\_GUEST
Router(config-ext-nacl)# deny ip 192.168.30.0 0.0.0.255 192.168.10.0 0.0.0.255
Router(config-ext-nacl)# deny ip 192.168.30.0 0.0.0.255 192.168.20.0 0.0.0.255
Router(config-ext-nacl)# permit ip any any
Router(config)# interface GigabitEthernet0/1.30
Router(config-subif)# ip access-group BLOCK\_GUEST in
```
5. OSPF Dynamic Routing
OSPF (Open Shortest Path First) is configured on the router to advertise all connected networks. This ensures scalability — when additional routers are introduced to the topology, routing updates propagate automatically without manual static route configuration.
```
Router(config)# router ospf 1
Router(config-router)# network 10.0.0.0 0.0.0.3 area 0
Router(config-router)# network 192.168.10.0 0.0.0.255 area 0
Router(config-router)# network 192.168.20.0 0.0.0.255 area 0
Router(config-router)# network 192.168.30.0 0.0.0.255 area 0
```
6. ASA 5505 Firewall
Cisco ASA 5505 deployed as perimeter firewall between the internal network and external-facing interface. Security zones defined with appropriate trust levels. Access control lists applied per interface direction to control inbound and outbound traffic flows.
```
ciscoasa(config)# interface vlan 2
ciscoasa(config-if)# nameif inside
ciscoasa(config-if)# security-level 100
ciscoasa(config)# access-list INSIDE\_OUT extended permit ip any any
ciscoasa(config)# access-group INSIDE\_OUT in interface inside
```
---
🔒 Security Policy Matrix
Source	Destination	Policy	Enforcement
VLAN 10 (Kantor)	VLAN 20 (Server)	✅ Permitted	Default routing
VLAN 10 (Kantor)	VLAN 30 (Guest)	✅ Permitted	Default routing
VLAN 30 (Guest)	VLAN 10 (Kantor)	❌ Denied	ACL BLOCK_GUEST
VLAN 30 (Guest)	VLAN 20 (Server)	❌ Denied	ACL BLOCK_GUEST
Any	Any (outbound)	✅ Permitted	ASA INSIDE_OUT
---
✅ Verification & Test Results
show vlan brief (Switch)
VLAN 10 → Fa0/1 (PC Kantor) ✅
VLAN 20 → Fa0/2 (Server Farm) ✅
VLAN 30 → Fa0/3 (PC Guest/DMZ) ✅
show ip interface brief (Router)
Gi0/0 → 10.0.0.2 — up/up ✅
Gi0/1.10 → 192.168.10.1 — up/up ✅
Gi0/1.20 → 192.168.20.1 — up/up ✅
Gi0/1.30 → 192.168.30.1 — up/up ✅
Ping Test — PC Kantor (VLAN 10)
```
ping 192.168.10.1 → Reply (0% loss) ✅
ping 192.168.20.1 → Reply (0% loss) ✅
ping 192.168.30.1 → Reply (0% loss) ✅
```
Ping Test — PC Guest/DMZ (VLAN 30) — ACL Verification
```
ping 192.168.10.2 → Destination host unreachable (100% loss) ✅
ping 192.168.20.2 → Destination host unreachable (100% loss) ✅
```
---
📁 Repository Structure
```
network-security-lab/
├── Project Portfolio.pkt       # Cisco Packet Tracer topology file
├── README.md                   # Project documentation
└── screenshots/
    ├── topology.png            # Network topology diagram
    ├── switch-vlan-brief.png   # VLAN segmentation verification
    ├── router-ip-brief.png     # Interface status verification
    ├── router-ip-route.png     # Routing table
    ├── asa-access-list.png     # Firewall ACL configuration
    ├── ping-kantor.png         # Connectivity test from VLAN 10
    └── ping-guest-blocked.png  # ACL enforcement verification
```
---
🧠 Lessons Learned
ASA 5505 architecture differs from standard routers — IP assignment is bound to VLAN interfaces, not physical ports, requiring switchport-to-VLAN mapping before interface configuration.
ACL placement matters — inbound ACL on the source VLAN sub-interface is more efficient than outbound ACL on the destination, as traffic is filtered before entering the routing process.
Router-on-a-Stick is a cost-effective inter-VLAN routing solution for small-to-medium environments, though it introduces a single point of congestion on the trunk uplink at scale.
OSPF in single-router topology advertises directly connected networks and prepares the routing infrastructure for horizontal scaling without reconfiguration.
---
🛠️ Tools Used
Cisco Packet Tracer 8.2.1
Cisco IOS CLI
GitHub (version control & documentation)
---
Built as part of a network engineering portfolio to demonstrate hands-on competency in enterprise network design and security implementation.
