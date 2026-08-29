# Stages 1–2 — Foundation & Network Connectivity

## Stage 1: Branch Systems

The Walid branch used three Ubuntu virtual machines with separated roles:

- `whansary-client` — user workstation and validation endpoint
- `whansary-server` — application/database server
- `whansary-router` — Layer-3 edge, routing, NAT and firewall boundary

## Walid Branch Addressing

| Component | Address / Role |
|---|---|
| VLAN | 404 |
| Branch LAN | `172.16.101.0/26` |
| Gateway | `172.16.101.1` |
| Client | DHCP; observed `172.16.101.53/26` during implementation |
| Server | Reserved `172.16.101.62/26` |
| Router transit | `192.168.6.82/30` |
| Core transit peer | `192.168.6.81/30` |
| Router OSPF RID | `4.0.4.4` |

## Stage 2: Connectivity

Stage 2 connected the previously isolated client, server and router systems through the physical Aruba switching environment.

The implemented workflow included:

- VLAN 404 segmentation for the branch client/server network.
- Layer-3 SVI/default-gateway configuration.
- DHCP for endpoint addressing plus a deterministic server reservation.
- A `/30` point-to-point transit between the Aruba infrastructure and Ubuntu router.
- OSPF using FRRouting on the Linux router.
- IPv4 forwarding on the router.
- nftables NAT masquerading for Internet access.
- SSH connectivity between the client, server, router and managed infrastructure.

## Implementation Evidence

The original CSP450 evidence submission records:

- Client DHCP address `172.16.101.53/26` and successful gateway/Internet/DNS connectivity.
- Server reservation `172.16.101.62/26` and successful external connectivity.
- Router transit `192.168.6.82/30` with an Internet-facing interface learned through Seneca DHCP.
- IPv4 forwarding enabled and nftables masquerading through `ens33`.
- FRRouting/OSPF active on the transit network.
- Switch routing, DHCP, VLAN and OSPF configuration.
- Successful SSH tests from the client to the server, router and switch.

## Troubleshooting Approach

Validation was performed layer by layer rather than treating connectivity as one problem:

1. Confirm interface addressing and routes.
2. Ping the local gateway.
3. Test the branch server.
4. Test the routed transit.
5. Verify OSPF neighbours and learned routes.
6. Verify Linux IPv4 forwarding.
7. Inspect nftables/NAT.
8. Test Internet reachability by IP.
9. Test DNS resolution.
10. Verify SSH/application access.

This workflow separated endpoint, VLAN, routing, firewall/NAT and service-layer failures.
