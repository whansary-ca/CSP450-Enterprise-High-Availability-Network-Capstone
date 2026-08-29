# About This Project

**CSP450 Enterprise High-Availability Network Capstone** is a hands-on networking and infrastructure project completed at Seneca Polytechnic.

The project models a resilient multi-branch business network for a musical-instrument retailer. It combines physical Aruba 6300 switching with Ubuntu client, server, and router systems to provide segmented branch connectivity, dynamic routing, Internet access, application services, and gateway redundancy.

## What I Implemented

- Configured VLAN-based branch segmentation and 802.1Q trunking on Aruba 6300 switches.
- Built Layer-3 connectivity using SVIs, `/26` branch networks, and `/30` routed transit links.
- Configured OSPF dynamic routing using Aruba AOS-CX and FRRouting on Ubuntu routers.
- Implemented VRRP high availability with preferred and backup core switches.
- Configured DHCP services and split address pools to improve service continuity during failover.
- Enabled Linux IPv4 forwarding and nftables NAT for controlled Internet connectivity.
- Deployed Apache/PHP and MariaDB to support a searchable multi-store inventory application.
- Used read-only remote database access for cross-branch query testing.
- Validated connectivity and failover using ping, SSH, routing tables, OSPF neighbour checks, VRRP state, Wireshark, tshark, and tcpdump.
- Performed failure testing by isolating the preferred core and verifying gateway and application recovery through the backup infrastructure.

## Technologies

`Aruba 6300` · `AOS-CX` · `Ubuntu Linux` · `VLANs` · `802.1Q` · `STP` · `DHCP` · `OSPF` · `FRRouting` · `VRRP` · `Netplan` · `nftables` · `NAT` · `SSH` · `Wireshark` · `Apache` · `PHP` · `MariaDB`

## Career-Relevant Skills

This project demonstrates practical skills relevant to **Network Support, Network Technician, Systems/Infrastructure Support, Cloud Support, Junior Network Engineer, and Junior Systems Administrator** roles, including routing and switching, Linux administration, high availability, troubleshooting, service validation, and technical documentation.
