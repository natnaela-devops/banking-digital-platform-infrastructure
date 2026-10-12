# Architecture and Service Relationships

This document presents the platform in functional layers rather than reproducing a customer-specific network diagram.

## Edge and access layer

Nginx provided TLS termination, reverse proxying, and route separation for mobile APIs, gateway endpoints, merchant/backoffice interfaces, WebViews, receipt/evidence endpoints, and identity services.

## Application platform

RKE2 provided the Kubernetes runtime, with Rancher used for operational visibility and administration. Longhorn supplied persistent storage for selected Kubernetes workloads, while Harbor supplied container images to the cluster.

```text
Users
  |
  v
Nginx / TLS
  |
  +--> APIs / gateways
  +--> merchant / backoffice / webview
  +--> Keycloak
  +--> receipt / evidence endpoints
          |
          v
       RKE2 apps
```

## Messaging and cache

Kafka/KRaft handled asynchronous messaging. Redis operated in clustered master/replica form, with Predixy providing a stable application-facing routing layer.

```text
Application services
   |             |
   v             v
Kafka         Predixy
                |
                v
          Redis Cluster
```

Kafka client security was hardened through a staged migration from legacy PLAINTEXT application listeners to mTLS.

## Data layer

Application data was spread across different systems according to workload needs:

- YugabyteDB for distributed SQL workloads
- TimescaleDB/PostgreSQL for standalone relational/time-series workloads
- Keycloak database integration with TimescaleDB/PostgreSQL
- SeaweedFS for receipt/image/object-style storage
- Longhorn for Kubernetes persistent volumes

## Observability layer

Prometheus collected platform/service metrics, Alertmanager handled alert routing, and OpenSearch provided centralized log/observability workflows. Application and platform/Kubernetes logs were separated operationally to reduce noise and improve troubleshooting.

## Operational principle

The architecture was operated as an integrated system. A healthy pod, reachable port, or green cluster state was never treated as sufficient by itself; application behavior, runtime configuration, authentication, storage, replication/quorum state, and business flow were verified together.