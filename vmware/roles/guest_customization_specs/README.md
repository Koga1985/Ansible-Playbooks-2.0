# guest_customization_specs (role)

Create/maintain **vCenter Guest Customization Specs** for **Windows** and **Linux** via PowerCLI:
- Windows: Full/Org name, **time zone**, **change SID**, **product key**, **admin password**,
  **auto logon**, **domain join** (with OU), workgroup, **run-once** commands, **hostname scheme**.
- Linux: domain name, **time zone**, UTC hardware clock flag, optional **script text**.
- Both: **NIC mappings** per position, DHCP or static, **DNS servers** and **DNS domain**.

Outputs a JSON report at `report_path` listing created/updated specs and NIC counts.

> Hostname naming supports `useVMName`, `prefix:<TEXT>`, and `fixed:<NAME>`.

## Requirements
- Controller/jump host with **PowerShell Core (`pwsh`)** and **VMware PowerCLI**
- vCenter privileges to manage OSCustomizationSpec

## Variables (see `defaults/main.yml`)
```yaml
specs:
  - name: "win-std"
    type: "windows"
    description: "Windows Server base customization"
    overwrite: true
    windows:
      full_name: "Acme IT"
      org_name: "Acme Corp"
      time_zone: 035
      change_sid: true
      product_key: ""
      admin_password: "{{ vault_win_admin_pwd }}"
      auto_logon_count: 0
      domain_join:
        enabled: true
        domain: "corp.example.com"
        username: "CORP\svc_join"
        password: "{{ vault_join_pwd }}"
        oupath: "OU=Servers,DC=corp,DC=example,DC=com"
      workgroup: ""
      run_once: []
      hostname: "useVMName"             # useVMName | prefix:WEB | fixed:SERVER01
    nics:
      - index: 0
        type: "dhcp"
        dns_servers: ["10.10.10.10","10.10.10.11"]
        dns_domain: "corp.example.com"

  - name: "lin-std"
    type: "linux"
    description: "Linux base customization"
    overwrite: true
    linux:
      domain: "corp.example.com"
      time_zone: "UTC"
      hwclock_utc: true
      script_text: |
        echo 'hello' > /root/hello.txt
    nics:
      - index: 0
        type: "static"
        ip: "10.30.0.25"
        subnet: "255.255.255.0"
        gateway: ["10.30.0.1"]
        dns_servers: ["10.10.10.10","10.10.10.11"]
        dns_domain: "corp.example.com"
```

## Example Play
```yaml
- hosts: localhost
  gather_facts: false
  roles:
    - role: guest_customization_specs
      vars:
        vcenter_hostname: "vcsa.example.local"
        vcenter_username: "{{ lookup('env','VCENTER_USERNAME') }}"
        vcenter_password: "{{ lookup('env','VCENTER_PASSWORD') }}"
        vcenter_validate_certs: false

        specs:
          - name: "win-std"
            type: "windows"
            description: "Windows Server base"
            overwrite: true
            windows:
              full_name: "Acme IT"
              org_name: "Acme Corp"
              time_zone: 035
              change_sid: true
              admin_password: "{{ vault_win_admin_pwd }}"
              domain_join: { enabled: true, domain: "corp.example.com", username: "CORP\svc_join", password: "{{ vault_join_pwd }}", oupath: "OU=Servers,DC=corp,DC=example,DC=com" }
              hostname: "useVMName"
            nics:
              - { index: 0, type: "dhcp", dns_servers: ["10.10.10.10","10.10.10.11"], dns_domain: "corp.example.com" }

          - name: "lin-std"
            type: "linux"
            description: "Linux base"
            overwrite: true
            linux:
              domain: "corp.example.com"
              time_zone: "UTC"
              hwclock_utc: true
            nics:
              - { index: 0, type: "dhcp", dns_servers: ["10.10.10.10","10.10.10.11"], dns_domain: "corp.example.com" }
```
