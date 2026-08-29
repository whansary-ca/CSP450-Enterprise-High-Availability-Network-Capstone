# Test Results

This document summarizes the validation workflow used for the CSP450 high-availability network capstone.

## Validation Matrix

| Test | Validation method | Expected result |
|---|---|---|
| Client addressing | `ip addr`, DHCP lease inspection | Client receives valid VLAN address, mask, gateway and DNS |
| Default gateway | `ping 172.16.101.1` | VRRP virtual gateway responds |
| Server reachability | `ping 172.16.101.62` | Branch server reachable |
| Router transit | Ping/route inspection | Core and Ubuntu router can reach the /30 transit peer |
| OSPF adjacency | `show ip ospf neighbor` / FRR `show ip ospf neighbor` | Neighbor state reaches FULL |
| Learned routes | `show ip route` / `ip route` | Remote branch networks appear in routing table |
| Internet/NAT | Ping or application test to external destination | Internal hosts reach outside network through Ubuntu router NAT |
| DNS | `nslookup`, `dig`, browser test | Names resolve using configured DNS |
| SSH | `ssh` between authorized lab hosts | Remote shell connectivity succeeds |
| Web application | Browser request to Apache/PHP server | Inventory lookup page loads and queries database |
| MariaDB access | Local/remote MariaDB client | Database connection succeeds with intended privileges |
| VRRP normal state | `show vrrp` | Core 1 is MASTER and Core 2 is BACKUP |
| VRRP failover | Remove/isolate preferred core and retest | Core 2 becomes MASTER and gateway remains available |
| Post-failover services | Repeat ping, DNS, SSH, web/database tests | Essential services remain reachable after convergence |
| Spanning Tree | `show spanning-tree` | Redundant Layer-2 topology remains loop-free |

## Walid Branch Reference Values

- VLAN `404`
- Network `172.16.101.0/26`
- Virtual gateway `172.16.101.1`
- Core 1 `172.16.101.2`
- Core 2 `172.16.101.3`
- Server `172.16.101.62`
- Observed DHCP client `172.16.101.53/26`
- Router transit `192.168.6.82/30`
- Core transit `192.168.6.81/30`

## Failover Test Sequence

1. Confirm Core 1 reports VRRP `MASTER` and Core 2 reports `BACKUP`.
2. Confirm the client can reach the virtual gateway and application server.
3. Verify OSPF neighbors and expected routes.
4. Isolate or power down the preferred core.
5. Wait for VRRP and routing convergence.
6. Confirm Core 2 transitions to `MASTER`.
7. Re-test virtual gateway reachability.
8. Re-test server, inter-branch routes, Internet/NAT and DNS.
9. Re-test SSH and the Apache/PHP inventory application.
10. Restore Core 1 and confirm the intended preferred-master state returns.

## Packet-Level Validation

Useful Wireshark display filters used during validation include:

```text
ospf
icmp
dns
tcp.port == 80
tcp.port == 3306
```

For Linux command-line captures, `tcpdump` or `tshark` can be used to verify whether packets reach an interface and whether replies return along the expected path.

## Interpretation

A successful ping alone does not prove the full design works. The project therefore validated the network at multiple layers: addressing, switching, routing, NAT, name resolution, remote access, database connectivity, web application behavior and redundancy.
