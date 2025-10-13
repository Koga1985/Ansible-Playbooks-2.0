# infoblox_rpz_policies

Manage **Infoblox Response Policy Zones (RPZ)** with Ansible:

- Create/ensure **RPZ zones** (primary, secondary, or feed-driven)
- Set **enforcement order** within a **DNS view**
- Optionally **sync** external feeds
- Export a summary JSON artifact for auditing

> The role uses WAPI endpoints that can vary by **NIOS** version; it records responses and won't hard-fail when a field isn't present.

## Requirements
- Ansible collection: `infoblox.nios_modules`
- WAPI user with rights to create/update RPZ zones and edit DNS views

## Variables (defaults)
```yaml
nios_host: ""
nios_username: ""
nios_password: ""
nios_validate_certs: false
nios_wapi_version: "v2.12"

rpz_dns_view: "default"
rpz_zones: []
rpz_enforcement_order: []
rpz_sync: false
artifact_dir: "/tmp/infoblox-rpz"
```

## RPZ zone schema examples
```yaml
rpz_zones:
  # Local primary RPZ (you manage rules in this zone)
  - name: "corp-threats.rpz"
    comment: "Internal RPZ for incident blocks"
    primary_type: "primary"            # "none" | "primary" | "secondary"
    grid_primary: "Grid Primary"       # adjust to your environment
    extattrs:
      Environment: { value: "prod" }

  # Secondary RPZ from an upstream provider
  - name: "vendor-feed1.rpz"
    primary_type: "secondary"
    grid_secondaries:
      - "Member-1"
      - "Member-2"

  # Feed-driven (external) RPZ
  - name: "open-intel.rpz"
    primary_type: "primary"
    external_feeds:
      - name: "OpenIntelDaily"
        url: "https://feeds.example.com/openintel.rpz"
        username: "feeduser"
        password: "feedpass"
        schedule: "daily"
    comment: "External community feed"
```

## Enforcement order
Provide an ordered list of RPZ **names**; the role resolves them to WAPI refs and attempts to set on the DNS view:
```yaml
rpz_enforcement_order:
  - "corp-threats.rpz"
  - "vendor-feed1.rpz"
  - "open-intel.rpz"
```

## Example Playbook
```yaml
- hosts: localhost
  connection: local
  gather_facts: false
  roles:
    - role: infoblox_rpz_policies
      vars:
        nios_host: "nios.example.local"
        nios_username: "{{ lookup('env','NIOS_USER') }}"
        nios_password: "{{ lookup('env','NIOS_PASS') }}"
        nios_validate_certs: false
        rpz_dns_view: "default"
        rpz_zones:
          - name: "corp-threats.rpz"
            comment: "Internal RPZ for incident blocks"
            primary_type: "primary"
            grid_primary: "Grid Primary"
          - name: "vendor-feed1.rpz"
            primary_type: "secondary"
            grid_secondaries: ["Member-1","Member-2"]
          - name: "open-intel.rpz"
            primary_type: "primary"
            external_feeds:
              - { name: "OpenIntelDaily", url: "https://feeds.example.com/openintel.rpz", username: "feeduser", password: "feedpass", schedule: "daily" }
        rpz_enforcement_order: ["corp-threats.rpz","vendor-feed1.rpz","open-intel.rpz"]
        rpz_sync: true
        artifact_dir: "/tmp/infoblox-rpz"
```

## Outputs
- `rpz_policies_summary.json` — existing RPZs, DNS view info, order-setting responses, sync results
