# Kubernetes Namespace Management Role

Manages Kubernetes namespaces with resource quotas, limit ranges, and network policies.

## Features

- Namespace creation with labels and annotations
- Resource quotas enforcement
- Limit ranges for containers
- Default network policies
- Pod Security Standards labels
- Protected namespace management

## Example

```yaml
- role: k8s-namespace-management
  vars:
    k8s_namespaces:
      - name: myapp
        labels:
          environment: prod
          pod-security.kubernetes.io/enforce: restricted
```

## STIG Controls

- V-242443: Resource quotas and limits
- Network segmentation via namespace isolation
