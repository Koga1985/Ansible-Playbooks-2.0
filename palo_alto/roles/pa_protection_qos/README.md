# pa_protection_qos

Production role for Palo Alto Networks to manage:
- **Zone Protection Profiles (ZPP)** and zone bindings
- **DoS protection** (profiles + policy rules, aggregate/classified)
- **QoS** (profiles + policy rules)

> Works for **firewalls** and **Panorama** (`pa_use_panorama: true` + `device_group`).

## Requirements
- Collection: `paloaltonetworks.panos`
- `httpapi` connection to firewall or Panorama

## Variables (see `defaults/main.yml`)
- `zpp_profiles`: list of Zone Protection Profiles
- `zpp_zone_bindings`: list of zone → ZPP mappings
- `dos_profiles`: DoS protection profiles (aggregate/classified)
- `dos_rules`: DoS policy rules referencing profiles
- `qos_profiles`: QoS class bandwidth profiles
- `qos_policies`: QoS policies (match on zones/apps/services, set class & optional interface)

## Example (firewall)
```yaml
- hosts: pan_firewalls
  gather_facts: false
  roles:
    - role: pa_protection_qos
      vars:
        zpp_profiles:
          - name: "ZPP-Internet"
            flood: { syn: { enable: true, red: true, alarm_rate: 1000, activate_rate: 2000, maximal_rate: 4000 } }
            reconnaissance: { host-sweep: true, tcp-port-scan: true, udp-port-scan: true }
        zpp_zone_bindings:
          - { zone: "untrust", profile: "ZPP-Internet" }
        dos_profiles:
          - { name: "DoS-Agg-HTTPS", type: "aggregate", max_rate: 50000, block_duration: 60 }
        dos_rules:
          - name: "DoS-from-Internet"
            from_zones: ["untrust"]
            to_zones: ["dmz"]
            source: ["any"]
            destination: ["any"]
            services: ["service-https"]
            type: "aggregate"
            action: "protect"
            profile: "DoS-Agg-HTTPS"
        qos_profiles:
          - name: "QoS-Default"
            class_bandwidth: { class1: 10000, class2: 5000, class3: 1000 }
        qos_policies:
          - name: "QoS-Web"
            from_zones: ["trust"]
            to_zones: ["untrust"]
            applications: ["web-browsing","ssl"]
            services: ["application-default"]
            class: "class2"
            interface: "ethernet1/1"
```

## Commit
Set `commit_after_changes: true` to auto-commit (default). Customize `commit_description` as needed.
