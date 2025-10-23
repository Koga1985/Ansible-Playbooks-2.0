
# ans_ctrl_inventory_model

Build orgs/teams, inventories & sources, credentials, projects, EE registries/images.

## Variables
See `defaults/main.yml` — pass object lists to create/update. Role is **idempotent** and **artifacts-first**.

## Example
```yaml
- hosts: localhost
  roles:
    - role: ans_ctrl_inventory_model
      vars:
        apply_changes: true
        organizations: [{ name: Platform }]
        teams: [{ name: NetOps, organization: Platform }]
        inventories: [{ name: Prod, organization: Platform }]
        projects: [{ name: Playbooks, organization: Platform, scm_url: "https://git/repo.git" }]
        registries: [{ name: Harbor, host: harbor.local, username: robot, password: vault_robot }]
        execution_environments: [{ name: ee-netops, image: harbor.local/ee/netops:latest, registry_credential: Harbor }]
```
