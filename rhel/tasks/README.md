# RHEL Standalone Tasks

This directory contains standalone, reusable Ansible tasks for common RHEL operations. These tasks can be included in any playbook for specific operations without using full roles.

## Available Tasks

### register-system.yml
Register RHEL system with Red Hat Subscription Manager.

**STIG Reference**: V-204392

**Usage:**
```yaml
- name: Register system
  ansible.builtin.include_tasks: rhel/tasks/register-system.yml
  vars:
    rhsm_username: "{{ vault_rhsm_username }}"
    rhsm_password: "{{ vault_rhsm_password }}"
```

### update-packages.yml
Update system packages and check for kernel updates.

**STIG Reference**: V-204393

**Usage:**
```yaml
- name: Update all packages
  ansible.builtin.include_tasks: rhel/tasks/update-packages.yml
```

### configure-ssh.yml
Apply SSH hardening configuration following STIG requirements.

**STIG Reference**: V-204596 through V-204612

**Usage:**
```yaml
- name: Harden SSH
  ansible.builtin.include_tasks: rhel/tasks/configure-ssh.yml
```

**Configuration applied:**
- Disable root login
- Disable password authentication
- Disable X11 forwarding
- Set MaxAuthTries to 3
- Configure session timeouts

### configure-firewall.yml
Configure firewalld with basic security settings.

**STIG Reference**: V-204500 through V-204510

**Usage:**
```yaml
- name: Configure firewall
  ansible.builtin.include_tasks: rhel/tasks/configure-firewall.yml
```

### configure-selinux.yml
Configure SELinux to enforcing mode.

**STIG Reference**: V-204401

**Usage:**
```yaml
- name: Configure SELinux
  ansible.builtin.include_tasks: rhel/tasks/configure-selinux.yml
```

### configure-audit.yml
Install and configure auditd service.

**STIG Reference**: V-204494 through V-204644

**Usage:**
```yaml
- name: Configure audit
  ansible.builtin.include_tasks: rhel/tasks/configure-audit.yml
```

### configure-aide.yml
Install and configure AIDE for file integrity monitoring.

**STIG Reference**: V-204447

**Usage:**
```yaml
- name: Configure AIDE
  ansible.builtin.include_tasks: rhel/tasks/configure-aide.yml
```

**Note:** AIDE database must be initialized manually:
```bash
aide --init
mv /var/lib/aide/aide.db.new.gz /var/lib/aide/aide.db.gz
```

### kernel-tuning.yml
Apply kernel security parameters.

**STIG Reference**: V-204480 through V-204525

**Usage:**
```yaml
- name: Apply kernel tuning
  ansible.builtin.include_tasks: rhel/tasks/kernel-tuning.yml
```

**Parameters configured:**
- Address Space Layout Randomization (ASLR)
- Kernel pointer restrictions
- Network security settings
- IP forwarding disabled
- SYN cookie protection

## Example Playbook

```yaml
---
- name: Apply specific RHEL hardening tasks
  hosts: rhel_servers
  become: true
  gather_facts: true

  tasks:
    - name: Harden SSH configuration
      ansible.builtin.include_tasks: rhel/tasks/configure-ssh.yml
      tags:
        - ssh
        - hardening

    - name: Configure firewall
      ansible.builtin.include_tasks: rhel/tasks/configure-firewall.yml
      tags:
        - firewall
        - hardening

    - name: Apply kernel security parameters
      ansible.builtin.include_tasks: rhel/tasks/kernel-tuning.yml
      tags:
        - kernel
        - hardening

    - name: Install and configure AIDE
      ansible.builtin.include_tasks: rhel/tasks/configure-aide.yml
      tags:
        - aide
        - integrity
```

## Tags

All tasks support the following tags:
- `validation`: Run validation checks
- Task-specific tags (ssh, firewall, selinux, audit, aide, kernel)

## Best Practices

1. **Check Mode**: Always test with `--check --diff` first
2. **Staged Rollout**: Use `--limit` for gradual deployment
3. **Backup**: Tasks include automatic backups where applicable
4. **Validation**: Each task includes validation steps
5. **Idempotency**: All tasks are idempotent and safe to run multiple times

## Security Considerations

- SSH hardening will disable password authentication - ensure SSH keys are configured
- Firewall changes may impact connectivity - always have console access
- SELinux mode changes may require relabeling and reboot
- Kernel parameter changes take effect immediately

## Related Documentation

See the main [RHEL README](../README.md) for:
- Complete role documentation
- Compliance frameworks
- Full playbook examples
- Troubleshooting guide
