# Red Hat Enterprise Linux (RHEL) Automation

Production-ready Ansible playbooks, roles, and tasks for Red Hat Enterprise Linux automation with DoD STIG and NIST compliance.

## Overview

This directory contains comprehensive automation for RHEL systems including:
- System hardening (DoD STIG and NIST SP 800-53 compliant)
- Patch management and updates
- Firewall and SELinux configuration
- User and access management
- Audit and logging configuration
- Security baseline enforcement

## Supported RHEL Versions

- RHEL 8.x (8.6+)
- RHEL 9.x (9.0+)

## Compliance Standards

All roles and tasks are designed to meet:
- **DoD STIG for RHEL** (V1R14 for RHEL 8, V1R3 for RHEL 9)
- **NIST SP 800-53 Rev 5** security controls
- **CIS Red Hat Enterprise Linux Benchmark** Level 1 and Level 2

## Directory Structure

```
rhel/
├── README.md                    # This file
├── requirements.yml             # Ansible collections dependencies
├── inventory.example            # Example inventory file
├── roles/                       # Reusable roles
│   ├── rhel-hardening/         # System hardening (STIG/NIST)
│   ├── rhel-patch-management/  # Patch and update management
│   ├── rhel-firewall-selinux/  # Firewall and SELinux config
│   ├── rhel-user-access/       # User and access management
│   └── rhel-audit-logging/     # Audit and logging configuration
├── tasks/                       # Standalone tasks
│   ├── register-system.yml     # System registration
│   ├── update-packages.yml     # Package updates
│   ├── configure-ssh.yml       # SSH hardening
│   ├── configure-firewall.yml  # Firewall rules
│   ├── configure-selinux.yml   # SELinux configuration
│   ├── configure-audit.yml     # Audit daemon setup
│   ├── configure-aide.yml      # File integrity monitoring
│   └── kernel-tuning.yml       # Kernel security parameters
└── playbooks/                   # Complete playbooks
    ├── system-hardening.yml    # Full system hardening
    ├── compliance-check.yml    # Compliance validation
    └── patch-management.yml    # Patch management
```

## Quick Start

### Prerequisites

1. **Install required collections:**
   ```bash
   ansible-galaxy collection install -r requirements.yml
   ```

2. **Configure inventory:**
   ```bash
   cp inventory.example inventory
   # Edit inventory with your RHEL hosts
   ```

3. **Set up credentials:**
   - Use Ansible Vault for sensitive data
   - Configure SSH keys or use `--ask-become-pass`

### Example Usage

#### 1. System Hardening (Full STIG/NIST Compliance)

```bash
# Apply full system hardening
ansible-playbook playbooks/system-hardening.yml -i inventory

# Dry run with check mode
ansible-playbook playbooks/system-hardening.yml -i inventory --check --diff

# Apply specific hardening areas
ansible-playbook playbooks/system-hardening.yml -i inventory --tags "ssh,firewall"
```

#### 2. Patch Management

```bash
# Update all packages
ansible-playbook playbooks/patch-management.yml -i inventory

# Security updates only
ansible-playbook playbooks/patch-management.yml -i inventory --tags "security-updates"

# Update without reboot
ansible-playbook playbooks/patch-management.yml -i inventory --skip-tags "reboot"
```

#### 3. Compliance Validation

```bash
# Run compliance checks without making changes
ansible-playbook playbooks/compliance-check.yml -i inventory

# Generate compliance report
ansible-playbook playbooks/compliance-check.yml -i inventory --tags "report"
```

#### 4. Using Individual Roles

```yaml
---
- name: Harden RHEL systems
  hosts: rhel_servers
  become: true

  roles:
    - role: rhel/roles/rhel-hardening
      vars:
        rhel_stig_level: high
        rhel_enable_fips: true

    - role: rhel/roles/rhel-firewall-selinux
      vars:
        rhel_selinux_state: enforcing
```

#### 5. Using Standalone Tasks

```yaml
---
- name: Configure SSH hardening
  hosts: rhel_servers
  become: true

  tasks:
    - name: Apply SSH hardening
      ansible.builtin.include_tasks: rhel/tasks/configure-ssh.yml
```

## Roles

### rhel-hardening

Comprehensive system hardening following DoD STIG and NIST SP 800-53.

**Features:**
- File system permissions and mount options
- SSH hardening (STIG V-204392 through V-204612)
- Password policies and account security
- Service hardening and removal of unnecessary services
- Network stack hardening
- Kernel parameter tuning
- FIPS 140-2 mode enablement
- Sudo configuration
- PAM configuration
- Login banner and warning messages

**Variables:**
- `rhel_stig_level`: Compliance level (low/medium/high/critical)
- `rhel_enable_fips`: Enable FIPS mode (true/false)
- `rhel_remove_unnecessary_services`: Remove unused services (true/false)
- `rhel_ssh_permit_root_login`: SSH root login (true/false)
- `rhel_password_min_length`: Minimum password length (14)

**Tags:**
- `hardening`, `stig`, `nist`, `ssh`, `network`, `filesystem`, `accounts`

### rhel-patch-management

Automated patch management with security update prioritization.

**Features:**
- Security update prioritization
- Kernel and system updates
- Package cleanup
- Reboot management with configurable scheduling
- Update history tracking
- Rollback capability preparation
- Subscription management integration

**Variables:**
- `rhel_auto_reboot`: Enable automatic reboot after updates (false)
- `rhel_reboot_delay`: Delay before reboot in seconds (300)
- `rhel_security_updates_only`: Only install security updates (false)
- `rhel_exclude_packages`: List of packages to exclude from updates

**Tags:**
- `patching`, `updates`, `security-updates`, `reboot`, `kernel`

### rhel-firewall-selinux

Firewall and SELinux configuration management.

**Features:**
- Firewalld configuration and zone management
- SELinux policy enforcement
- Custom SELinux policies
- Port and service management
- Rich rules configuration
- SELinux troubleshooting setup
- Audit2allow integration

**Variables:**
- `rhel_selinux_state`: SELinux mode (enforcing/permissive/disabled)
- `rhel_selinux_policy`: SELinux policy type (targeted/mls)
- `rhel_firewall_default_zone`: Default firewall zone (public)
- `rhel_firewall_services`: List of services to allow
- `rhel_firewall_ports`: List of ports to open

**Tags:**
- `firewall`, `selinux`, `security`, `network`

### rhel-user-access

User and access management with least privilege enforcement.

**Features:**
- User account creation and management
- Group management
- Sudo privilege configuration
- SSH key management
- Account expiration policies
- Password aging
- Disabled account cleanup
- Root access restrictions

**Variables:**
- `rhel_users`: List of users to create/manage
- `rhel_sudo_groups`: Groups with sudo access
- `rhel_password_max_days`: Maximum password age (60)
- `rhel_account_inactive_days`: Days until inactive account disabled (35)
- `rhel_lock_inactive_accounts`: Auto-lock inactive accounts (true)

**Tags:**
- `users`, `access`, `sudo`, `accounts`, `ssh-keys`

### rhel-audit-logging

Audit daemon and system logging configuration.

**Features:**
- Auditd configuration (STIG compliant)
- Audit rules for security events
- Log rotation and retention
- Remote logging configuration
- File integrity monitoring (AIDE)
- Rsyslog hardening
- Log file permissions
- Audit log monitoring

**Variables:**
- `rhel_audit_log_retention_days`: Days to retain audit logs (90)
- `rhel_audit_space_left_action`: Action when disk space low (email)
- `rhel_remote_logging_enabled`: Enable remote logging (false)
- `rhel_remote_log_server`: Remote syslog server address
- `rhel_aide_check_schedule`: AIDE check cron schedule (daily)

**Tags:**
- `audit`, `logging`, `aide`, `syslog`, `monitoring`

## Standalone Tasks

Located in `tasks/` directory, these can be included in any playbook:

| Task File | Description | STIG References |
|-----------|-------------|-----------------|
| `register-system.yml` | Register system with Red Hat Subscription Manager | V-204392 |
| `update-packages.yml` | Update system packages and kernel | V-204393 |
| `configure-ssh.yml` | SSH hardening configuration | V-204596 through V-204612 |
| `configure-firewall.yml` | Firewalld configuration | V-204500 through V-204510 |
| `configure-selinux.yml` | SELinux policy configuration | V-204401 |
| `configure-audit.yml` | Auditd configuration | V-204494 through V-204644 |
| `configure-aide.yml` | AIDE file integrity monitoring | V-204447 |
| `kernel-tuning.yml` | Kernel security parameters | V-204480 through V-204493 |

## Playbooks

### system-hardening.yml

Complete system hardening playbook applying all security controls.

```bash
# Full hardening
ansible-playbook playbooks/system-hardening.yml -i inventory

# Specific components
ansible-playbook playbooks/system-hardening.yml -i inventory --tags "ssh,audit"

# Check mode (dry run)
ansible-playbook playbooks/system-hardening.yml -i inventory --check --diff
```

### compliance-check.yml

Validate system compliance without making changes.

```bash
# Run compliance checks
ansible-playbook playbooks/compliance-check.yml -i inventory

# Generate report
ansible-playbook playbooks/compliance-check.yml -i inventory --tags "report" > compliance-report.txt
```

### patch-management.yml

System update and patch management.

```bash
# Full system update
ansible-playbook playbooks/patch-management.yml -i inventory

# Security updates only
ansible-playbook playbooks/patch-management.yml -i inventory -e "rhel_security_updates_only=true"
```

## Variables and Configuration

### Global Variables

Create `group_vars/all.yml` or `group_vars/rhel_servers.yml`:

```yaml
---
# RHEL Global Configuration

# Compliance Level
rhel_stig_level: high                    # low, medium, high, critical
rhel_apply_nist_controls: true
rhel_apply_cis_benchmark: true

# FIPS Mode
rhel_enable_fips: true

# SSH Configuration
rhel_ssh_permit_root_login: false
rhel_ssh_password_authentication: false
rhel_ssh_max_auth_tries: 3
rhel_ssh_max_sessions: 10

# Password Policies
rhel_password_min_length: 14
rhel_password_max_days: 60
rhel_password_min_days: 1
rhel_password_warn_age: 7

# SELinux
rhel_selinux_state: enforcing
rhel_selinux_policy: targeted

# Firewall
rhel_firewall_enabled: true
rhel_firewall_default_zone: public

# Audit and Logging
rhel_audit_enabled: true
rhel_audit_log_retention_days: 90
rhel_remote_logging_enabled: false

# Patch Management
rhel_auto_reboot: false
rhel_reboot_delay: 300
rhel_security_updates_only: false
```

### Host-Specific Variables

Create `host_vars/hostname.yml`:

```yaml
---
# Host-specific overrides
rhel_firewall_services:
  - ssh
  - http
  - https

rhel_firewall_ports:
  - 8080/tcp
  - 8443/tcp

rhel_users:
  - name: appuser
    groups: wheel
    ssh_key: "{{ lookup('file', '~/.ssh/id_rsa.pub') }}"
```

## Tags

All roles and tasks support granular execution with tags:

### Compliance Tags
- `stig`: DoD STIG controls
- `nist`: NIST SP 800-53 controls
- `cis`: CIS Benchmark controls
- `compliance`: All compliance checks
- `validation`: Validation tasks only

### Functional Tags
- `hardening`: System hardening tasks
- `security`: Security-related tasks
- `ssh`: SSH configuration
- `firewall`: Firewall configuration
- `selinux`: SELinux configuration
- `audit`: Audit configuration
- `logging`: Logging configuration
- `users`: User management
- `accounts`: Account security
- `network`: Network hardening
- `filesystem`: File system security
- `patching`: Patch management
- `updates`: System updates
- `kernel`: Kernel updates and tuning

### Execution Tags
- `preflight`: Pre-execution validation
- `report`: Generate reports
- `never`: Tasks that only run when explicitly tagged

### Example Tag Usage

```bash
# Apply only SSH and firewall hardening
ansible-playbook playbooks/system-hardening.yml --tags "ssh,firewall"

# Apply all STIG controls
ansible-playbook playbooks/system-hardening.yml --tags "stig"

# Skip audit configuration
ansible-playbook playbooks/system-hardening.yml --skip-tags "audit"

# Run validation only
ansible-playbook playbooks/compliance-check.yml --tags "validation"
```

## Testing

All roles include Molecule tests for validation:

```bash
# Test a specific role
cd roles/rhel-hardening
molecule test

# Run converge only (no destroy)
molecule converge

# Run verify tests
molecule verify

# Test on specific platform
molecule test --scenario-name rhel8
```

## Security Considerations

### Secrets Management

**Never commit plaintext secrets.** Use Ansible Vault:

```bash
# Create encrypted variable file
ansible-vault create group_vars/rhel_servers/vault.yml

# Edit encrypted file
ansible-vault edit group_vars/rhel_servers/vault.yml

# Run playbook with vault
ansible-playbook playbooks/system-hardening.yml --ask-vault-pass
```

### FIPS Mode

Enabling FIPS mode requires a reboot and may impact:
- SSH connections (non-FIPS ciphers disabled)
- Application compatibility
- Performance (cryptographic operations)

Test thoroughly before production deployment.

### SELinux

Transitioning SELinux modes:
1. **Permissive → Enforcing**: Monitor audit logs first
2. **Disabled → Enforcing**: Requires reboot and relabeling
3. Always test in non-production first

### Network Impact

Firewall changes may impact connectivity. Always:
1. Ensure SSH access is maintained
2. Test with `--check` mode first
3. Have console/out-of-band access
4. Stage rollout with `--limit`

## Staged Rollout

For production environments, use gradual rollout:

```bash
# Test on single host
ansible-playbook playbooks/system-hardening.yml --limit "rhel-test-01"

# Deploy to subset
ansible-playbook playbooks/system-hardening.yml --limit "rhel-dev*"

# Deploy to production with serial execution
ansible-playbook playbooks/system-hardening.yml --limit "rhel-prod*" --forks 1
```

## Validation and Compliance Checking

### Manual Validation

```bash
# Check STIG compliance
sudo oscap xccdf eval --profile xccdf_org.ssgproject.content_profile_stig \
  /usr/share/xml/scap/ssg/content/ssg-rhel8-ds.xml

# Check CIS compliance
sudo oscap xccdf eval --profile xccdf_org.ssgproject.content_profile_cis \
  /usr/share/xml/scap/ssg/content/ssg-rhel8-ds.xml
```

### Ansible Playbook Validation

```bash
# Run compliance validation playbook
ansible-playbook playbooks/compliance-check.yml -i inventory

# Generate detailed report
ansible-playbook playbooks/compliance-check.yml -i inventory --tags "report" | tee compliance-$(date +%Y%m%d).txt
```

## Troubleshooting

### Common Issues

**Issue: SSH connection fails after hardening**
- Check SSH configuration: `/etc/ssh/sshd_config`
- Verify allowed ciphers and key exchange algorithms
- Check firewall rules: `firewall-cmd --list-all`

**Issue: SELinux denials**
- Check audit logs: `ausearch -m AVC -ts recent`
- Generate policy: `audit2allow -a`
- Set to permissive temporarily: `setenforce 0`

**Issue: System won't boot after FIPS enablement**
- Boot into rescue mode
- Disable FIPS: `fips-mode-setup --disable`
- Review FIPS requirements

**Issue: Package updates fail**
- Check subscription: `subscription-manager status`
- Refresh repositories: `dnf clean all && dnf makecache`
- Check available space: `df -h`

## Support Matrix

| RHEL Version | DoD STIG Version | SCAP Content | Status |
|--------------|------------------|--------------|--------|
| RHEL 8.6+ | V1R14 | ssg-rhel8-ds.xml | Tested |
| RHEL 9.0+ | V1R3 | ssg-rhel9-ds.xml | Tested |

## References

### DoD STIG
- [RHEL 8 STIG](https://public.cyber.mil/stigs/downloads/?_dl_facet_stigs=operating-systems%2Cunix-linux)
- [RHEL 9 STIG](https://public.cyber.mil/stigs/downloads/?_dl_facet_stigs=operating-systems%2Cunix-linux)

### NIST
- [NIST SP 800-53 Rev 5](https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final)
- [NIST SP 800-123](https://csrc.nist.gov/publications/detail/sp/800-123/final) - Guide to General Server Security

### CIS Benchmarks
- [CIS Red Hat Enterprise Linux 8 Benchmark](https://www.cisecurity.org/benchmark/red_hat_linux)
- [CIS Red Hat Enterprise Linux 9 Benchmark](https://www.cisecurity.org/benchmark/red_hat_linux)

### Red Hat Security
- [Red Hat Security Guide](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/security_hardening/index)
- [SCAP Security Guide](https://www.open-scap.org/security-policies/scap-security-guide/)

## Contributing

When adding new roles or tasks:
1. Follow existing patterns and directory structure
2. Include STIG/NIST references in comments
3. Add Molecule tests
4. Update this README
5. Test on multiple RHEL versions
6. Document all variables in `defaults/main.yml`

## License

This automation is provided as-is for use in securing RHEL systems. Review and test thoroughly before production use.

## Disclaimer

While these playbooks are designed to meet DoD STIG and NIST compliance requirements, organizations should:
1. Review all configurations against their specific security requirements
2. Test thoroughly in non-production environments
3. Conduct security assessments after implementation
4. Maintain ongoing compliance monitoring
5. Adjust configurations based on organizational policies

Compliance is an ongoing process requiring regular updates and validation.
