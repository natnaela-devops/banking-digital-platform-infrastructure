# Databases and Storage

The platform used several data and storage technologies for different workload characteristics rather than forcing every service onto one database or storage model.

## YugabyteDB

YugabyteDB was operated as a distributed SQL cluster for application workloads requiring horizontal distribution and PostgreSQL-compatible access.

Operational responsibilities included:

- cluster health validation
- YSQL readiness troubleshooting
- application connectivity checks
- schema/migration verification
- separating database health from application startup failures

## TimescaleDB / PostgreSQL

Standalone PostgreSQL/TimescaleDB workloads supported services such as identity and application data where a separate relational database fit the workload better.

Operational work included:

- service availability
- connectivity and credentials
- schema/migration verification
- application integration
- backup/recovery awareness

## SeaweedFS

SeaweedFS supported receipt/image/object-style storage flows. It was part of the application path for transaction evidence and related file content.

## Longhorn

Longhorn supplied persistent storage for selected Kubernetes workloads. Operations included PVC handling, storage health, node/disk awareness, and troubleshooting stale webhook/PVC conditions when they occurred.

## Storage principle

Data-layer troubleshooting was handled by validating each dependency independently:

```text
Application
   |
   +--> database connectivity
   +--> schema / migration state
   +--> persistent volume health
   +--> object/file storage path
   +--> credentials / permissions
```

Public documentation excludes real schemas, customer data, database credentials, internal endpoints, and exact production sizing.