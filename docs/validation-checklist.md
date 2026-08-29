# Validation Checklist

Use this checklist to verify the CSP450 environment from Layer 2 through the application layer.

## Aruba Core / Access Verification

```text
show interface brief
show ip interface brief
show vlan
show spanning-tree
show mac-address-table
show vrrp
show ip ospf neighbor
show ip route
show dhcp-server
show arp
```

Expected results:
- Required VLANs exist and trunk/access membership is correct.
- STP is active across redundant trunk paths.
- Core 1 is preferred VRRP MASTER during normal operation.
- Core 2 is VRRP BACKUP during normal operation.
- OSPF neighbors reach FULL state.
- Branch routes are present.
- DHCP is enabled on both cores with split dynamic scopes.

## Walid Router Verification

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
show ip ospf interface
show ip route
show ip route ospf
show running-config
```

Expected results:
- Transit interface uses `192.168.6.82/30`.
- OSPF RID is `4.0.4.4`.
- The core-side neighbor is reachable over the `/30` transit.
- IPv4 forwarding is enabled.
- NAT masquerade is active toward `ens33`.

## Walid Client Verification

```bash
ip -br a
ip route
ping -c 3 172.16.101.1
ping -c 3 172.16.101.62
ping -c 3 8.8.8.8
ping -c 3 google.ca
```

Expected results:
- Client receives a DHCP address from VLAN 404.
- Default route points to `172.16.101.1`.
- Local server, Internet IP, and DNS-based reachability succeed.

## Server Verification

```bash
ip -br a
ip route
systemctl status ssh --no-pager
systemctl status apache2 --no-pager
systemctl status mariadb --no-pager
```

Expected results:
- Server uses reserved address `172.16.101.62/26`.
- SSH, Apache, and MariaDB services are available.

## Application Verification

1. Open the Instrument Lookup page on the branch server.
2. Search for inventory such as `New Guitar` with a maximum price of `$1500`.
3. Confirm matching rows are returned.
4. Verify remote MariaDB access using the read-only account where applicable.

Useful packet filters:

```text
icmp
ospf
dns
tcp.port == 80
tcp.port == 3306
```

## High-Availability Test

### Before failure
- Core 1: VRRP MASTER
- Core 2: VRRP BACKUP
- OSPF routes/neighbors healthy
- Client, server, Internet, DNS, SSH and application tests pass

### During preferred-core failure
- Isolate Core 1
- Wait for VRRP/OSPF convergence
- Confirm Core 2 becomes MASTER
- Re-run route, reachability, DNS, SSH and application tests

### After restoration
- Restore Core 1
- Wait for convergence
- Confirm Core 1 returns to preferred MASTER role
- Repeat end-to-end validation

A successful test demonstrates that the design supports service continuity across a core-gateway failure rather than only working under normal conditions.
