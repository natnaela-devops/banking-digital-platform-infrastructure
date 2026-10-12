# Operations, Troubleshooting, and Recovery

The platform was operated with an evidence-driven troubleshooting approach: inspect first, preserve rollback, isolate layers, and verify the actual application path after changes.

## Operating principles

- back up configuration before edits
- separate infrastructure health from application health
- validate quorum/replication before and after clustered-service changes
- use rolling restarts where the architecture allows it
- verify the runtime configuration actually loaded by the application
- test authentication and TLS directly when possible
- distinguish reachability from functional success
- do not call an incident resolved until the service path is proven healthy

## Example troubleshooting sequence

```text
Symptom
  |
  v
Check process / pod state
  |
  v
Check network / listener
  |
  v
Check authentication / TLS
  |
  v
Check runtime application config
  |
  v
Check cluster / database / storage dependency
  |
  v
Verify business flow
```

## Recovery mindset

Recovery procedures were designed around the specific failure mode rather than a generic restart-first approach. Examples included:

- rolling clustered-service changes with quorum checks
- preserving legacy Kafka access while mTLS clients were migrated
- validating certificate stores before restarting applications
- checking database schema/migration state before blaming application code
- using Longhorn and filesystem evidence when storage symptoms appeared
- separating Nginx/TLS routing problems from backend service failures
- using direct protocol tests to prove infrastructure health before changing application configuration

## Public evidence

The detailed sanitized runbooks for many of these cases live in:

[DevOps Troubleshooting Runbooks](https://github.com/natnaela-devops/devops-troubleshooting-runbooks)

That repository includes Kafka/KRaft, RKE2, Redis, Linux, Java/WildFly, databases, Longhorn, Harbor, Nginx/networking, OpenSearch, Prometheus/Alertmanager, and other operational cases.

## Confidentiality boundary

No real customer addresses, certificates, secrets, keystores, cluster IDs, production logs, or proprietary application data are published.