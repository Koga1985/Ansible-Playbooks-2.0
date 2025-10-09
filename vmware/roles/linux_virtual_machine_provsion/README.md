# vmware_linux_provision (role)

Clone a Linux VM from a vSphere template, attach it to the requested portgroup,
and join it to Active Directory using **realmd/SSSD** (or optionally stop after VMware customization).

## Requirements
- Ansible >= 2.15
- Collections: community.vmware, ansible.posix, community.general
- Control node Python: `pyvmomi` for VMware modules
- Linux template prepared with:
  - VMware tools / open-vm-tools
  - SSH enabled for initial configuration (for `join_method: realmd`)

## Variables (see `defaults/main.yml` for full list)
- vCenter: `vcenter_hostname`, `vcenter_username`, `vcenter_password`, `vcenter_datacenter`, `vcenter_cluster`, `vcenter_folder`, `vcenter_datastore`
- Template + VM: `vm_template`, `vm_name`, `vm_cpu`, `vm_memory_mb`, `vm_disk_gb`, `vm_networks`
- Join method: `join_method` = `realmd` (default) or `vmware`
- AD: `ad_domain`, `ad_domain_user`, `ad_domain_password`, `ad_domain_ou`, `ad_allow_groups`
- SSH to guest: `ssh_user`, `ssh_password` or `ssh_private_key_file`, `ssh_port`

## Example Play
```yaml
- hosts: localhost
  gather_facts: false
  roles:
    - role: vmware_linux_provision
      vars:
        vcenter_hostname: "vcenter.example.local"
        vcenter_username: "{{ lookup('env','VCENTER_USERNAME') }}"
        vcenter_password: "{{ lookup('env','VCENTER_PASSWORD') }}"
        vcenter_validate_certs: false
        vcenter_datacenter: "DC1"
        vcenter_cluster: "Cluster01"
        vcenter_folder: "/DC1/vm/Linux"
        vcenter_datastore: "vsanDatastore"

        vm_template: "RHEL9-Base"
        vm_name: "rhel9-ans-001"
        vm_networks:
          - name: "Prod-VLAN20"
            type: dhcp
            device_type: vmxnet3

        join_method: "realmd"

        ad_domain: "example.local"
        ad_domain_user: "EXAMPLE\\joinuser"
        ad_domain_password: "{{ vault_ad_join_password }}"
        ad_allow_groups:
          - "Domain Admins"
          - "Linux Admins"

        ssh_user: "cloud-user"
        ssh_private_key_file: "~/.ssh/id_rsa"
```

## Notes
- The role uses `community.vmware.vmware_guest` to clone from template and attach networks.
- AD join uses `realm join` with SSSD, following Red Hat guidance for realmd/SSSD. Ensure DNS and time sync are correct.
- For static IPs, set the first entry in `vm_networks` to `type: static` and provide `ip`, `netmask`, `gateway`, and optional `dns_servers`.
