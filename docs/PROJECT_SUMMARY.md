# Project Summary

## CSP450 Enterprise High-Availability Network Capstone

This capstone models a resilient multi-branch network for a musical-instrument business. The environment combines enterprise switching, Linux routing, dynamic routing, gateway redundancy, DHCP, NAT, and a small web/database application so the design can be validated end to end.

## What I Built

- Three branch networks using separate VLANs and /26 LANs
- Aruba 6300 switching with 802.1Q trunks and Spanning Tree
- Layer-3 routing between branch networks
- OSPF dynamic routing on Aruba and Ubuntu/FRRouting devices
- VRRP default-gateway redundancy using preferred and backup core switches
- Split DHCP scopes across redundant core switches
- Ubuntu router with IPv4 forwarding and nftables masquerading/NAT
- Apache/PHP inventory lookup application backed by MariaDB
- Remote read-only database access for controlled cross-branch testing
- Connectivity, routing, application, and failover validation using CLI tools and packet analysis

## Walid Branch

- VLAN: `404`
- LAN: `172.16.101.0/26`
- VRRP virtual gateway: `172.16.101.1`
- Core 1 SVI: `172.16.101.2/26`
- Core 2 SVI: `172.16.101.3/26`
- Server: `172.16.101.62`
- Core-to-router transit: `192.168.6.80/30`
- Core-side transit address: `192.168.6.81`
- Ubuntu router transit address: `192.168.6.82`
- Ubuntu router OSPF router ID: `4.0.4.4`

## High-Availability Design

Core 1 was configured as the preferred VRRP gateway with priority `110`; Core 2 used priority `90` as the backup. Both cores participated in the routed topology while Spanning Tree protected the redundant Layer-2 paths. The links were separate 802.1Q trunks rather than an LACP bundle.

The failover test removed the preferred core from service, verified that Core 2 became VRRP `MASTER`, and then retested gateway, server, routing, DNS, Internet, SSH, and application connectivity.

## Application Flow

```text
Client Browser
     |
     v
Apache + PHP
     |
     v
MariaDB Inventory Database
```

The Stage 3 application allowed a client to search instrument inventory through a PHP web page. MariaDB stored the inventory data, and remote database tests used a read-only account to reduce unnecessary modification privileges.

## Troubleshooting Approach

The project required troubleshooting across multiple layers rather than testing one device in isolation. The workflow included checking:

1. Link and VLAN membership
2. DHCP addressing and default gateway
3. Local subnet reachability
4. Routing table entries
5. OSPF adjacency and route learning
6. NAT and Internet reachability
7. DNS resolution
8. SSH and application access
9. VRRP state before and after failure
10. Packet captures when control-plane or application behavior needed deeper verification

## Technologies

Aruba AOS-CX, Aruba 6300, VLANs, 802.1Q, STP, DHCP, OSPF, VRRP, IPv4 subnetting, Ubuntu, FRRouting, Netplan, nftables, NAT, SSH, Apache2, PHP, MariaDB, SQL, Wireshark, tcpdump and tshark.

## Portfolio Relevance

This project demonstrates practical skills relevant to junior network engineering, infrastructure support, network support, systems administration, cloud support and technical operations roles. It is academic project experience and should be presented as hands-on coursework rather than paid production experience.
