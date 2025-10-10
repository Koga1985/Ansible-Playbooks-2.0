# infoblox_dnssec

Enable/configure **DNSSEC** on Infoblox zones and export **DS** records for parent delegation.

What it does
- Optionally **creates** zones (via `nios_zone`) if `allow_create: true`
- Toggles DNSSEC and **NSEC/NSEC3** (+ opt-out)
- Sets **KSK/ZSK** algorithms and sizes (RSA/ECDSA)
- Stores rollover preferences as **Extensible Attributes** (KSK/ZSK days)
- Re-reads zone and writes **DS records** to files + a JSON aggregate

> Uses WAPI on the `zone_auth` object for DNSSEC attributes; this keeps behavior consistent across collection versions.

## Requirements
- Ansible collection: `infoblox.nios_modules`
- Control node: `infoblox-client` (pulled by the collection)
- NIOS WAPI user with zone-modify permissions

## Variables (see `defaults/main.yml`)
```yaml
nios_host: "nios.example.local"
nios_username: "{{ lookup('env','NIOS_USER') }}"
nios_password: "{{ lookup('env','NIOS_PASS') }}"
nios_wapi_version: "v2.12"

zones_dnssec:
  - fqdn: "corp.example.com"
    view: "default"
    allow_create: false
    enable: true
    nsec3: true
    nsec3_optout: false
    ksk_algorithm: "RSASHA256"
    ksk_size: 2048
    zsk_algorithm: "RSASHA256"
    zsk_size: 1024
    rollover: { ksk_days: 365, zsk_days: 30 }

ds_out_dir: "/tmp/infoblox-dnssec-ds"
ds_json_path: "/tmp/infoblox-dnssec-ds/ds_records.json"
```

## Example Play
```yaml
- hosts: localhost
  connection: local
  gather_facts: false
  roles:
    - role: infoblox_dnssec
      vars:
        nios_host: "nios.example.local"
        nios_username: "{{ lookup('env','NIOS_USER') }}"
        nios_password: "{{ lookup('env','NIOS_PASS') }}"
        zones_dnssec:
          - { fqdn: "corp.example.com", view: "default", enable: true, nsec3: true, ksk_algorithm: "RSASHA256", ksk_size: 2048, zsk_algorithm: "RSASHA256", zsk_size: 1024 }
          - { fqdn: "partner.example.net", view: "default", enable: true, nsec3: false, ksk_algorithm: "ECDSAP256SHA256" }
```
## Notes
- Infoblox handles key generation & periodic rollovers per grid policy. This role exposes preferred values and records them in **EAs** for audit.
- DS export writes one `*.ds.txt` per zone and also appends each run to `ds_records.json` (newline-delimited JSON). You can feed this to your registrar workflow.
