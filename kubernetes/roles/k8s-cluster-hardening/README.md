# Kubernetes Cluster Hardening Role

Production-ready Ansible role for hardening Kubernetes clusters according to DoD STIG V1R11 and NIST SP 800-190 standards.

## Description

This role applies comprehensive security hardening to Kubernetes clusters, implementing:
- API Server hardening with audit logging
- Kubelet security configuration
- ETCD encryption and authentication
- Pod Security Standards (PSS)
- Network policies
- Encryption at rest for secrets
- File permissions and ownership
- Compliance validation

## Requirements

- Ansible 2.14 or higher
- Kubernetes 1.24 or higher
- kubectl installed and configured
- Access to Kubernetes cluster with admin privileges
- `kubernetes.core` collection

## Role Variables

### API Server Configuration
```yaml
k8s_api_server_audit_log_enabled: true
k8s_api_server_audit_log_path: /var/log/kubernetes/audit.log
k8s_api_server_audit_log_maxage: 30
k8s_tls_min_version: VersionTLS12
```

### Kubelet Configuration
```yaml
k8s_kubelet_anonymous_auth: false
k8s_kubelet_authorization_mode: Webhook
k8s_kubelet_read_only_port: 0
```

### Pod Security
```yaml
k8s_pod_security_standard: restricted  # privileged, baseline, or restricted
k8s_enforce_pod_security_standards: true
```

### Encryption
```yaml
k8s_enable_encryption_at_rest: true
k8s_encryption_provider_config: /etc/kubernetes/encryption-config.yaml
```

See `defaults/main.yml` for complete variable list.

## Dependencies

Install required Ansible collection:
```bash
ansible-galaxy collection install kubernetes.core
```

## Example Playbook

```yaml
---
- name: Harden Kubernetes Cluster
  hosts: k8s_control_plane
  become: true
  roles:
    - role: k8s-cluster-hardening
      vars:
        k8s_pod_security_standard: restricted
        k8s_enable_encryption_at_rest: true
```

## STIG Controls Implemented

| STIG ID | Control | Implementation |
|---------|---------|----------------|
| V-242381 | Audit logging must be enabled | audit-logging.yml |
| V-242382 | Audit logs must be retained for 30 days | defaults/main.yml |
| V-242400 | TLS 1.2 minimum | api-server-hardening.yml |
| V-242402 | Anonymous auth disabled | api-server-hardening.yml |
| V-242417 | Admission controllers enabled | api-server-hardening.yml |
| V-242424-433 | Kubelet hardening | kubelet-hardening.yml |
| V-242434-437 | ETCD security | etcd-hardening.yml |
| V-242450-458 | File permissions | file-permissions.yml |
| V-242462-463 | Encryption at rest | encryption-at-rest.yml |

## NIST SP 800-190 Controls

- **Container Isolation**: Pod Security Standards, security contexts
- **Network Segmentation**: Network policies, namespace isolation
- **Secure Orchestration**: RBAC, admission controllers
- **Logging & Monitoring**: Audit logs, auditd integration
- **Runtime Defense**: Seccomp, AppArmor, read-only root filesystem

## Usage

### Basic Hardening
```bash
ansible-playbook -i inventory site.yml --tags hardening
```

### Specific Components
```bash
# API Server only
ansible-playbook -i inventory site.yml --tags api-server

# Kubelet only
ansible-playbook -i inventory site.yml --tags kubelet

# Run compliance validation
ansible-playbook -i inventory site.yml --tags compliance-validation
```

## Validation

Run compliance validation:
```bash
ansible-playbook -i inventory site.yml --tags validation
```

This generates a compliance report at `/tmp/k8s-compliance-report.html`.

## Testing

Molecule tests are included:
```bash
cd kubernetes/roles/k8s-cluster-hardening
molecule test
```

## Security Considerations

1. **Backup Before Changes**: This role modifies critical Kubernetes components
2. **Test in Non-Production**: Validate in test environment first
3. **Encryption Keys**: Securely manage encryption keys
4. **Certificate Rotation**: Implement automated certificate rotation
5. **Audit Log Storage**: Ensure sufficient storage for audit logs

## Troubleshooting

### API Server Won't Start
- Check `/var/log/kubernetes/audit.log` for errors
- Verify audit policy file syntax
- Check encryption config file

### Kubelet Fails to Start
- Review `/var/log/syslog` or `journalctl -u kubelet`
- Verify certificate paths
- Check file permissions on kubelet config

### Pods Rejected by PSS
- Review Pod Security Standard level (restricted is most stringent)
- Check pod security context configuration
- Use baseline or privileged for specific namespaces if needed

## License

MIT

## Author

DevOps Team
