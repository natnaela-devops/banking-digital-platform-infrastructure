# UAT to Production Workflow

Changes were validated in UAT before being introduced into production, with emphasis on preserving rollback paths and verifying runtime behavior rather than only static configuration.

## Typical workflow

1. Inspect the current environment and identify dependencies.
2. Back up configuration and record the current working state.
3. Apply and validate the change in UAT.
4. Test infrastructure health and application behavior separately.
5. Prepare rollback steps before touching production.
6. Use rolling changes for clustered services when possible.
7. Validate quorum, replication, storage, logs, client connectivity, and business flow after each step.
8. Keep legacy paths temporarily when staged migration lowers risk.
9. Remove old configuration only after all consumers are confirmed migrated.

## Example: Kafka mTLS migration

The secure Kafka rollout followed an additive approach:

```text
legacy PLAINTEXT listeners
          |
          +---- remain available temporarily
          |
          v
add parallel mTLS listener
          |
          v
issue broker/client identities
          |
          v
roll brokers one at a time
          |
          v
migrate applications
          |
          v
validate runtime + consumer groups + partitions
          |
          v
retire legacy access after verification
```

This reduced the risk of coupling broker security changes to application migration in one irreversible step.

## Verification layers

Production verification included, depending on the component:

- systemd/service state
- listening sockets and network reachability
- cluster quorum or replication health
- TLS and certificate validation
- application runtime properties
- Kubernetes workload readiness and logs
- database/service connectivity
- consumer-group or message-flow behavior
- end-to-end application/business flow

The operating principle was simple: a configuration change was not considered complete until the consuming application path was proven healthy.