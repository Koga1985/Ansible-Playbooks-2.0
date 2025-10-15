# panorama_fleet

Production role to onboard/manage firewalls via **Panorama**:

- Create/maintain **Templates** and **Template Stacks**
- Create/maintain **Device Groups**
- Attach devices (by **serial**) to stacks and groups
- Commit to Panorama and **push** to device-groups

> Run this role **against Panorama** using the `paloaltonetworks.panos` httpapi connection.

## Requirements
- Collection: `paloaltonetworks.panos`
- Inventory pointing at Panorama (httpapi, SSL, creds/api key)
- Firewalls already configured to point to Panorama (and Device Registration Auth Key if your version requires it).

## Defaults (see `defaults/main.yml`)
```yaml
panorama_fleet_devices: []
panorama_device_groups: []
panorama_templates: []
panorama_template_stacks: []

commit_description: "Automated Panorama commit via Ansible"
commit_after_changes: true
commit_push_device_groups: []
commit_serial: true
include_template: false
include_device_and_network: false
```

## Example playbook
```yaml
- hosts: panorama
  gather_facts: false
  roles:
    - role: panorama_fleet
      vars:
        panorama_fleet_devices:
          - serial: "0123456789"
            device_group: "Prod"
            template_stack: "Prod-Stack"
        panorama_device_groups:
          - name: "Prod"
            description: "Production"
            devices: ["0123456789"]
        panorama_templates:
          - name: "Base"
          - name: "Datacenter"
        panorama_template_stacks:
          - name: "Prod-Stack"
            templates: ["Base","Datacenter"]
            devices: ["0123456789"]
        commit_after_changes: true
        commit_push_device_groups: ["Prod"]
```

## Notes
- Ensure Panorama can reach the devices (mgmt plane) and required ports are open.
- If your collection version doesn’t support passing `devices` in `panos_device_group`, device attachment is still handled via **template stacks**.
