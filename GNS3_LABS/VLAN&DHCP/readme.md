# Simple VLAN and DHCP Network Setup

## Objective
This lab demonstrates a **VLAN-based network** with DHCP services using a **Router-on-a-Stick (ROAS)** configuration.  
It includes VLAN segmentation, inter-VLAN routing, DHCP and DNS services, NAT, and access control between VLANs.

---

## Network Overview

### VLANs and IP Addressing
| VLAN  | Subnet           | Gateway        | Description                  |
|-------|-----------------|---------------|------------------------------|
| VLAN10| 192.168.10.0/25  | 192.168.10.1  | Office Network               |
| VLAN20| 192.168.10.128/26| 192.168.10.129| Guest Network                |
| Server| 192.168.10.192/29| 192.168.10.193| Debian DNS/DHCP Server       |

---

### Topology Description
- The router implements **Router-on-a-Stick (ROAS)** on interface `G1/0` for VLANs 10, 20, and 30.  
- Router interface `G0/0` is connected to a NAT node and configured to obtain an IP via DHCP.  
- Network Address Translation (NAT) and **Access Control Lists (ACLs)** are implemented to allow internet access.  
- Additional ACLs block communication between VLAN 10 and VLAN 20.  
- A Debian server provides **DNS and DHCP services** for both VLAN 10 and VLAN 20.

---

## Configuration Notes

### Router (ROAS) Key Settings
- Subinterfaces configured for each VLAN (`G1/0.10`, `G1/0.20`, `G1/0.30`)  
- `ip nat inside` applied on interface connecting to the internal network  
- ACLs implemented to prevent inter-VLAN communication where required

### DHCP/DNS Server (Debian)
- Provides DHCP ranges for VLAN10 and VLAN20  
- DNS resolution for internal hosts and forwards external queries to the internet

---

## Issues Faced
**Problem:** Devices in VLAN20 were not receiving DHCP addresses from the server.  
**Solution:** Added `ip nat inside` on the router interface connected to the server (`G1/0.20`) to allow proper DHCP relay and NAT functionality.

---

## Verification
- `ping` tests between hosts and router gateways  
- `show ip route` and `show vlan brief` on routers and switches  
- DHCP lease validation on hosts  
- NAT translations verified with `show ip nat translations`  

---

## Observations & Conclusion
- Router-on-a-Stick allows multiple VLANs to share a single router interface.  
- ACLs effectively block undesired inter-VLAN traffic.  
- NAT and DHCP must be properly configured to ensure internet connectivity and address assignment.  
- Lab demonstrates practical VLAN segmentation, DHCP deployment, and network security controls.

---

## Repository Structure
