# vmware_esxi_stig_hardening (role)

Apply a practical **STIG/hardening** baseline to ESXi hosts, targeting either:
- a **single host** (`harden_scope: single`), or
- **all hosts in a cluster** (`harden_scope: cluster`).

## What it enforces
- **Lockdown mode** (normal/strict) with optional **exception users**
- **Disable SSH / ESXi Shell** by default (policies configurable)
- **Acceptance level** (vmware_accepted by default)
- **NTP servers** + service policy
- **Remote syslog** and `Syslog.global.logDirUnique`
- A set of **advanced host configuration** keys (timeouts, password quality, warnings)
- Optional **firewall ruleset** adjustments
- Optional exit from **maintenance mode**

## Requirements
- Ansible >= 2.15
- Collection: `community.vmware`
- Control node Python: `pyvmomi`

## Key variables (see `defaults/main.yml`)
```yaml
harden_scope: "single" | "cluster"

# Single
esxi_hostname: "esxi01.example.local"

# Cluster
vcenter_hostname: "vcenter.example.local"
vcenter_username: "{{ lookup('env','VCENTER_USERNAME') }}"
vcenter_password: "{{ lookup('env','VCENTER_PASSWORD') }}"
vcenter_validate_certs: false
vcenter_datacenter: "DC1"
vcenter_cluster: "Compute-Cluster"

# Common access
esxi_username: "root"
esxi_password: "{{ vault_esxi_root_password }}"

lockdown_enable: true
lockdown_mode: "normal"               # normal|strict
lockdown_exception_users: []

ssh_policy: "off"                      # on|off|automatic
esxi_shell_policy: "off"

acceptance_level: "vmware_accepted"

ntp_servers: ["time1.example.local","time2.example.local"]
ntp_policy: "on"

syslog_remote: "udp://10.0.0.5:514"
syslog_dir_unique: true

advanced_config:
  "UserVars.ESXiShellTimeOut": 600
  "UserVars.ESXiShellInteractiveTimeOut": 900
  "UserVars.SuppressShellWarning": 1
  "Security.PasswordQualityControl": "retry=3 min=disabled,15,3"
  "UserVars.SuppressDCOEnabledWarning": 0

firewall_rulesets:
  - { name: "sshServer", enabled: false, allowed_hosts: "ALL" }
```

## Example runners

**Harden one host**
```yaml
- hosts: localhost
  gather_facts: false
  roles:
    - role: vmware_esxi_stig_hardening
      vars:
        harden_scope: single
        esxi_hostname: "esxi01.example.local"
        esxi_username: "root"
        esxi_password: "{{ vault_esxi_root_password }}"
        ntp_servers: ["time1.example.local","time2.example.local"]
        syslog_remote: "udp://10.0.0.5:514"
```

**Harden all hosts in a cluster**
```yaml
- hosts: localhost
  gather_facts: false
  roles:
    - role: vmware_esxi_stig_hardening
      vars:
        harden_scope: cluster
        vcenter_hostname: "vcenter.example.local"
        vcenter_username: "{{ lookup('env','VCENTER_USERNAME') }}"
        vcenter_password: "{{ lookup('env','VCENTER_PASSWORD') }}"
        vcenter_validate_certs: false
        vcenter_datacenter: "DC1"
        vcenter_cluster: "Compute-Cluster"
        ntp_servers: ["time1.example.local","time2.example.local"]
        syslog_remote: "udp://10.0.0.5:514"
        lockdown_mode: "strict"
```

## Notes
- Some organizations require additional keys (FIPS, TLS ciphers, audit) not directly exposed by VMware modules; extend `advanced_config` accordingly.
- This role **does not install ESXi**; it configures existing hosts.
- Review in a lab before applying to production.
