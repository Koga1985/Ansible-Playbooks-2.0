# vmware_vm_stig_hardening (role)

Apply a practical, DISA-aligned **STIG/hardening baseline** to a **vSphere VM** using Ansible:
- Sets secure **advanced configuration (extraConfig)** keys (copy/paste, dnd, hotplug, VNC, disk shrink/wiper, etc.).
- Enforces **EFI + Secure Boot** (where supported).
- Removes risky devices (**CD-ROM**, **Floppy**) and disconnects serial/parallel.
- Adds an annotation banner for audit.

## Requirements
- Ansible >= 2.15
- Collection: `community.vmware`
- Control node Python: `pyvmomi`
- VM must already exist in the target Datacenter.

## Variables (see `defaults/main.yml`)
- vCenter connection: `vcenter_hostname`, `vcenter_username`, `vcenter_password`, `vcenter_validate_certs`, `vcenter_datacenter`
- Target: `vm_name`
- Switches: `stig_disable_copy_paste`, `stig_disable_dnd`, `stig_disable_vnc`, `stig_disable_device_hotplug`, `stig_disable_disk_shrink_wiper`, `stig_disable_gui_options`, etc.
- Hardware: `stig_require_efi_secure_boot`, `stig_remove_cdrom`, `stig_remove_floppy`, `stig_disconnect_parallel_serial`
- Optional banner: `stig_banner_annotation`

## Example Play
```yaml
- hosts: localhost
  gather_facts: false
  roles:
    - role: vmware_vm_stig_hardening
      vars:
        vcenter_hostname: "vcenter.example.local"
        vcenter_username: "{{ lookup('env','VCENTER_USERNAME') }}"
        vcenter_password: "{{ lookup('env','VCENTER_PASSWORD') }}"
        vcenter_validate_certs: false
        vcenter_datacenter: "DC1"
        vm_name: "win2019-ans-001"

        stig_require_efi_secure_boot: true
        stig_remove_cdrom: true
        stig_remove_floppy: true
        stig_disable_copy_paste: true
        stig_disable_dnd: true
        stig_disable_vnc: true
        stig_disable_device_hotplug: true
        stig_disable_disk_shrink_wiper: true
        stig_disable_cpu_hotadd: true
        stig_disable_mem_hotadd: true
```
