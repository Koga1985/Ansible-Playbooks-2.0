# pa_vpn_ipsec

Production role to build and monitor **IPsec VPNs** on Palo Alto Networks firewalls or via Panorama templates:
- IKE and IPsec **crypto profiles**
- **IKE Gateways** (v1/v2) with PSK & IDs
- **Tunnel interfaces** with optional IPs, VR & Zone binding
- **IPsec tunnels** (auto-key) with monitor/ToS-copy
- Optional **static routes** for tunnel next-hops
- **Monitoring**: export IKE/IPsec SA status to CSV

> For Panorama-based deployments, pass `pa_use_panorama: true` and set `template` or `template_stack`.

## Requirements
- Collection: `paloaltonetworks.panos`
- Run with `httpapi` connection to firewall or Panorama.

## Quick Example
```yaml
- hosts: pan_firewalls
  gather_facts: false
  roles:
    - role: pa_vpn_ipsec
      vars:
        ike_crypto_profiles:
          - { name: ike-strong, dh_group: [group14], authentication: [sha256], encryption: [aes-256-cbc], lifetime_seconds: 28800 }
        ipsec_crypto_profiles:
          - { name: ipsec-strong, esp_auth: [sha256], esp_encryption: [aes-256-gcm], dh_group: [group14], lifetime_seconds: 3600 }
        ike_gateways:
          - { name: gw-to-branch1, version: ikev2, interface: ethernet1/1, peer_ip_value: 198.51.100.10, pre_shared_key: REDACTED, IKEv2_proposal: ike-strong }
        tunnel_interfaces:
          - { name: tunnel.10, vr: default, zone: vpn, ipv4_address: 10.250.0.1/30 }
        ipsec_tunnels:
          - { name: ipsec-to-branch1, tunnel_interface: tunnel.10, ak_ike_gateway: gw-to-branch1, ak_ipsec_crypto_profile: ipsec-strong, enable_tunnel_monitor: true, monitor_dest_ip: 10.250.0.2 }
        tunnel_static_routes:
          - { vr: default, destination: 10.20.0.0/16, next_hop: 10.250.0.2, interface: tunnel.10, metric: 10 }
```

## Notes
- Some collection versions expose proxy-id lists directly on `panos_ipsec_tunnel`. If yours does, add them there.
- Ensure reachability to peer public IPs and UDP 500/4500 open.
- Commit behavior controlled by `commit_after_changes`.
