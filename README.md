# CSP450 Enterprise High-Availability Network Capstone

## Resilient Multi-Store Inventory Network

This repository documents a multi-stage enterprise networking capstone built at Seneca Polytechnic. The project models a multi-branch musical-instrument business where staff can search inventory locally or across connected branches while the network provides segmentation, dynamic routing, controlled Internet access, and default-gateway redundancy.

## Project Outcome

The final environment combined three branch networks, physical Aruba 6300 switching, Ubuntu client/server/router virtual machines, OSPF routing, DHCP, NAT/firewalling, an Apache/PHP front end, MariaDB inventory services, and VRRP gateway failover.

### Walid branch

- VLAN: `404`
- Branch subnet: `172.16.101.0/26`
- VRRP default gateway: `172.16.101.1`
- Server: `172.16.101.62`
- Router transit: `192.168.6.82/30`
- Linux router OSPF RID: `4.0.4.4`

## Five-Stage Build

| Stage | Focus | Major Technologies |
|---|---|---|
| 1 | Foundation | Ubuntu Client, Server and Router VMs; role-specific packages |
| 2 | Connectivity | Aruba 6300, VLANs, 802.1Q trunks, DHCP, Layer-3 routing, OSPF, NAT |
| 3 | Business application | Apache, PHP, MariaDB, SQL, remote read-only database access |
| 4 | High availability | Dual core/access paths, STP, VRRP, split DHCP scopes, failover testing |
| 5 | Capstone validation | Architecture explanation, troubleshooting, packet-flow validation, failure testing |

## Architecture

```text
                         SenecaNet / Internet
                                  |
                         Ubuntu Branch Routers
                        FRRouting + nftables/NAT
                                  |
                         /30 routed transit links
                                  |
                  +---------------+---------------+
                  |                               |
          Aruba 6300 Core 1               Aruba 6300 Core 2
          VRRP preferred                   VRRP backup
          OSPF RID 1.1.1.1                OSPF RID 2.2.2.2
                  | \                           / |
                  |  \  redundant 802.1Q      /  |
                  |   \      trunks + STP     /   |
                  |    \                     /    |
             Access 6300s -------------------------
                  |
       VLAN 402 / VLAN 404 / VLAN 420
                  |
           Client + Server VMs
                  |
        Apache/PHP -> MariaDB inventory
```

## High-Availability Design

The core switches present a stable virtual default gateway using VRRP. Core 1 uses the higher priority and normally owns the virtual gateway; Core 2 remains ready to take over. The project validated failover by isolating the preferred core, confirming the backup core became `MASTER`, and retesting gateway, server, inter-branch, Internet, DNS, SSH, and application connectivity.

The design also used separate redundant trunks protected by Spanning Tree. These links were **not** configured as an LACP bundle.

## Application Layer

Stage 3 turned the infrastructure into a working business system. A client browser accesses an Apache/PHP instrument-search page on the branch server. PHP queries a MariaDB inventory database and returns matching stock. Remote database testing used a read-only account to limit modification privileges.

## Technologies

**Networking:** Aruba AOS-CX, Aruba 6300, VLANs, 802.1Q, STP, IPv4 subnetting, DHCP, OSPF, VRRP, routing, NAT

**Linux & Security:** Ubuntu, FRRouting, nftables, SSH, IPv4 forwarding, Netplan

**Application:** Apache2, PHP, MariaDB, SQL, CSV import

**Troubleshooting:** Wireshark, tshark, tcpdump, ping, route inspection, OSPF neighbour verification, VRRP state validation

## Skills Demonstrated

- Enterprise routing and switching
- High-availability network design
- Dynamic routing with OSPF
- Gateway redundancy with VRRP
- VLAN segmentation and Layer-2 loop prevention
- Linux routing and firewall/NAT administration
- DHCP and IP addressing design
- Web/database service deployment
- Cross-layer troubleshooting from client to database
- Failure testing and service validation
- Technical documentation and architecture communication

## Portfolio Highlights

- [`docs/PROJECT_SUMMARY.md`](docs/PROJECT_SUMMARY.md) — recruiter-focused overview
- [`docs/TEST_RESULTS.md`](docs/TEST_RESULTS.md) — validation matrix and failover test sequence
- [`docs/SECURITY_IMPROVEMENTS.md`](docs/SECURITY_IMPROVEMENTS.md) — production-hardening recommendations
- [`docs/RESUME_BULLETS.md`](docs/RESUME_BULLETS.md) — resume-ready project bullets
- [`docs/INTERVIEW_NOTES.md`](docs/INTERVIEW_NOTES.md) — technical interview talking points

## Repository Documentation

- [`docs/architecture.md`](docs/architecture.md)
- [`docs/stage-1-2-foundation-connectivity.md`](docs/stage-1-2-foundation-connectivity.md)
- [`docs/stage-3-application.md`](docs/stage-3-application.md)
- [`docs/stage-4-high-availability.md`](docs/stage-4-high-availability.md)
- [`docs/stage-5-validation.md`](docs/stage-5-validation.md)
- [`docs/troubleshooting-guide.md`](docs/troubleshooting-guide.md)
- [`docs/validation-checklist.md`](docs/validation-checklist.md)

## Configuration Files

- [`configs/aruba/core1-master.cfg`](configs/aruba/core1-master.cfg)
- [`configs/aruba/core2-backup.cfg`](configs/aruba/core2-backup.cfg)
- [`configs/aruba/walid-access.cfg`](configs/aruba/walid-access.cfg)
- [`configs/walid-router-frr.conf`](configs/walid-router-frr.conf)
- [`configs/router-nftables.conf`](configs/router-nftables.conf)
- [`configs/netplan/client-server-dhcp.yaml`](configs/netplan/client-server-dhcp.yaml)
