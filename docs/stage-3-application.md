# Stage 3 — Web & Database Application

Stage 3 added a usable business application on top of the routed network.

## Objective

Turn the branch server into an application server that could support inventory searches from the client and, when routing permitted, from connected branches.

## Components

- Apache2 web server
- PHP application layer
- MariaDB inventory database
- CSV inventory import
- Browser-based instrument lookup
- MariaDB client testing
- Read-only remote database account

## Application Flow

```text
Client browser
     |
     v
Apache / PHP
     |
     v
MariaDB inventory database
     |
     +--> matching inventory rows
```

The branch server used `172.16.101.62` and hosted the web/database services.

## Security Design

Remote database testing used the read-only account `csp450ro`. This limited remote clients to query operations instead of granting broad modification privileges.

## Validation Performed

The project evidence demonstrates:

- Remote MariaDB login from the client using the read-only account.
- Inventory data imported from a flat file and supplemented with custom records.
- Browser search for `New Guitar` with a maximum price of `$1500`.
- Successful display of custom inventory records.
- Web application access from the branch client.
- Remote peer database/search testing when the connected branch was available.

## Skills Demonstrated

- Linux application-server administration
- Apache and PHP deployment
- MariaDB user and privilege management
- SQL and inventory-data operations
- Secure read-only remote database access
- Client-to-web-to-database troubleshooting
- Validation of application dependencies on network routing
