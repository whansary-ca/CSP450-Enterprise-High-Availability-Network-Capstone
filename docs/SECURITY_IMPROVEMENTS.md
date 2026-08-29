# Security and Production-Hardening Improvements

The capstone was designed as a controlled training environment. The following changes describe how the same architecture could be hardened for a production deployment.

## 1. Replace permissive forwarding with least privilege

The lab nftables configuration used permissive filter policies so routing and failover behavior could be validated quickly. A production design should use a default-deny forwarding policy and explicitly allow only required traffic.

Example policy goals:

- Allow established and related sessions
- Permit required inter-VLAN application flows only
- Restrict database access to known application or administration hosts
- Permit management protocols only from management networks
- Log and drop unexpected forwarding traffic

## 2. Separate management traffic

Create a dedicated management VLAN for switch, router and server administration. Restrict SSH and device-management access to trusted administrator systems rather than exposing management interfaces to user VLANs.

## 3. Harden SSH

- Use key-based authentication
- Disable direct root login
- Disable password authentication where operationally appropriate
- Restrict SSH by source subnet or management ACL
- Keep OpenSSH patched and review authentication logs

## 4. Apply inter-VLAN ACLs

VLAN segmentation is more useful when supported by policy enforcement. ACLs should limit unnecessary east-west traffic between branch users, servers, management interfaces and database services.

## 5. Protect MariaDB

The project already used a read-only account for remote testing. A production implementation should additionally:

- Bind the database service only to required interfaces
- Restrict TCP/3306 with host firewall rules and network ACLs
- Use unique service accounts with minimum required permissions
- Require strong authentication and protected secret storage
- Encrypt database connections where appropriate
- Back up data and test restoration procedures

## 6. Harden the web tier

- Enable HTTPS with managed certificates
- Validate and sanitize application input
- Use parameterized SQL queries
- Disable unnecessary Apache modules
- Avoid exposing application or database credentials in source code
- Apply secure file permissions to application configuration

## 7. Improve network-device access controls

For Aruba infrastructure:

- Use centralized AAA when available
- Restrict management access with ACLs
- Disable unused switch ports
- Assign unused ports to an isolated VLAN or administratively shut them down
- Use secure management protocols only
- Back up running configurations and track approved changes

## 8. Add monitoring and logging

Centralize logs from Linux routers, servers and network devices. Useful events include:

- Authentication failures
- Interface state changes
- OSPF neighbor changes
- VRRP transitions
- Firewall drops
- DHCP events
- Web/database failures

Alerts on repeated failovers or routing-neighbor changes would help detect instability before users report an outage.

## 9. Protect the routing control plane

Where supported and justified, use routing-protocol authentication, infrastructure ACLs and management-plane restrictions. Limit who can form routing adjacencies and monitor for unexpected OSPF neighbors or route changes.

## 10. Build operational resilience

High availability is not only gateway redundancy. A production design should also consider:

- Configuration backups
- Database backups and tested restore procedures
- Patch management
- UPS/power redundancy
- Monitoring of link and device health
- Documented failover and rollback procedures
- Change control
- Capacity monitoring

## Security Takeaway

The lab demonstrates the network and high-availability mechanisms. Production hardening would preserve that functionality while applying least privilege, stronger management isolation, encrypted administration, controlled east-west access, centralized logging and formal operational controls.
