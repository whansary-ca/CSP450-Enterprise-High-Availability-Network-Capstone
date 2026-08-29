# Stage 4 — High Availability with VRRP

Stage 4 replaced the single-gateway design with redundant core switching and tested whether branch services remained usable after a preferred-core failure.

## High-Availability Design

Two Aruba 6300 core switches provided Layer-3 SVIs, OSPF and VRRP for the branch VLANs.

For Walid VLAN 404:

- Virtual gateway: `172.16.101.1`
- Core 1 SVI: `172.16.101.2/26`
- Core 2 SVI: `172.16.101.3/26`
- Core 1 VRRP priority: `110`
- Core 2 VRRP priority: `90`

The preferred core normally owns the virtual gateway. If it becomes unavailable, the backup core assumes the VRRP `MASTER` role.

## Redundant Layer-2 Paths

Access switches had redundant 802.1Q trunk paths toward the two cores. Spanning Tree was used to prevent Layer-2 loops while preserving the redundant physical paths.

These links were separate redundant trunks, not an LACP bundle.

## DHCP Resilience

The dynamic DHCP address space was divided between the two cores so the surviving core could continue serving clients after a failure. The server retained a deterministic address through a static reservation.

## Failover Test

The validation workflow was:

1. Confirm normal end-to-end operation with both cores online.
2. Verify Core 1 was `MASTER` and Core 2 was `BACKUP`.
3. Isolate/power down the preferred Core 1.
4. Wait for VRRP and OSPF convergence.
5. Verify Core 2 transitioned to `MASTER`.
6. Recheck routing and OSPF neighbours.
7. Retest gateway, own server, remote server, Internet, DNS, SSH and Stage 3 application services.
8. Restore Core 1 and allow the topology to reconverge.
9. Verify Core 1 returned to the preferred role and repeat service testing.

## Example Core-1 Configuration Extract

```text
router vrrp enable

interface vlan 404
 no shutdown
 ip address 172.16.101.2/26
 ip ospf 1 area 0.0.0.0
 vrrp 4 address-family ipv4
  address 172.16.101.1 primary
  priority 110
  no shutdown
 exit
exit

router ospf 1
 router-id 1.1.1.1
 area 0.0.0.0
exit
```

## Troubleshooting Logic

Examples of the project's fault-isolation workflow:

| Symptom | Area to investigate |
|---|---|
| VM receives no IPv4 address | VLAN membership, trunks, DHCP, VM network mapping |
| Router cannot reach `/30` peer | routed port, VM mapping, ARP |
| OSPF neighbour missing while ping works | OSPF process/network type |
| Gateway works but Internet fails | default route, IP forwarding, NAT |
| Gateway failover works but DHCP renewal fails | surviving-core DHCP scope |

This stage demonstrated availability engineering rather than only configuration: the project deliberately introduced failure and verified that routing and business services recovered as designed.
