# Kubernetes Ansible Playbooks

Production-ready, drop-in Ansible playbooks and roles for Kubernetes cluster management with full DoD STIG V1R11 and NIST SP 800-190 compliance.

## Overview

This collection provides comprehensive Kubernetes automation with enterprise-grade security controls:

- **DoD STIG V1R11 Compliant**: Full implementation of Kubernetes STIG controls
- **NIST SP 800-190 Compliant**: Application Container Security Guide
- **Production Ready**: Battle-tested configurations for enterprise environments
- **Drop-in**: Minimal configuration required, works out of the box
- **Functionally Tested**: Includes Molecule tests for all roles

## Directory Structure

```
kubernetes/
├── roles/                          # Ansible roles
│   ├── k8s-cluster-hardening/     # Complete cluster security hardening
│   ├── k8s-rbac-management/       # RBAC and access control
│   ├── k8s-namespace-management/  # Namespace lifecycle management
│   └── k8s-secrets-management/    # Secure secrets handling
├── tasks/                          # Reusable tasks
│   ├── deploy-application.yml     # Secure app deployment
│   ├── backup-resources.yml       # Cluster backup
│   ├── health-check.yml           # Health monitoring
│   ├── scale-deployment.yml       # Scaling operations
│   └── rolling-update.yml         # Zero-downtime updates
├── playbook-cluster-hardening.yml # Cluster hardening playbook
├── playbook-deploy-app.yml        # Application deployment playbook
├── playbook-full-setup.yml        # Complete setup playbook
├── inventory.example              # Example inventory file
└── requirements.yml               # Ansible collection requirements

```

## Quick Start

### 1. Install Dependencies

```bash
# Install required Ansible collections
ansible-galaxy collection install -r requirements.yml

# Install kubectl if not already installed
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

### 2. Configure Inventory

Copy and customize the inventory file:
```bash
cp inventory.example inventory
# Edit inventory with your Kubernetes control plane IPs
```

### 3. Run Playbooks

Complete cluster setup with all security controls:
```bash
ansible-playbook -i inventory playbook-full-setup.yml
```

Or run specific components:
```bash
# Cluster hardening only
ansible-playbook -i inventory playbook-cluster-hardening.yml

# Deploy an application
ansible-playbook -i inventory playbook-deploy-app.yml
```

## Features

### Cluster Hardening (k8s-cluster-hardening)

- ✅ API Server hardening with audit logging
- ✅ Kubelet security configuration
- ✅ ETCD encryption and authentication
- ✅ TLS 1.2+ enforcement
- ✅ Pod Security Standards (restricted profile)
- ✅ Network policies
- ✅ Encryption at rest for secrets
- ✅ File permissions and ownership (STIG compliant)
- ✅ Admission controllers
- ✅ Compliance validation and reporting

### RBAC Management (k8s-rbac-management)

- ✅ Service account token controls
- ✅ Namespaced and cluster roles
- ✅ Role bindings with least privilege
- ✅ RBAC audit and reporting
- ✅ System:masters detection
- ✅ Wildcard permission analysis

### Namespace Management (k8s-namespace-management)

- ✅ Namespace creation with Pod Security Standards labels
- ✅ Resource quotas per namespace
- ✅ Limit ranges for containers
- ✅ Default network policies
- ✅ Protected namespace management

### Secrets Management (k8s-secrets-management)

- ✅ Encrypted secrets (at rest validation)
- ✅ Docker registry credentials
- ✅ TLS certificate management
- ✅ Secret exposure scanning
- ✅ RBAC-based access control

### Reusable Tasks

- ✅ Secure application deployment
- ✅ Cluster resource backup
- ✅ Comprehensive health checks
- ✅ Deployment scaling
- ✅ Rolling updates with auto-rollback

## DoD STIG Controls Implemented

| STIG ID | Control Description | Implementation |
|---------|---------------------|----------------|
| V-242381 | Audit logging enabled | audit-logging.yml |
| V-242382 | Audit log retention 30+ days | Configured in defaults |
| V-242400 | TLS 1.2 minimum | api-server-hardening.yml |
| V-242402 | Anonymous auth disabled | api-server-hardening.yml |
| V-242417 | Admission controllers enabled | api-server-hardening.yml |
| V-242424-433 | Kubelet hardening | kubelet-hardening.yml |
| V-242434-437 | ETCD security | etcd-hardening.yml |
| V-242443 | Resource quotas | namespace-management |
| V-242445 | Service account tokens | rbac-management |
| V-242450-458 | File permissions | file-permissions.yml |
| V-242462-463 | Encryption at rest | encryption-at-rest.yml |

## NIST SP 800-190 Controls

- **Image Security**: Container image scanning and signature verification
- **Registry Security**: Secure registry configuration
- **Orchestrator Security**: Kubernetes hardening and RBAC
- **Container Security**: Pod Security Standards, security contexts
- **Host Security**: Node hardening, file permissions
- **Network Security**: Network policies, TLS enforcement

## Usage Examples

### Example 1: Harden Existing Cluster

```yaml
---
- hosts: k8s_control_plane
  roles:
    - role: kubernetes/roles/k8s-cluster-hardening
      vars:
        k8s_pod_security_standard: restricted
        k8s_enable_encryption_at_rest: true
```

### Example 2: Deploy Secure Application

```yaml
---
- hosts: localhost
  tasks:
    - include_tasks: kubernetes/tasks/deploy-application.yml
      vars:
        app_name: myapp
        app_namespace: production
        app_image: registry1.dso.mil/myapp:1.0.0
        app_replicas: 3
```

### Example 3: Setup RBAC

```yaml
---
- hosts: k8s_control_plane
  roles:
    - role: kubernetes/roles/k8s-rbac-management
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

## Testing

All roles include Molecule tests:

```bash
# Test cluster hardening role
cd kubernetes/roles/k8s-cluster-hardening
molecule test

# Test RBAC management role
cd kubernetes/roles/k8s-rbac-management
molecule test
```

## Compliance Validation

Run compliance validation to generate a detailed report:

```bash
ansible-playbook -i inventory playbook-cluster-hardening.yml --tags validation
```

This generates:
- `/tmp/k8s-compliance-report.html` - Detailed compliance report
- `/tmp/k8s-rbac-audit-report.txt` - RBAC audit report

## Security Considerations

1. **Backup First**: Always backup your cluster before applying changes
2. **Test in Non-Production**: Validate in test environment before production
3. **Review Defaults**: Adjust variables in `defaults/main.yml` for your environment
4. **Certificate Management**: Ensure certificate rotation is configured
5. **Audit Logs**: Monitor disk space for audit logs (default 30 day retention)
6. **Network Policies**: Default deny-all may break existing apps - review carefully

## Troubleshooting

### API Server Won't Start
```bash
# Check audit log
tail -f /var/log/kubernetes/audit.log

# Verify configuration
kubectl get --raw /healthz
```

### Pods Rejected by Pod Security Standards
```bash
# Check namespace PSS level
kubectl get ns -o json | jq '.items[] | {name: .metadata.name, enforce: .metadata.labels["pod-security.kubernetes.io/enforce"]}'

# Adjust namespace PSS level if needed
kubectl label ns myapp pod-security.kubernetes.io/enforce=baseline --overwrite
```

### RBAC Permission Denied
```bash
# Check user permissions
kubectl auth can-i --list

# Review RBAC audit
cat /tmp/k8s-rbac-audit-report.txt
```

## Requirements

- Ansible 2.14+
- Kubernetes 1.24+ (for Pod Security Standards)
- kubectl configured with admin access
- Python 3.8+
- Root/sudo access on control plane nodes

## License

MIT

## Contributing

Contributions are welcome! Please ensure:
- All changes maintain STIG/NIST compliance
- Molecule tests pass
- Documentation is updated
- Security best practices are followed

## Support

For issues and questions:
1. Check the troubleshooting section
2. Review individual role README files
3. Examine example playbooks
4. Check Ansible output for detailed error messages

## References

- [Kubernetes STIG V1R11](https://www.stigviewer.com/stig/kubernetes/)
- [NIST SP 800-190](https://csrc.nist.gov/publications/detail/sp/800-190/final)
- [Kubernetes Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)
- [CIS Kubernetes Benchmark](https://www.cisecurity.org/benchmark/kubernetes)
