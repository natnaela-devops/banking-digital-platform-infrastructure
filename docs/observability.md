# Observability and Alerting

The platform combined metrics, alerting, logs, and application telemetry rather than relying on a single monitoring source.

## Prometheus and Alertmanager

Prometheus collected platform and service metrics, while Alertmanager handled alert routing and notification workflows.

Operational work included:

- node and service metrics
- disk/resource monitoring
- retention checks
- alert-rule validation
- notification testing
- confirming the difference between a transient firing alert and an actual unresolved incident

## OpenSearch

OpenSearch was used for centralized logging and observability workflows.

A key operational improvement was separating **application/deployment logs** from **RKE2/Kubernetes/platform logs** so service owners and platform operators could work with cleaner views.

## Telemetry flow

```text
Applications ----> logs/traces ----> OpenSearch
     |
     +------------> metrics --------> Prometheus
                                      |
                                      v
                                  Alertmanager
```

OpenTelemetry was integrated where applicable for application telemetry.

## Troubleshooting approach

Incidents were investigated by correlating:

- logs
- metrics
- alert state
- runtime configuration
- service/process health
- network/protocol validation
- application behavior

No single green dashboard or reachable port was treated as sufficient proof of service health.

Public documentation excludes customer dashboards, internal endpoints, alert destinations, credentials, and environment-specific labels that could identify the production environment.