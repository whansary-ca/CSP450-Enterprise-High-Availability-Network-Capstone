# Stage 5 — Capstone Validation & Architecture Defense

Stage 5 brought the previous stages together into one resilient business system and focused on explaining the design, demonstrating service behavior, and proving failure recovery.

## Final Environment

The completed Pod A design included three branch VLANs:

| Branch | VLAN | Subnet | VRRP Gateway | Server |
|---|---:|---|---|---|
| Jahid | 402 | `172.16.100.128/26` | `172.16.100.129` | `172.16.100.190` |
| Walid | 404 | `172.16.101.0/26` | `172.16.101.1` | `172.16.101.62` |
| Abdul | 420 | `172.16.105.0/26` | `172.16.105.1` | `172.16.105.62` |

The environment used two Aruba 6300 cores, redundant access paths, Linux branch routers, OSPF and VRRP.

## What the Capstone Proved

- Branches remained separated at Layer 2 with VLANs.
- 802.1Q trunks carried the required VLANs between access and core infrastructure.
- STP prevented redundant paths from creating Layer-2 loops.
- OSPF dynamically exchanged branch reachability.
- `/30` networks were used for point-to-point router/core transit.
- Linux routers provided controlled Internet access through IPv4 forwarding and NAT.
- Apache/PHP and MariaDB provided an actual cross-layer business application.
- VRRP maintained a stable default gateway during a preferred-core failure.
- Normal operation and failure behavior were both tested.

## Validation Commands

### Aruba Core

```text
show vrrp
show ip ospf neighbor
show ip route
show vlan
show spanning-tree
show mac-address-table
```

### Ubuntu Router

```bash
ip -br a
ip route
sysctl net.ipv4.ip_forward
sudo nft list ruleset
sudo vtysh
```

Inside `vtysh`:

```text
show ip ospf neighbor
show ip route
show running-config
```

### Client / Server

```bash
ip -br a
ip route
ping <gateway>
ping <own-server>
ping <remote-server>
ping 8.8.8.8
```

## Packet Analysis

Useful Wireshark/tshark filters included:

```text
ospf
icmp
dns
tcp.port == 80
tcp.port == 3306
```

These allowed routing, reachability, DNS, HTTP and MariaDB traffic to be inspected separately.

## Engineering Decisions

### Why VRRP?
Hosts require a stable default gateway even when the physical core device changes. VRRP allows two cores to present one virtual gateway while only one forwards as the active owner at a time.

### Why OSPF?
Dynamic routing removes the need to maintain static routes for every remote branch and supports reconvergence when topology changes.

### Why `/30` transit networks?
A point-to-point routed link requires two usable addresses, making `/30` a clear and address-efficient choice for the lab design.

### Why split DHCP scopes?
Both cores can issue valid client leases without allocating the same dynamic addresses, and a surviving core retains DHCP capability during failover.

### Why STP?
Multiple physical Layer-2 paths provide redundancy but can create forwarding loops. STP preserves redundancy while controlling which path forwards.

## Final Business Result

The application requirement was simple: a salesperson should be able to search inventory without caring which branch owns the item. Under that experience, the project integrated switching, VLAN segmentation, IP addressing, DHCP, OSPF, Linux routing, NAT/firewalling, web/database services and VRRP redundancy into one functioning system.
