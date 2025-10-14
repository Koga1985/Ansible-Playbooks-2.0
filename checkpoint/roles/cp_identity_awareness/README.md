# Role: cp_identity_awareness

Enable Identity Awareness, configure identity sources, build Access Roles, and create identity-based rules.

## Requirements
- Connection: `httpapi` to the Management Server
- Collection: `check_point.mgmt`

## Defaults
```yaml
cp_layer: "Network"
publish_changes: false
artifacts_dir: "/tmp/checkpoint-artifacts"

ia_gateways: []
ia_identity_sources: []
ia_access_roles: []
ia_rules: []
```

## Example vars
```yaml
ia_gateways:
  - { name: "dc-gw1", identity_aware: true }

ia_identity_sources:
  - name: "corp-ad"
    ad_domain: "corp.local"
    servers: ["dc1.corp.local","dc2.corp.local"]
    user: "CORP\\svc_ia"
    password: "{{ vault_ia_pwd }}"
    use_ssl: true

ia_access_roles:
  - name: "Role-IT-Admins"
    users: ["CN=IT Admins,OU=Groups,DC=corp,DC=local"]
    networks: ["corp-lan"]

ia_rules:
  - name: "IT Admins to Mgmt"
    source_roles: ["Role-IT-Admins"]
    destination: ["mgmt-servers"]
    service: ["ssh","https"]
    action: "Accept"
    tags: ["identity","owner:secops"]
```

## Usage
```yaml
- hosts: checkpoint_mgmt
  gather_facts: false
  roles:
    - role: cp_identity_awareness
      vars:
        publish_changes: true
        ia_gateways:
          - { name: "dc-gw1", identity_aware: true }
        ia_identity_sources: "{{ lookup('file', 'vars/ia_identity_sources.yml') | from_yaml }}"
        ia_access_roles: "{{ lookup('file', 'vars/ia_access_roles.yml') | from_yaml }}"
        ia_rules: "{{ lookup('file', 'vars/ia_rules.yml') | from_yaml }}"
```
