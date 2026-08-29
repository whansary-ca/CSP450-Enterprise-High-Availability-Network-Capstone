# Network Architecture

## Business Requirement

The capstone models a multi-store musical-instrument business. Staff should be able to search inventory without caring which branch physically owns an item. The network therefore has to provide segmentation, routed connectivity, application access, Internet access, and gateway resilience.

## Logical Topology

```text
                          SenecaNet / Internet
                                  |
                     +------------+------------+
                     |                         |
              Ubuntu Router A            Ubuntu Router B/C
              FRRouting / OSPF            FRRouting / OSPF
              nftables + NAT              nftables + NAT
                     |                         |
                  /30 transit links to Aruba cores
                     |                         |
          +----------+-------------------------+----------+
          |                                             |
  Aruba 6300 Core 1                              Aruba 6300 Core 2
  VRRP preferred                                 VRRP backup
  OSPF RID 1.1.1.1                              OSPF RID 2.2.2.2
  STP priority 0                                 STP priority 4
          | \                                         / |
          |  \                                       /  |
          |   +------ redundant 802.1Q trunks ------+   |
          |                                             |
     Aruba access switches protected by STP
          |
   +------+------+------+
   |             |      |
VLAN 402      VLAN 404  VLAN 420
Jahid         Walid     Abdul
   |             |      |
Client+Server Client+Server Client+Server
                 |
         172.16.101.0/26
         VIP 172.16.101.1
         Server 172.16.101.62
                 |
        Apache/PHP + MariaDB
```

## Walid Branch

| Item | Value |
|---|---|
| VLAN | 404 |
| LAN | `172.16.101.0/26` |
| VRRP VIP | `172.16.101.1` |
| Core 1 SVI | `172.16.101.2/26` |
| Core 2 SVI | `172.16.101.3/26` |
| Server | `172.16.101.62/26` |
| Router transit network | `192.168.6.80/30` |
| Core transit | `192.168.6.81` |
| Router transit | `192.168.6.82` |
| Router OSPF RID | `4.0.4.4` |

## Layer 2

- Each branch is isolated in its assigned VLAN.
- Access switches use 802.1Q trunks toward both cores.
- Redundant trunk paths are controlled by Spanning Tree.
- The links are independent redundant trunks, not an LACP bundle.

## Layer 3

- Core switches host branch SVIs.
- VRRP presents a stable virtual gateway to endpoints.
- OSPF exchanges routes between the cores and Linux branch routers.
- `/30` networks provide point-to-point routed transit links.

## Internet Edge

The Ubuntu router forms the branch security/routing boundary:

```text
Private branch -> Aruba core -> /30 transit -> Ubuntu router -> SenecaNet/Internet
```

The router uses:
- FRRouting for OSPF
- IPv4 forwarding
- nftables
- NAT masquerading through the external interface

## Application Path

```text
Client Browser
     |
     v
Apache / PHP on 172.16.101.62
     |
     v
MariaDB inventory
     |
     v
Search results returned to user
```

The business application depends on every lower layer functioning correctly: VLAN membership, addressing, DHCP, routing, firewall/NAT, web service, and database access.

## High Availability

Core 1 is configured with the higher VRRP priority and normally owns the virtual gateways. Core 2 remains available as backup. During the failure test, the preferred core was isolated and the backup core was expected to assume MASTER state while end-to-end services were retested.
