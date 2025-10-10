# infoblox_grid_bootstrap

Bootstrap your **Infoblox Grid**:
- Ensure **members** (standalone or **HA pair** with VIP) exist
- Set Grid-wide **NTP**, **SNMP** (RO community/contact/location), and **SMTP** (relay/from)
- Optional **admin password rotation**
- **Join tokens** per member (best-effort; writes helper files)
- Export **license summary** + member inventory as JSON

> Uses WAPI directly to be version-agnostic. Some fields differ by NIOS release; the role is best-effort and logs warnings where an attribute is not accepted.

## Variables
```yaml
nios_host: "nios.example.local"
nios_username: "{{ lookup('env','NIOS_USER') }}"
nios_password: "{{ lookup('env','NIOS_PASS') }}"
nios_wapi_version: "v2.12"

members:
  - name: "nios-mem1"
    lan1_ipv4: "10.10.1.20"
    comment: "Core DNS/DHCP in AZ1"
  - name: "nios-ha1"
    ha:
      enabled: true
      node1_lan1: "10.10.2.21"
      node2_lan1: "10.10.2.22"
      vip: "10.10.2.20"
    join_token: true

ntp_servers: ["0.pool.ntp.org","1.pool.ntp.org"]
snmp: { ro_communities: ["public"], sys_contact: "noc@example.com", sys_location: "ATL-DC1" }
smtp: { relay: "smtp.example.com", sender: "infoblox@example.com" }
admin_password_rotate: "{{ vault_new_admin_pwd }}"

report_path: "/tmp/infoblox-grid-bootstrap-report.json"
token_dir: "/tmp/infoblox-grid-tokens"
```

## Example
```yaml
- hosts: localhost
  connection: local
  gather_facts: false
  roles:
    - role: infoblox_grid_bootstrap
      vars:
        nios_host: "nios.example.local"
        nios_username: "{{ lookup('env','NIOS_USER') }}"
        nios_password: "{{ lookup('env','NIOS_PASS') }}"
        ntp_servers: ["0.pool.ntp.org","1.pool.ntp.org"]
        members:
          - { name: "nios-mem1", lan1_ipv4: "10.10.1.20" }
          - name: "nios-ha1"
            ha: { enabled: true, node1_lan1: "10.10.2.21", node2_lan1: "10.10.2.22", vip: "10.10.2.20" }
            join_token: true
```
## Notes
- Joining an appliance to the Grid typically requires **CLI steps** on the member; when `join_token: true`, the role writes a helper file with the token and CLI sequence.
- Some organizations prefer managing members with `nios_member` or discovery; you can swap the WAPI calls for modules as needed.
