# Messaging and Cache: Kafka + Redis/Predixy

## Kafka/KRaft

Kafka operated as a multi-node KRaft cluster. Application security was improved through a staged migration from legacy PLAINTEXT listeners to a dedicated mTLS listener using per-broker certificates and separate client identities.

Validation covered:

- KRaft quorum health and follower lag
- broker listener availability
- TLS handshake and certificate verification
- PKCS12/JKS client stores
- runtime Java/Spring/WildFly client configuration
- consumer-group joins and partition assignments
- multi-broker metadata and message flow

A key troubleshooting principle was to separate **cluster health** from **client configuration health**. A healthy quorum did not prove a producer/consumer was using the right protocol, truststore, keystore, password, or runtime property.

## Redis Cluster + Predixy

Redis operated as a clustered master/replica topology, with Predixy acting as the application-facing proxy/routing layer.

```text
Application
    |
    v
 Predixy
    |
    +--> Redis master
    +--> Redis master
    +--> Redis master
          |
          +--> replicas
```

Operational work included:

- validating cluster state and slot distribution
- handling `MOVED` behavior and cluster-aware routing
- validating Redis authentication through the application path
- confirming Predixy connectivity and routing
- distinguishing proxy/authentication issues from Redis cluster health

## Security

Public examples do not include real broker addresses, Redis credentials, certificates, keystores, cluster IDs, or application secrets. Detailed sanitized procedures are documented in the companion troubleshooting repository.