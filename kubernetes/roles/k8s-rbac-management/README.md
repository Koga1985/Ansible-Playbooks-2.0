># Kubernetes RBAC Management Role

Production-ready Ansible role for managing Kubernetes RBAC with least privilege principles and DoD STIG compliance.

## Features

- Service account management with token automount controls
- Namespaced and cluster-wide role creation
- Role binding management
- RBAC audit and compliance reporting
- System:masters group detection
- Wildcard permission analysis

## Requirements

- Ansible 2.14+
- Kubernetes 1.24+
- `kubernetes.core` collection

## Example Usage

```yaml
---
- hosts: k8s_control_plane
  roles:
    - role: k8s-rbac-management
      vars:
        k8s_automount_sa_token: false
        k8s_namespaced_roles:
          - name: developer
            namespace: development
            rules:
              - apiGroups: ["apps"]
                resources: ["deployments"]
                verbs: ["get", "list", "create", "update"]
```

## STIG Controls

- V-242402: RBAC authorization mode
- V-242445: Service account token management

## License

MIT
