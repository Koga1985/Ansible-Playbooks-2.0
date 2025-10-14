# Role: cp_inventory_model

Build and maintain Check Point network inventory (hosts, networks/ranges, groups) from a source-of-truth (YAML, CMDB export, etc.).

## Requirements
- Connection: `httpapi` to the Management Server
- Collection: `check_point.mgmt`

## Defaults
```yaml
publish_changes: false
artifacts_dir: "/tmp/checkpoint-artifacts"
cp_hosts: []
cp_networks: []
cp_address_ranges: []
cp_groups: []
replace_members: true
```

## Example vars
```yaml
cp_hosts:
  - { name: app01, ip: 10.10.1.10, color: blue, comment: "App server" }

cp_networks:
  - { name: prod-net, subnet: 10.10.0.0, mask: 16, color: red, comment: "Prod /16" }

cp_address_ranges:
  - { name: dr-range, start: 10.20.50.10, end: 10.20.50.200, comment: "DR pool" }

cp_groups:
  - { name: web-servers, members: [app01, 10.10.1.11, prod-net], color: green }
```

## Usage
```yaml
- hosts: checkpoint_mgmt
  gather_facts: false
  roles:
    - role: cp_inventory_model
      vars:
        publish_changes: true
        cp_hosts: "{{ lookup('file', 'vars/hosts.yml') | from_yaml }}"
        cp_networks: "{{ lookup('file', 'vars/networks.yml') | from_yaml }}"
        cp_groups: "{{ lookup('file', 'vars/groups.yml') | from_yaml }}"
```

## Tags
- `inventory`, `hosts`, `networks`, `groups`, `publish`
