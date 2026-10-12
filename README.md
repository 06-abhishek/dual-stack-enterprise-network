# Project Documentation: Dual-Stack Global FinTech Enterprise Architecture

## 1. Project Objective and Architectural Overview

This project involves the design and deployment of a highly available, dual-stack (IPv4/IPv6) 3-tier enterprise network in Cisco Packet Tracer. The topology simulates a financial services company, requiring strict segregation of duties, gateway redundancy, and secure branch communication over a simulated public internet.

The infrastructure is divided into four logical zones:

* **Zone 1: HQ Campus:** Utilizes a Cisco 3-Tier Campus Architecture (Core, Distribution, Access) focusing on high-speed switching, OSPF internal routing, HSRP, and corporate Wi-Fi.


* **Zone 2: Data Centre / Server Farm:** Hosts centralized IPv4/IPv6 DHCP, DNS, Web, and Syslog services, secured via ACLs.


* **Zone 3: Enterprise Edge & ISP Cloud:** Features dual-homed WAN connectivity utilizing external BGP (eBGP) peering and NAT Overload (PAT).


* **Zone 4: Remote Branch Office:** Securely connected to the HQ Data Centre via an IPsec-over-GRE Virtual Private Network (VPN) tunnel.



## 2. Hardware Provisioning and Modifications

The project utilizes specific Cisco hardware models, incorporating several engineering pivots to bypass hardware port limitations and simulator constraints.

* **Core Switches:** 2x Catalyst 3650-24PS (CORE-1, CORE-2) provisioned with AC-POWER-SUPPLY modules.


* **Distribution Switches:** 2x Catalyst 3650-24PS (DIST-1, DIST-2) provisioned with AC-POWER-SUPPLY modules.


* **Access Switches:** 4x Catalyst 2960-24TT (ACC-1, ACC-2, ACC-3, ACC-4).


* **Wireless:** 1x WLC-3504 and 2x LAP-PT. The LAP-PT units required physical AC power adapters because the 2960-24TT switches do not support Power over Ethernet (PoE).


* **Data Centre Gateway (Engineering Pivot):** The original ISR 4331 router lacked sufficient built-in Gigabit ports to establish a redundant OSPF connection to both Core switches. The hardware was swapped to a Cisco ISR 2911 router (DC-RTR), which natively features three GigabitEthernet ports, ensuring full High Availability without requiring additional hardware modules.


* **Edge & Branch Routers:** 3x ISR 4331 (EDGE-1, EDGE-2, BRANCH-RTR), each provisioned with a NIM-2T (2-Port Serial) module.


* **ISP Routers (Engineering Pivot):** 2x ISR 4331 (ISP-A, ISP-B). ISP-A required the installation of a second NIM-2T module in slot 0/2 to accommodate three simultaneous serial connections (to EDGE-1, ISP-B, and BRANCH-RTR).



## 3. Physical Cabling Matrix

The physical topology was wired using the following exact port specifications to ensure symmetrical redundancy.

### Core, Distribution, and Access Switching

* **CORE-1 (Gig1/0/1) to CORE-2 (Gig1/0/1):** Copper Cross-Over (L3 EtherChannel).


* **CORE-1 (Gig1/0/2) to CORE-2 (Gig1/0/2):** Copper Cross-Over (L3 EtherChannel).


* **DIST-1 (Gig1/0/1) to DIST-2 (Gig1/0/1):** Copper Cross-Over (L2 EtherChannel).


* **DIST-1 (Gig1/0/2) to DIST-2 (Gig1/0/2):** Copper Cross-Over (L2 EtherChannel).


* **CORE-1 to DIST-1 / DIST-2:** Copper Cross-Over links connecting CORE-1 (Gig1/0/3, Gig1/0/4) to DIST-1 (Gig1/0/3) and DIST-2 (Gig1/0/3).


* **CORE-2 to DIST-1 / DIST-2:** Copper Cross-Over links connecting CORE-2 (Gig1/0/4, Gig1/0/5) to DIST-1 (Gig1/0/4) and DIST-2 (Gig1/0/4).


* **DIST-1 to Access Layer:** Copper Straight-Through links connecting Gig1/0/5, Gig1/0/6, Gig1/0/7, and Gig1/0/8 to Gig0/1 on ACC-1, ACC-2, ACC-3, and ACC-4 respectively.


* **DIST-2 to Access Layer:** Copper Straight-Through redundant links connecting Gig1/0/5, Gig1/0/6, Gig1/0/7, and Gig1/0/8 to Gig0/2 on ACC-1, ACC-2, ACC-3, and ACC-4 respectively.



### Data Centre and Edge Routing

* **CORE-1 (Gig1/0/24) to EDGE-1 (Gig0/0/0):** Copper Straight-Through.


* **CORE-2 (Gig1/0/24) to EDGE-2 (Gig0/0/0):** Copper Straight-Through.


* **CORE-1 (Gig1/0/23) to DC-RTR (Gig0/0):** Copper Straight-Through (Primary OSPF Link).


* **CORE-2 (Gig1/0/23) to DC-RTR (Gig0/1):** Copper Straight-Through (Redundant OSPF Link).


* **DC-RTR (Gig0/2) to DC-SW (Gig0/1):** Copper Straight-Through.



### WAN, BGP, and Branch Links

* **EDGE-1 (Ser0/1/0) to ISP-A (Ser0/1/0):** Serial DCE.


* **EDGE-2 (Ser0/1/0) to ISP-B (Ser0/1/0):** Serial DCE.


* **ISP-A (Ser0/1/1) to BRANCH-RTR (Ser0/1/0):** Serial DCE.


* **ISP-B (Ser0/1/1) to ISP-A (Ser0/1/2):** Serial DCE.



### End-Devices and Wireless Networking

* **WLC-3504 (GigEthernet1) to ACC-4 (Fa0/24):** Copper Straight-Through.


* **LAP-PT 1 (GigEthernet0) to ACC-2 (Fa0/24):** Copper Straight-Through.


* **LAP-PT 2 (GigEthernet0) to ACC-3 (Fa0/24):** Copper Straight-Through.


* **User PCs:** HR-PC connected to ACC-1 (Fa0/1), FIN-PC connected to ACC-2 (Fa0/1), IT-PC connected to ACC-3 (Fa0/1), and MGMT-PC connected to ACC-4 (Fa0/1).



## 4. Dual-Stack IP Subnetting Scheme

The network was designed with a logically summarized, scalable dual-stack IP schema to support both IPv4 and IPv6 concurrently.

### HQ Campus VLANs (Distribution SVI Gateways)

* **VLAN 10 (HR):** `10.1.10.0/24` (GW: .254) | `2001:db8:1:10::/64` (GW: ::1)


* **VLAN 20 (Finance):** `10.1.20.0/24` (GW: .254) | `2001:db8:1:20::/64` (GW: ::1)


* **VLAN 30 (IT):** `10.1.30.0/24` (GW: .254) | `2001:db8:1:30::/64` (GW: ::1)


* **VLAN 50 (Wi-Fi):** `10.1.50.0/24` (GW: .254) | `2001:db8:1:50::/64` (GW: ::1)


* **VLAN 99 (Management):** `10.1.99.0/24` (GW: .254) | `2001:db8:1:99::/64` (GW: ::1)



### LAN and Point-to-Point Routing Subnets

* **Data Centre LAN:** `10.2.0.0/24` | `2001:db8:2::/64`

* **Remote Branch LAN:** `192.168.10.0/24` | `2001:db8:3::/64`

* **CORE-1 to CORE-2:** `10.255.255.16/30` | `2001:db8:A::10/126`

* **CORE-1 to DIST-1:** `10.255.255.20/30` | `2001:db8:A::14/126`

* **CORE-1 to DIST-2:** `10.255.255.24/30` | `2001:db8:A::18/126`

* **CORE-2 to DIST-1:** `10.255.255.28/30` | `2001:db8:A::1C/126`

* **CORE-2 to DIST-2:** `10.255.255.32/30` | `2001:db8:A::20/126`

* **CORE-1 to EDGE-1:** `10.255.255.0/30` | `2001:db8:A::0/126`

* **CORE-2 to EDGE-2:** `10.255.255.4/30` | `2001:db8:A::4/126`

* **CORE-1 to DC-RTR:** `10.255.255.8/30` | `2001:db8:A::8/126`

* **CORE-2 to DC-RTR:** `10.255.255.12/30` | `2001:db8:A::C/126`


### Public WAN, GRE, and BGP Subnets

* **EDGE-1 to ISP-A:** `203.0.113.0/30` | `2001:db8:B:1::0/126`

* **EDGE-2 to ISP-B:** `198.51.100.0/30` | `2001:db8:B:2::0/126`

* **ISP-A to ISP-B:** `8.8.8.0/30` | `2001:db8:B:3::0/126`

* **ISP-A to BRANCH-RTR:** `200.1.1.0/30` | `2001:db8:B:4::0/126`

* **GRE Tunnel (Edge-1 to Branch):** `172.16.0.0/30` | `2001:db8:C::0/126`


## 5. Implementation Phases & Architecture Pivots

### 5.1 Layer 2 Switching Foundation

* IPv4 and IPv6 routing capabilities were globally enabled on Core and Distribution switches.


* DIST-1 was configured as the VTP Server for domain `FINTECH`, synchronizing VLANs 10, 20, 30, 50, and 99 down to all Access layer VTP clients.


* Uplinks between the Access and Distribution layers were configured as 802.1Q Trunks, restricted to native VLAN 99.


* A Layer 2 EtherChannel using LACP (Port-Channel 1) was bundled between DIST-1 and DIST-2 to facilitate HSRP synchronization.


* Access ports facing end-user PCs were assigned to their respective VLANs, and Spanning-Tree PortFast was enabled to bypass the listening/learning phases, ensuring rapid DHCP acquisition.



### 5.2 Layer 3 Gateways & First-Hop Redundancy (HSRP)

* Switch Virtual Interfaces (SVIs) were generated on DIST-1 and DIST-2 for all VLANs.


* Physical uplinks pointing toward the Core layer on the Distribution switches were converted from switchports to Layer 3 routed ports.


* HSRP Load Balancing was implemented: DIST-1 was configured with increased priority (and preemption) to act as the Active gateway for VLANs 10 and 20. DIST-2 was configured as the Active gateway for VLANs 30 and 50.


* **Engineering Pivot (Simulator Constraint):** Packet Tracer 9.0 lacks the software support for IPv6 First-Hop Redundancy Protocols (HSRP/VRRP). The architecture was adjusted to abandon IPv6 HSRP; the topology relies exclusively on IPv4 HSRP for gateway redundancy demonstration, with static IPv6 SLAAC addresses placed on the SVIs.


* **Engineering Pivot (EtherChannel Order of Operations):** When attempting to assign an IP address to the Core-to-Core EtherChannel, a "Not a convertible port" error occurred. The port-channel was destroyed, the physical ports were explicitly converted to routed ports (`no switchport`), and the bundle was recreated at Layer 3 to resolve the software bug.



### 5.3 Internal Dynamic Routing (OSPFv2 and OSPFv3)

* OSPF was activated across the Core, Distribution, Edge, and DC-RTR devices.


* **Engineering Pivot (Dual-Stack OSPF Crash Prevention):** To prevent OSPFv3 from crashing due to the lack of a 32-bit name, both the IPv4 (`router ospf 1`) and IPv6 (`ipv6 router ospf 1`) processes were strictly assigned identical, hardcoded Router IDs (e.g., `1.1.1.1` on CORE-1).


* OSPFv2 was enabled via global network wildcard commands, while OSPFv3 was enabled directly at the interface level (`ipv6 ospf 1 area 0`).


* **Engineering Pivot (Routing Black Hole Fix):** SVI interfaces on the Distribution switches were initially set as `passive-interface` to prevent Hello packets from reaching end-users. This inadvertently severed the OSPF adjacency between DIST-1 and DIST-2. The configuration was corrected by explicitly removing the passive designation (`no passive-interface`) on the Native Management VLAN 99, allowing the Distribution switches to form a FULL adjacency across the Layer 2 EtherChannel.



### 5.4 Centralized Network Services (DHCP & Syslog)

* A dedicated physical DHCP Server was deployed in the Data Centre with separate pools for VLANs 10, 20, 30, and 50.


* DHCP Relay (`ip helper-address`) was configured on all SVI interfaces at the Distribution layer to intercept client broadcasts and route them to the Data Centre server.


* A centralized Syslog server was activated, with Core, Distribution, and Edge devices configured to forward NOC monitoring logs to its static IP.



### 5.5 WAN Edge, NAT Overload, and BGP Peering

* External BGP (eBGP) peering was established between EDGE-1 (Private AS 65000) and ISP-A (AS 100), enabling the HQ to learn the simulated internet routes (e.g., Google's `8.8.8.8` DNS).


* NAT Overload (Port Address Translation) was applied to EDGE-1, mapping an Access Control List (ACL) of the internal `10.0.0.0/8` network to the outside Serial interface facing the ISP.


* A static default route (`0.0.0.0 0.0.0.0`) was injected into the internal OSPF domain using the `default-information originate` command, successfully allowing internal VLAN PCs to ping public internet addresses.



### 5.6 Secure Site-to-Site VPN (GRE over IPsec)

* A logical GRE Tunnel (`Tunnel 0`) was configured between EDGE-1 and BRANCH-RTR, encapsulating private `192.168.10.0/24` branch traffic across the untrusted public serial links.


* The private tunnel subnet (`172.16.0.0/30`) and Branch LAN were advertised into OSPF Area 0, ensuring the HQ and Branch could route internally without advertising public ISP IP addresses. NAT bypass was achieved by intentionally omitting `ip nat outside` from the Tunnel interface.


* **Engineering Pivot (Simulator Constraint):** The Cisco 4331 emulator in Packet Tracer does not support modern IPsec Profiles (`crypto ipsec profile`). The VPN was rebuilt using legacy Crypto Maps applied directly to the physical Serial WAN interfaces.


* **Engineering Pivot (100% Packet Loss Fix):** Initial ping tests across the tunnel failed due to an asymmetrical cryptography configuration. The ISAKMP policy (AES encryption, SHA hashing, Diffie-Hellman Group 2), Pre-Shared Key, and Transform-Set were missing on BRANCH-RTR. Upon mirroring the Phase 1 and Phase 2 configurations across both routers, the IPsec Security Associations established successfully, allowing the Remote Branch PC to securely ping the Data Centre Web Server.



### 5.7 Corporate Wireless LAN Integration

* A Cisco WLC-3504 was configured via its GUI Dashboard with a Management IP of `10.1.50.10` residing in VLAN 50.


* A WPA2 Personal WLAN network (`FINTECH-CORP`) was provisioned on the controller.


* Lightweight Access Points (LAPs) dynamically retrieved DHCP addresses from the Data Centre, located the WLC, and established CAPWAP tunnels to broadcast the corporate SSID.


* **Engineering Pivot (WLC CAPWAP Refresh):** The Packet Tracer initial setup wizard failed to bind the original SSID to the Access Point radios. A secondary WLAN was created to force a CAPWAP configuration refresh, resolving the broadcast issue. Additionally, hardcoded static IPs on testing laptops were removed in favor of DHCP, resolving a simulated L1 helpdesk connectivity ticket and completing the enterprise architecture.
