# Identity and Keycloak

Keycloak provided identity and access management for multiple application boundaries, including separate identity contexts for different products where required.

## Responsibilities

- operating Keycloak services
- database integration with PostgreSQL/TimescaleDB-backed workloads
- exposing identity endpoints through Nginx/TLS
- supporting realm/client configuration and application integration
- troubleshooting startup, connectivity, permissions, and application authentication paths
- keeping identity services logically separated where product boundaries required it

## Simplified flow

```text
Client application
      |
      v
 Nginx / TLS
      |
      v
   Keycloak
      |
      v
PostgreSQL / TimescaleDB
```

## Operational validation

Identity validation went beyond checking whether the Keycloak process was running. Checks included:

- service startup and logs
- database reachability
- reverse-proxy routing
- TLS endpoint availability
- application redirect/authentication behavior
- runtime configuration actually loaded by the service

Public documentation excludes real realms, clients, secrets, certificates, domains, and database credentials.