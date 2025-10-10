# host_profiles_enforce (role)

Capture, attach, test, and remediate **VMware Host Profiles**. Supports optional
capture from a reference host, cluster/host scope, and maintenance mode handling.

## Requirements
- Ansible >= 2.15
- Collection: `community.vmware` (for maintenance mode and cluster host info)
- Controller/jump host with **PowerShell Core (pwsh)** and **VMware.PowerCLI**

## Variables (see `defaults/main.yml`)
```yaml
vcenter_hostname: "vcenter.example.local"
vcenter_username: "{{ lookup('env','VCENTER_USERNAME') }}"
vcenter_password: "{{ lookup('env','VCENTER_PASSWORD') }}"
vcenter_validate_certs: false

profile_name: "ESXi-Std"
capture_from_reference: true
reference_host: "esxi01.lab.local"
replace_existing_profile: true

attach_to_cluster: "Compute-Cluster"
# OR:
# attach_to_hosts: ["esxi01.lab.local","esxi02.lab.local"]

remediate: true
ensure_maintenance_mode: true
evacuate_vms: true
mm_timeout: 3600
```

## Example Play
```yaml
- hosts: localhost
  gather_facts: false
  roles:
    - role: host_profiles_enforce
      vars:
        vcenter_hostname: "vcenter.example.local"
        vcenter_username: "{{ lookup('env','VCENTER_USERNAME') }}"
        vcenter_password: "{{ lookup('env','VCENTER_PASSWORD') }}"
        vcenter_validate_certs: false

        profile_name: "ESXi-Std"
        capture_from_reference: true
        reference_host: "esxi01.lab.local"
        replace_existing_profile: true

        attach_to_cluster: "Compute-Cluster"
        ensure_maintenance_mode: true
        evacuate_vms: true
        mm_timeout: 3600
        remediate: true
```
