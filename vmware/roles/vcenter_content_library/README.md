# vcenter_content_library (role)

Create/maintain **local** and **subscribed** vCenter Content Libraries, publish
OVAs/OVFs/ISOs as items, and optionally sync subscribers.

## Requirements
- Ansible >= 2.15
- Collection: `community.vmware` (see `requirements.yml`)
- Uploads require controller access to the file path or URL.

## Variables (see `defaults/main.yml`)
```yaml
libraries:
  - name: "LocalLib"
    type: local
    datastore: "vsanDatastore"
    publication: { published: true, allow_subscribe: true }

  - name: "UpstreamLib"
    type: subscribed
    subscription_url: "https://publisher.example.com/lib.json"
    auth_type: none
    automatic_sync: true

library_items:
  - { library: "LocalLib", name: "ubuntu-22.04-base", file: "/files/ubuntu.ova", state: present }

sync_subscribed: true
```

## Example Play
```yaml
- hosts: localhost
  gather_facts: false
  roles:
    - role: vcenter_content_library
      vars:
        vcenter_hostname: "vcenter.example.local"
        vcenter_username: "{{ lookup('env','VCENTER_USERNAME') }}"
        vcenter_password: "{{ lookup('env','VCENTER_PASSWORD') }}"
        vcenter_validate_certs: false
        libraries:
          - { name: "LocalLib", type: local, datastore: "vsanDatastore", publication: { published: true, allow_subscribe: true } }
          - { name: "UpstreamLib", type: subscribed, subscription_url: "https://example/lib.json", automatic_sync: true }
        library_items:
          - { library: "LocalLib", name: "ubuntu-22.04-base", file: "/files/ubuntu.ova", state: present }
        sync_subscribed: true
```
