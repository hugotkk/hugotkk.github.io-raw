---
title: "Openstack with kolla ansible"
date: 2023-12-19
tags:
- openstack
- ansible
- kolla-ansible
---

## install kolla-ansible

all we need is this doc: [Kolla-Ansible Quickstart](https://docs.openstack.org/kolla-ansible/latest/user/quickstart.html)

## Additional Config

### Swift Object Storage

To enable Swift with storage policies and additional disks for the VMs, follow the guide below and create the necessary ring files:

- [Swift Guide](https://docs.openstack.org/kolla-ansible/latest/reference/storage/swift-guide.html)

### Cinder Block Storage

For Cinder, label your disks accordingly:

- [Cinder Guide](https://docs.openstack.org/kolla-ansible/latest/reference/storage/cinder-guide.html)

### Additional Features

- Volume backup
- LUKS encryption with Barbican in Nova and Cinder (Note: It's tricky because Barbican is not configured on Swift and Cinder by default).

To override the default configurations, place your custom config files into `/etc/kolla/config`. These will be merged with the existing ones, hence the `config` folder in this repository.

### TLS and Troubleshooting

I attempted to enable TLS; however, it caused issues with the volume backup service's connectivity to the database.

This may be due to enabling TLS at pos- setup. Therefore, it is currently disabled. It's advised to start with a fresh installation if you wish to enable TLS.
