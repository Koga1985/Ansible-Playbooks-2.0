# infoblox_inventory_model

Create and standardize your **Infoblox inventory model**:
- **Extensible Attribute** (EA) definitions
- **Network (DHCP/IPAM) views**
- **DNS views**
- **VLANs**
- **IPv4/IPv6 network containers** (supernets)

## Requirements
- Ansible collection: `infoblox.nios_modules`
- Control host Python package: `infoblox-client`
- A WAPI user with rights to create/modify the above objects

## Variables (see `defaults/main.yml` for full examples)
```yaml
ea_definitions:
  - { name: "Environment", type: "list", list_values: ["prod","dev","stage"], flags: "IL", comment: "Workload env" }
  - { name: "AppTier", type: "list", list_values: ["web","app","db"], flags: "IL" }
  - { name: "Owner", type: "string", flags: "L" }

network_views:
  - { name: "Corp-NV", comment: "Corporate networks", extattrs: { Environment: { value: "prod" } }, state: present }

dns_views:
  - { name: "Corp-View", comment: "Split-horizon corporate DNS", state: present }

vlans:
  - { name: "VLAN10-Users", id: 10, parent: "default", description: "User access", extattrs: { Environment: { value: "prod" } } }

network_containers_v4:
  - { cidr: "10.0.0.0/8", view: "Corp-NV", comment: "All corp space", extattrs: { Environment: { value: "prod" } } }

network_containers_v6:
  - { cidr: "2001:db8::/32", view: "Corp-NV", comment: "Global v6" }
```

## Example Playbook
```yaml
- hosts: localhost
  connection: local
  gather_facts: false
  roles:
    - role: infoblox_inventory_model
      vars:
        nios_host: "nios.example.local"
        nios_username: "{{ lookup('env','NIOS_USER') }}"
        nios_password: "{{ lookup('env','NIOS_PASS') }}"
        nios_validate_certs: false

        ea_definitions:
          - { name: "Environment", type: "list", list_values: ["prod","dev","stage"], flags: "IL" }
          - { name: "AppTier", type: "list", list_values: ["web","app","db"], flags: "IL" }

        network_views:
          - { name: "Corp-NV", comment: "Corporate networks", extattrs: { Environment: { value: "prod" } } }

        dns_views:
          - { name: "Corp-View", comment: "Split-horizon corporate DNS" }

        vlans:
          - { name: "VLAN10-Users", id: 10, parent: "default", description: "User access" }

        network_containers_v4:
          - { cidr: "10.0.0.0/8", view: "Corp-NV", comment: "All corp space" }

        network_containers_v6:
          - { cidr: "2001:db8::/32", view: "Corp-NV", comment: "Global v6" }
```

## Notes
- `nios_network` with `container: true` creates **network containers** (supernets), not leaf networks.
- Use **EA definitions** first so objects can be consistently tagged as they are created.
- For split-horizon, keep **Network Views** (IPAM/DHCP) separate from **DNS Views** (query resolution).
