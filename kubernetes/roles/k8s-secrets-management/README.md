# Kubernetes Secrets Management Role

Secure management of Kubernetes secrets with encryption validation and access controls.

## Features

- Secret creation and management
- Docker registry (image pull) secrets
- TLS secrets for ingress
- Encryption at rest validation
- Secret exposure scanning
- RBAC-based access control

## STIG Compliance

- V-242462: Encryption at rest for secrets
- V-242463: Secret protection and access control

## Example

```yaml
- role: k8s-secrets-management
  vars:
    k8s_secrets_encryption_enabled: true
    k8s_scan_for_exposed_secrets: true
```

## Security Notes

- All secret operations use `no_log: true`
- Validates encryption at rest is enabled
- Scans for accidentally exposed secrets in ConfigMaps
- Implements least privilege RBAC for secret access
