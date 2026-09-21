\# Enterprise Campus Network Infrastructure (High Availability \& Redundant WAN)



Designed and implemented a resilient dual-core enterprise campus topology in Cisco Packet Tracer. The design focuses on eliminating Single Points of Failure (SPOF) across Layer 2 and Layer 3, implementing HSRP first-hop gateway redundancy, dynamic routing via OSPFv2, and stateful NAT/PAT across redundant ISP links.



!\[Topology Diagram](topology.png)



\---



\## Technical Specifications \& Protocols



\* \*\*Switching (L2):\*\* 

&#x20; \* 802.1Q VLAN Trunking (VLAN 10: IT, VLAN 20: HR)

&#x20; \* LACP (802.3ad) Port-Channel 1 between Core distribution switches (200 Mbps aggregate link)

&#x20; \* PVST+ Root Bridge optimization (Core-SW1 Priority: 4096, Core-SW2 Priority: 8192)

&#x20; \* Spanning-Tree PortFast enabled on end-user access ports



\* \*\*Routing \& Redundancy (L3):\*\*

&#x20; \* Router-on-a-Stick (ROAS) with sub-interface encapsulation

&#x20; \* HSRP (v1) deployed for Virtual Default Gateways:

&#x20;   \* VLAN 10 VIP: `192.168.10.1` (Edge-R1 Priority: 110, Preempt | Edge-R2 Priority: 100)

&#x20;   \* VLAN 20 VIP: `192.168.20.1` (Edge-R1 Priority: 110, Preempt | Edge-R2 Priority: 100)

&#x20; \* OSPFv2 (Area 0) running on dedicated point-to-point interconnect (`10.0.0.0/30`) for state sync



\* \*\*Edge \& WAN Access:\*\*

&#x20; \* Dual-homed WAN connections to ISP Router

&#x20; \* NAT Overload (PAT) bound to WAN interfaces for RFC 1918 translation

&#x20; \* Static Default Routing pointing to upstream ISP next-hops



\---



\## IP Addressing Plan



| Device | Interface | IP Address | Subnet Mask | Role / VIP |

| :--- | :--- | :--- | :--- | :--- |

| \*\*Edge-R1\*\* | `Gi0/0.10` | 192.168.10.2 | 255.255.255.0 | VLAN 10 Active GW (VIP: 192.168.10.1) |

| \*\*Edge-R1\*\* | `Gi0/0.20` | 192.168.20.2 | 255.255.255.0 | VLAN 20 Active GW (VIP: 192.168.20.1) |

| \*\*Edge-R1\*\* | `Gi0/1` | 198.51.100.2 | 255.255.255.252 | Primary WAN Uplink |

| \*\*Edge-R1\*\* | `Gi0/2` | 10.0.0.1 | 255.255.255.252 | OSPF Area 0 Backbone Link |

| \*\*Edge-R2\*\* | `Gi0/0.10` | 192.168.10.3 | 255.255.255.0 | VLAN 10 Standby GW |

| \*\*Edge-R2\*\* | `Gi0/0.20` | 192.168.20.3 | 255.255.255.0 | VLAN 20 Standby GW |

| \*\*Edge-R2\*\* | `Gi0/1` | 198.51.100.6 | 255.255.255.252 | Backup WAN Uplink |

| \*\*Edge-R2\*\* | `Gi0/2` | 10.0.0.2 | 255.255.255.252 | OSPF Area 0 Backbone Link |

| \*\*ISP-Router\*\* | `Gi0/0` | 198.51.100.1 | 255.255.255.252 | ISP Link to Edge-R1 |

| \*\*ISP-Router\*\* | `Gi0/1` | 198.51.100.5 | 255.255.255.252 | ISP Link to Edge-R2 |

| \*\*ISP-Router\*\* | `Gi0/2` | 8.8.8.1 | 255.255.255.0 | Public Web Segment |

| \*\*Web Server\*\* | `Fa0` | 8.8.8.8 | 255.255.255.0 | Simulated Public Web Destination |



\---



\## CLI Verification \& State Output



\### 1. HSRP State Check (Edge-R1)

```text

Edge-R1# show standby brief

Interface   Grp  Pri P State   Active          Standby         Virtual IP

Gig0/0.10   10   110 P Active  local           192.168.10.3    192.168.10.1

Gig0/0.20   20   110 P Active  local           192.168.20.3    192.168.20.1

