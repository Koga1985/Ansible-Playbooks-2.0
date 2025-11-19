# Kubernetes Ansible Roles

Production-ready Ansible roles for Kubernetes management with DoD STIG V1R11 and NIST SP 800-190 compliance.

## Available Roles

### k8s-cluster-hardening
Comprehensive cluster hardening implementing all DoD STIG and NIST controls.

**Features:**
- API Server security configuration
- Kubelet hardening
- ETCD encryption and authentication
- Audit logging
- Pod Security Standards
- Network policies
- Encryption at rest
- File permissions
- Compliance validation

**Usage:**
```yaml
- role: k8s-cluster-hardening
  vars:
    k8s_pod_security_standard: restricted
    k8s_enable_encryption_at_rest: true
```

### k8s-rbac-management
Role-Based Access Control management with least privilege principles.

**Features:**
- Service account management
- Role and ClusterRole creation
- Role bindings
- RBAC audit and reporting
- System:masters detection

**Usage:**
```yaml
- role: k8s-rbac-management
  vars:
    k8s_automount_sa_token: false
```

### k8s-namespace-management
Namespace lifecycle with quotas and limits.

**Features:**
- Namespace creation with labels
- Resource quotas
- Limit ranges
- Network policies per namespace
- Protected namespace management

**Usage:**
```yaml
- role: k8s-namespace-management
  vars:
    k8s_enable_resource_quotas: true
```

### k8s-secrets-management
Secure secrets management and validation.

**Features:**
- Secret creation and management
- Docker registry secrets
- TLS secrets
- Encryption validation
- Secret exposure scanning
- RBAC controls

**Usage:**
```yaml
- role: k8s-secrets-management
  vars:
    k8s_secrets_encryption_enabled: true
    k8s_scan_for_exposed_secrets: true
```

## Installation

Install required collections:
```bash
ansible-galaxy collection install -r requirements.yml
```

## Quick Start

1. Update inventory file with your Kubernetes control plane nodes
2. Run full setup:
```bash
ansible-playbook -i inventory playbook-full-setup.yml
```

3. Or run individual roles:
```bash
ansible-playbook -i inventory playbook-cluster-hardening.yml
```

## Compliance

All roles implement:
- **DoD STIG V1R11**: Complete coverage of Kubernetes STIG controls
- **NIST SP 800-190**: Application Container Security Guide
- **Pod Security Standards**: Restricted profile by default
- **CIS Benchmarks**: Kubernetes hardening guidelines

## Testing

Each role includes Molecule tests:
```bash
cd kubernetes/roles/k8s-cluster-hardening
molecule test
```

## Documentation

See individual role README files for detailed documentation.
