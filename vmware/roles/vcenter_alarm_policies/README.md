# vcenter_alarms_policies (role)

Standardize **vCenter Alarms** across your estate. Two ways to define thresholds:
1) Use a module-supported **preset** for common conditions (simple).
2) Provide full **raw XML** when you need exact metrics/triggers.

Actions can include **email** and **status** (and other module-supported actions).
For webhooks, point email to a ticketing gateway or use an external automation
(e.g., vRO/PowerCLI) subscribed to the alarm.

## Requirements
- Ansible >= 2.15
- Collection: `community.vmware`
- vCenter SMTP must be configured for email actions to work.

## Variables (see `defaults/main.yml`)
```yaml
alarms:
  - name: "Datastore usage high"
    target_type: "Datastore"
    expression: { preset: "datastore_usage_high" }
    actions:
      - { type: email, to: "ops@example.local" }
      - { type: status }
    enabled: true

  - name: "VM CPU Ready high"
    target_type: "VirtualMachine"
    alarm_expression_xml: |
      <alarmSpec>...</alarmSpec>
    actions:
      - { type: email, to: "ops@example.local" }
    enabled: true

remove_alarms_not_listed: false   # when true, delete alarms not declared above
```

## Example Play
```yaml
- hosts: localhost
  gather_facts: false
  roles:
    - role: vcenter_alarms_policies
      vars:
        vcenter_hostname: "vcenter.example.local"
        vcenter_username: "{{ lookup('env','VCENTER_USERNAME') }}"
        vcenter_password: "{{ lookup('env','VCENTER_PASSWORD') }}"
        vcenter_validate_certs: false

        alarms:
          - name: "Datastore usage high"
            target_type: "Datastore"
            expression: { preset: "datastore_usage_high" }
            actions:
              - { type: email, to: "ops@example.local" }
              - { type: status }

          - name: "Host connection and power state"
            target_type: "HostSystem"
            expression: { preset: "host_connection_and_power_state" }
            actions:
              - { type: email, to: "noc@example.local" }

        remove_alarms_not_listed: false
```
