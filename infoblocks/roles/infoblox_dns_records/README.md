# infoblox_dns_records

Bulk-manage Infoblox DNS records with Ansible (via `infoblox.nios_modules`). Supports:
- **Host records** (A/AAAA + PTR, aliases)
- **A / AAAA / CNAME / MX / SRV / TXT / NS**
- Load records from **external YAML files**
- Optional global **TTL enforcement**

## Requirements
- Ansible collection: `infoblox.nios_modules`
- Control host Python: `infoblox-client` (used by collection)
- Credentials with WAPI rights

## Variables (see `defaults/main.yml`)
```yaml
nios_host: "nios.example.local"
nios_username: "{{ lookup('env','NIOS_USER') }}"
nios_password: "{{ lookup('env','NIOS_PASS') }}"
nios_validate_certs: false

dns_view: "default"
enforce_ttl: true
ttl_default: 3600

# Optionally load more records from YAML files
record_files:
  - "group_vars/dns/prod_records.yml"
  - "files/dns/additional.yml"

hosts:
  - name: "web01.corp.example.com"
    ipv4: [{ address: "10.10.10.15", create_ptr: true }]
    aliases: ["web.corp.example.com"]
    ttl: 300
    view: "Corp-View"
    state: present

a_records:
  - { name: "api.corp.example.com", address: "10.10.20.50", ttl: 300, view: "Corp-View", state: present }

cname_records:
  - { name: "db.corp.example.com", canonical: "db01.corp.example.com", ttl: 300, view: "Corp-View" }
```
> Each record item accepts `state: present|absent` and optional `extattrs` where supported.

## Example Playbook
```yaml
- hosts: localhost
  connection: local
  gather_facts: false
  roles:
    - role: infoblox_dns_records
      vars:
        nios_host: "nios.example.local"
        nios_username: "{{ lookup('env','NIOS_USER') }}"
        nios_password: "{{ lookup('env','NIOS_PASS') }}"
        dns_view: "Corp-View"
        ttl_default: 600
        enforce_ttl: true

        record_files:
          - "inventory/dns/prod.yml"

        hosts:
          - name: "app01.corp.example.com"
            ipv4: [{ address: "10.30.0.21", create_ptr: true }]
            aliases: ["app.corp.example.com"]
            ttl: 600

        a_records:
          - { name: "api.corp.example.com", address: "10.30.0.50", ttl: 300 }

        cname_records:
          - { name: "db.corp.example.com", canonical: "db01.corp.example.com", ttl: 300 }
```
## Tips
- Prefer **Host records** for real machines; they manage forward + reverse in one object.
- Keep bulk data in versioned YAML, then list files in `record_files` for easy imports.
- For split-horizon, set `view` per record or change global `dns_view`.
