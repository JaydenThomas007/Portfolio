# Redundant Small Enterprise Network Architecture

A high-availability, three-tier hierarchical network implementation focusing on redundancy, traffic engineering, and granular security.

## Project Overview

This project simulates a small enterprise network environment designed for zero-downtime. It utilizes a **Three-Tier Hierarchical Model** (Core, Distribution, Access) to ensure scalability and manageability. By combining **FHRP (VRRP)** and **Multi-Instance Spanning Tree (MST)**, the network achieves active-active load balancing across the distribution layer.

### Key Technical Features

* **Redundancy:** FHRP (VRRP) for gateway redundancy; OSPF for dynamic path recovery.
* **Traffic Engineering:** MST instances used to load balance specific VLANs across different physical uplinks.
* **Scalability:** EtherChannel (LACP) used to increase bandwidth between switching tiers.
* **Security:**  VLAN Segmentation & Private/Native VLAN 99 implementation.
* **Port Security:** **PortFast** and **BPDU Guard** enabled on all access ports.
* **Access Control**: ACLs implemented to isolate IoT (VLAN 40) and prevent unauthorized inter-VLAN routing.
* **Edge Protection:** NAT Overload (PAT) and DHCP-assigned ISP interfaces.

### Infrastructure Services (Debian Linux)
To centralize network management, a Debian Server was deployed to provide critical infrastructure services across all subnets.

* **DHCP Service:** Configured with multiple scopes to dynamically assign IP addresses to VLANs 10 and 20 based on the VLSM plan. VLAN 30 is apart of the management range and VLAN 40 is for IoT connections thus the were remove from the getting dhcp and dns servrices and assigned IP addresses statically.

* **DNS Service:** Provides internal name resolution for local resources and forwards external queries to the ISP routers.

* **Relay Logic:** Since the server sits in a specific subnet, ip helper-address was configured on all SVI (Switch Virtual Interfaces) on the Distribution Switches to forward DHCP broadcasts to the Debian server's IP.

---

## Network Topology & Addressing

The network uses a private $/16$ block ($172.16.0.0$) subdivided using **VLSM** to maximize address efficiency.

### IP Addressing Schema

| Department / Link | VLAN | Subnet (CIDR) | IP Range | Gateway (VRRP) |
| --- | --- | --- | --- | --- |
| **Corporate Staff** | 10 | 172.16.0.0/26 | .1 - .62 | 172.16.0.1 |
| **Finance & HR** | 20 | 172.16.0.64/28 | .65 - .78 | 172.16.0.65 |
| **Security/IoT** | 40 | 172.16.0.80/28 | .81 - .94 | 172.16.0.81 |
| **Management** | 30 | 172.16.0.96/29 | .97 - .102 | 172.16.0.97 |
| **Transit: DSW to DSW** | 50 | 172.16.0.104/30 | .105 - .106 | P2P Link |
| **Transit: DSW to Core** | -- | 172.16.0.108/30+ | Various | P2P Links |

---

## Implementation Details

### 1. Layer 2 Redundancy (MST & EtherChannel)

To prevent the waste of blocked ports in standard Spanning Tree, **MST** was configured:

* **Instance 1 (VLAN 10, 20):** Primary Root on `DistributionSwitch1`.
* **Instance 2 (VLAN 30, 40):** Primary Root on `DistributionSwitch2`.
This ensures that both distribution switches are actively forwarding traffic simultaneously.

### 2. Layer 3 Redundancy (VRRP & OSPF)

* **VRRP:** Provides a virtual default gateway for end-devices.
* **OSPF:** Configured with `Point-to-Point` network types on transit links to ensure fast convergence and neighbor stability.
* **Metric Manipulation:** `CoreRouter1` is configured with a lower OSPF cost/metric to serve as the primary exit point to the ISP, with `CoreRouter2` acting as a seamless failover.

### 3. Security & Access Control

* **IoT Isolation:** VLAN 40 is restricted via ACLs. It can communicate with the Management VLAN (VLAN 30) for data logging but is blocked from the Internet and other internal departments.
* **Management Security:** All devices are password-protected with encrypted secrets and assigned to a non-routed management VLAN.

---

## Troubleshooting Log

| Issue | Root Cause | Resolution |
| --- | --- | --- |
| **Trunking Failure** | Interface G0/0 stuck in Access mode mismatch. | Cycled interface (`shutdown`/`no-sh`) and manually forced `switchport mode trunk`. |
| **VLAN 30/40 Connectivity** | Incorrect VLAN assignment on end-devices. | Verified and corrected host IP addressing within the valid VLSM range. |
| **OSPF Adjacency Drops** | Network type mismatch (Broadcast vs. P2P). | Uniformly configured OSPF interfaces as `ip ospf network point-to-point`. |
| **VRRP Failure** | VRRP Hellos blocked by a routed (L3) PortChannel. | Converted the DSW-to-DSW link to a Layer 2 PortChannel to allow multicast VRRP traffic. |

---


**Author:** [Jayden Thomad/JaydenThomas007]
**Date:** March 11, 2026
**Tools:** GNS3 / Cisco Routers & Switchs / Linux 
