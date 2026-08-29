# CSP450 Troubleshooting Guide

This guide documents the fault-isolation workflow used during the CSP450 high-availability network capstone.

## 1. Client or Server Has No IPv4 Address

Likely causes:
- Incorrect VM network mapping
- Wrong access VLAN
- Trunk not carrying the VLAN
- DHCP scope unavailable

Checks:
```bash
ip -br a
ip route
sudo tcpdump -ni ens33 port 67 or port 68
```
On Aruba:
```text
show vlan
show mac-address-table
show dhcp-server
show spanning-tree
```

## 2. Router Cannot Reach Core Transit IP

Likely causes:
- Wrong VM interface mapping
- Incorrect /30 addressing
- Routed switch port down
- ARP failure

Checks:
```bash
ip -br a
ip route
sudo tcpdump -ni ens37 arp
```
On Aruba:
```text
show ip interface brief
show arp
```

## 3. OSPF Neighbor Missing but Ping Works

Likely causes:
- OSPF process mismatch
- Wrong network type
- Interface not included in OSPF

Checks:
```text
show ip ospf neighbor
show ip ospf interface
show running-config
```
Both sides of the /30 transit were configured as point-to-point.

## 4. Client Reaches Gateway but Not Internet

Likely causes:
- Missing default route
- Linux IPv4 forwarding disabled
- nftables/NAT problem

Checks:
```bash
ip route
sysctl net.ipv4.ip_forward
sudo nft list ruleset
ping -c 3 8.8.8.8
ping -c 3 google.ca
```

## 5. Internet Works by IP but DNS Fails

Check:
```bash
resolvectl status
ping -c 3 8.8.8.8
ping -c 3 google.ca
```
Also verify the DHCP-delivered DNS server.

## 6. VRRP Gateway Works but DHCP Renewal Fails During Failover

Likely cause:
- Only the failed core has a usable DHCP scope

The project addressed this by splitting the dynamic DHCP range between the two cores.

## 7. Application Page Is Unreachable

Test each dependency separately:
```bash
ping 172.16.101.62
ssh <user>@172.16.101.62
curl http://172.16.101.62
```
Then verify Apache/PHP and MariaDB on the server.

## 8. Database Search Fails While Web Page Loads

Check:
- MariaDB service status
- Database user permissions
- Port 3306 reachability
- PHP database connection settings

Useful packet filter:
```text
tcp.port == 3306
```

## 9. Failover Validation Sequence

Before failure:
- Confirm Core 1 is VRRP MASTER
- Confirm Core 2 is BACKUP
- Confirm OSPF neighbors and routes
- Verify application, DNS, Internet and SSH

During failure:
- Isolate the preferred core
- Wait for convergence
- Verify Core 2 becomes MASTER
- Recheck OSPF and routes
- Retest services

After restoration:
- Restore Core 1
- Wait for convergence
- Confirm preferred role returns
- Repeat end-to-end tests

## Troubleshooting Principle

Work from the lowest failing layer upward:

1. Interface/link
2. VLAN
3. IP addressing
4. Routing
5. NAT/firewall
6. DNS
7. SSH/web/database application

This avoids changing multiple layers at once and makes root-cause isolation faster.
