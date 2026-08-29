# Technical Interview Notes

This file provides concise talking points for explaining the CSP450 project in interviews.

## 1. What was the project?

A multi-stage enterprise networking capstone that connected multiple branch networks and added high availability. The final design used Aruba 6300 switches, Ubuntu routers, OSPF, VRRP, DHCP, NAT, Apache/PHP and MariaDB.

## 2. Why use VLANs?

VLANs separate Layer-2 broadcast domains and provide logical segmentation. Each branch/user environment can be placed in its own subnet and policy boundary while sharing switching infrastructure.

## 3. Why use OSPF instead of static routing everywhere?

OSPF allows routers to exchange reachability dynamically. In a multi-branch environment, that reduces manual route maintenance and provides a better foundation for convergence when paths change.

## 4. What does VRRP solve?

VRRP removes the default gateway as a single point of failure. Hosts use one virtual IP address as their gateway, while two physical routers or Layer-3 switches negotiate which device currently owns that address.

## 5. How was VRRP configured in the Walid VLAN?

- Virtual gateway: `172.16.101.1`
- Core 1: `172.16.101.2/26`, preferred priority `110`
- Core 2: `172.16.101.3/26`, backup priority `90`

Core 1 normally operated as MASTER. During failure testing, Core 2 was expected to transition to MASTER and continue answering for the virtual gateway.

## 6. What is the difference between STP and VRRP here?

STP protects the Layer-2 topology from loops caused by redundant switched links. VRRP provides Layer-3 default-gateway redundancy. They solve different problems and were both required for the redundant design.

## 7. Were the redundant trunks using LACP?

No. The project used separate 802.1Q trunk links protected by Spanning Tree. They should not be described as an LACP bundle.

## 8. What did FRRouting do?

FRRouting provided routing-protocol functionality on the Ubuntu router. The Walid router used OSPF router ID `4.0.4.4` and advertised/participated in the relevant routed networks while originating the default route for downstream routing.

## 9. What did nftables do?

The Ubuntu router used nftables for NAT/masquerading so hosts on private branch subnets could access the external network through the router's Internet-facing interface.

## 10. How did you troubleshoot a client that did not receive DHCP?

A structured workflow would be:

1. Confirm the access port is in the correct VLAN.
2. Confirm the trunk carries that VLAN.
3. Check the VLAN SVI is up.
4. Verify the DHCP pool exists and has free addresses.
5. Check for exclusions/reservations.
6. Renew the client lease.
7. Capture DHCP traffic if necessary and verify DISCOVER/OFFER/REQUEST/ACK behavior.

## 11. How did you troubleshoot OSPF?

- Check interface addressing and subnet masks
- Confirm the transit network is included in OSPF
- Verify the interface/network type
- Confirm router IDs are unique
- Check neighbor state with `show ip ospf neighbor`
- Inspect routing tables after adjacency forms
- Use packet capture to verify OSPF hellos if the neighbor never appears

## 12. Why use /30 transit links?

A traditional /30 provides four addresses: one network, two usable host addresses and one broadcast address. It is simple and efficient for a point-to-point routed link between two devices.

## 13. How did you validate failover?

First confirm normal MASTER/BACKUP VRRP state and service connectivity. Then remove or isolate the preferred core, wait for convergence, confirm the backup becomes MASTER, and repeat gateway, routing, DNS, SSH, Internet and application tests.

## 14. What application services were included?

The server hosted an Apache/PHP inventory-search page connected to MariaDB. The application gave the project an end-to-end workload so network tests went beyond ping and could validate real web/database traffic.

## 15. Why use a read-only database account?

It follows least privilege. A user or remote system that only needs to query inventory should not receive unnecessary INSERT, UPDATE or DELETE permissions.

## 16. What would you improve for production?

- Default-deny firewall policy
- Dedicated management VLAN
- ACLs between user/server/management networks
- SSH keys and centralized AAA
- HTTPS and safer secret handling
- Database access restrictions and encryption
- Centralized monitoring/logging
- Configuration and database backups
- Formal change control and failover procedures

## 17. Strong interview explanation

> I built the networking portion of a multi-branch high-availability lab using Aruba 6300 switches and Ubuntu routing. I configured VLANs, trunks, DHCP, OSPF and VRRP, used FRRouting and nftables on the Linux router, and then validated the design with routing, failover and application tests. The most useful part was troubleshooting across layers instead of treating each device separately—for example checking VLAN membership, addressing, OSPF neighbors, routes, NAT and application connectivity in sequence.
