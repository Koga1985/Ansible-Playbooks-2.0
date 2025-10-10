# infoblox_grid_upgrade

Stage and orchestrate **NIOS** upgrades/rollbacks in **waves**, with prechecks and optional image upload & activation via WAPI.

## What it does
- **Prechecks**: gather Grid/member versions; optional ping of member LAN/VIP.
- **Stage image**: download from `image_url` or use `image_file`, optional SHA256 verify.
- **Upload** (best-effort): POST to `grid?_function=upload_file` (varies by NIOS).
- **Plan waves**: per-wave JSON plan files (members, services to drain, notes).
- **Activate** (best-effort): POST to `grid?_function=activate_software` with optional reboot.
- **Reports**: `report_path` (JSON) plus `plan_dir` with precheck/plan/upload/activate artifacts.
- **Dry run**: default `dry_run: true` builds the plan and prechecks without calling upgrade endpoints.

> Service drain tasks are placeholders by default—wire them to your DNS/DHCP policies or explicit WAPI calls for your NIOS version.

## Variables (see `defaults/main.yml`)
```yaml
nios_host: "nios.example.local"
nios_username: "{{ lookup('env','NIOS_USER') }}"
nios_password: "{{ lookup('env','NIOS_PASS') }}"
nios_wapi_version: "v2.12"

image_url: "https://repo.example.com/nios/nios-9.0.0-1.bin"
target_version: "9.0.0-1"
checksum_sha256: ""

dry_run: true
activate_now: false
reboot_after_activate: false

waves:
  - name: "wave1-edge"
    members: ["nios-edge-01","nios-edge-02"]
    drain_services: ["dns","dhcp"]
    pre_window_note: "After-hours 22:00-00:00 ET"
  - name: "wave2-core"
    members: ["nios-core-01","nios-core-02"]
    drain_services: ["dns"]
```

## Example play
```yaml
- hosts: localhost
  connection: local
  gather_facts: false
  roles:
    - role: infoblox_grid_upgrade
      vars:
        nios_host: "nios.example.local"
        nios_username: "{{ lookup('env','NIOS_USER') }}"
        nios_password: "{{ lookup('env','NIOS_PASS') }}"
        image_url: "https://repo.example.com/nios/nios-9.0.0-1.bin"
        target_version: "9.0.0-1"
        dry_run: true
        waves:
          - { name: "edge", members: ["nios-edge-01","nios-edge-02"], drain_services: ["dns","dhcp"] }
          - { name: "core", members: ["nios-core-01","nios-core-02"], drain_services: ["dns"] }
```
## Notes
- API endpoints for upload/activate can differ across NIOS versions; the role captures responses and won’t fail hard if an endpoint isn’t present—review artifacts in `plan_dir`.
- For rollbacks, re-run with an older image and the same waves; many teams also schedule a **Grid backup** before starting (see your `infoblox_grid_backup` role).
