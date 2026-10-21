# Networking, Nginx, and TLS

Nginx served as the controlled edge and reverse-proxy layer for multiple application entry points rather than exposing backend services directly.

## Responsibilities

- TLS termination and certificate-based HTTPS exposure
- reverse proxying to internal application services
- route separation for mobile/API gateways, receipt/evidence endpoints, WebViews, merchant/backoffice interfaces, and identity services
- controlled exposure of multiple application domains
- verification of backend reachability, proxy routing, and application response behavior

## Simplified flow

```text
Mobile / Web / Merchant clients
            |
           TLS
            |
            v
          Nginx
    +-------+-------+
    |       |       |
    v       v       v
 Gateway  Web UI  Keycloak
    |
    v
 RKE2 services
```

## Operational checks

A successful Nginx reload was not treated as proof of service health. Validation included:

- configuration syntax
- certificate/key availability and permissions
- DNS and route correctness
- upstream reachability
- HTTP status and application response
- redirects and path rewriting where applicable
- logs from both Nginx and the backend service

## Security boundary

Public examples intentionally omit real hostnames, IP addresses, certificates, ciphers, and customer routing rules. The purpose of this document is to demonstrate the operational pattern, not reproduce a bank's edge configuration.
## Representative edge validation case

An anonymized UAT edge change combined several independently verified checks: corrected DNS hostnames, a shared public-IP/TLS entry point, internal backend health, and a restricted object-store receipt URL pattern. After a valid file was retrieved through the public proxy, additional negative HTTP tests confirmed that directory paths and POST/PUT/DELETE/PATCH/OPTIONS were denied.

**Security acceptance boundary:** a gateway generating a signed-looking URL is not evidence that Nginx or the storage service verifies its signature. Cryptographic receipt-specific authorization, expiry and cross-file denial remain independent go-live gates. Never publish actual receipt URLs, private IPs, tokens or customer documents.

Full sanitized procedure: [DNS/NAT/TLS edge and receipt-access runbook](https://github.com/natnaela-devops/devops-troubleshooting-runbooks/blob/main/runbooks/networking/nginx-dns-nat-tls-receipt-restrictions.md).
