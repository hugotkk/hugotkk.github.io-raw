---
title: "Openstack - Cinder"
date: 2023-12-19
tags:
- openstack
- cinder
---

## LVM2 Backend

To configure the LVM2 backend in cinder.conf, follow these steps:

- Create an LVM volume group named "cinder-volumes."

- Define a configuration section like the one shown below in cinder.conf:

```
[lvm-1]
volume_group = cinder-volumes
volume_driver = cinder.volume.drivers.lvm.LVMVolumeDriver
volume_backend_name = lvm-1
target_helper = tgtadm
target_protocol = iscsi
```

- Restart the Cinder service.

- Set the `volume_backend_name` property of the desired volume type to match the `volume_backend_name` defined in the cinder conf:

```bash
openstack volume type set --property volume_backend_name=lvm-1 <VOLUME_TYPE_NAME>
```

If you need additional LVM backends, you can create more volume groups based on your requirements.

For more details, you can refer to the documentation at: [https://docs.openstack.org/kolla-ansible/latest/reference/storage/cinder-guide.html](https://docs.openstack.org/kolla-ansible/latest/reference/storage/cinder-guide.html)

## LUKS Encryption

Enable Barbican in Kolla Configuration:

vim /etc/kolla/global.yml

```yaml
enable_barbican: "yes"
barbican_crypto_plugin: "simple_crypto"
barbican_library_path: "/usr/lib/libCryptoki2_64.so"
```

Configure Cinder with Barbican:

vim /etc/kolla/config/cinder.conf

```
[key_manager]
backend = barbican
```

Restart cinder-api, cinder-volume and cinder-backup.


vim /etc/kolla.config/nova.conf

```
[key_manager]
backend = barbican
```

Deploy the Configuration:

Run the Kolla-Ansible deploy command:

```bash
kolla-ansible -i ./all-in-one deploy
```

Use the OpenStack CLI to create a LUKS encrypted volume type:

```bash
openstack volume type create --encryption-provider luks --encryption-cipher aes-xts-plain64 --encryption-key-size 256 --encryption-control-location front-end LUKS
```

Now, create a volume using the LUKS type:

```bash
openstack volume create --size 1 --type LUKS enc_vol
```

<https://docs.openstack.org/cinder/latest/configuration/block-storage/volume-encryption.html#>
