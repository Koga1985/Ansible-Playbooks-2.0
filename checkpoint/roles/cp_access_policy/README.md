# Role: cp_access_policy

Build and operate the Check Point Access Policy lifecycle: sections, rules, tagging, safe removals, publish/install, and reports.

## Requirements
- Connection: `httpapi` to the Management Server
- Collection: `check_point.mgmt`

## Defaults
```yaml
cp_layer: "Network"
policy_package: "Standard"
install_targets: []
artifacts_dir: "/tmp/checkpoint-artifacts"
publish_changes: false
parallel_batches: 1
managed_tag: "ansible-managed"
hit_threshold: 0
```

## Typical Vars
```yaml
cp_access_sections:
  - { name: "00-Global", position: "top" }
  - { name: "10-Prod-Web", position: "below 00-Global" }

cp_access_rules:
  - name: "Allow Web to App"
    position: "below 10-Prod-Web"
    source: ["web-servers"]
    destination: ["app-servers"]
    service: ["http","https"]
    action: "Accept"
    track: "Log"
    enabled: true
    tags: ["ansible-managed","owner:netops","app:payments"]
```

## Usage
```yaml
- hosts: checkpoint_mgmt
  gather_facts: false
  roles:
    - role: cp_access_policy
      vars:
        cp_access_sections: "{{ lookup('file', 'vars/policy_sections.yml') | from_yaml }}"
        cp_access_rules: "{{ lookup('file', 'vars/policy_rules.yml') | from_yaml }}"
        install_targets: ["gw1","gw2"]
```

### Selective runs with tags
- `--tags sections` → scaffold sections
- `--tags rules` → apply rules
- `--tags tags` → add rule tags
- `--tags remove` → controlled deletes (use `cp_allow_rule_delete: true` and consider `dry_run: true`)
- `--tags publish` → publish session
- `--tags install` → install policy
- `--tags report,hitcount` → zero/low-hit report
- `--tags report,shadowing` → shadowing analysis (heuristic)
```

## Notes
- Use `managed_tag` in your desired rules to safely replace only what you own.
- Reports write CSVs under `artifacts_dir`; stash them as CI artifacts or attach to change tickets.
