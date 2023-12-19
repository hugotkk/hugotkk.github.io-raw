---
title: "Openstack - Image, Volume and Snapshot"
date: 2023-12-19
tags:
- openstack
- linux
---

## Conversion between Image, Volume and Snapshot

Each of these can serve as a source for creating a new VM.

| From/To | Image | Volume | Snapshot |
|---|---|---|---|
| Image | Y |   |   |
| Volume |   |  |  Y |
| Snapshot |   |  Y |  |

## Public image vs shared image

|                   | Public Images                           | Shared Images                                 |
|-------------------|-----------------------------------------|-----------------------------------------------|
| **Visibility**    | Accessible to all projects.             | Visible only to the owning project by default.|
| **Use Case**      | For common base images or standard configurations. | For controlled access to specific images.      |
| **Access Control**| All users can list, view, and use these images. | Admins can share with certain projects; unlisted for others. |


Add / remove project to shared image
```bash
openstack image set <image> --shared
openstack image add project <image> <project>
openstack image remove project <image> <project>
```

## Backup and Restore VM

Only images can be imported/exported to/from the OpenStack environment.

Volumes and snapshots remain within the OpenStack system.

To export an image, the `openstack server image save` command is used.

### Take Snapshot

```bash
openstack server image create
```

### Restore from Image

```bash
openstack server create --image <image>
```

### Download image

```bash
openstack image save <image> --file <output_file>
```

## Volume without cinder

Cinder serves as OpenStack's volume service (akin to AWS EBS).

It's worth noting that OpenStack can still be utilized even in the absence of Cinder (similar to AWS's ephemeral storage).

When Cinder is not present, two key distinctions become apparent:

1. The "Volumes" tab will not appear in the Horizon dashboard.
2. The "--boot-from-volume" option becomes inaccessible when using the "openstack server create" command.

## 0 byte image & snapshot

If you select "Yes" for "Create New Volume" (--boot-from-volume in cli) during VM creation, the image won't be copied into the disk.

Instead, it will only store metadata to establish a linkage to the image.

Therefore, if you intend to export / snapshot a Volume-booted VM, you will end up with a 0-byte image.

This displays the metadata associated with the snapshot:

```json
[
  {
    "volume_id": null,
    "guest_format": null,
    "encrypted": null,
    "volume_type": null,
    "snapshot_id": "d23dd96f-e96e-450a-a832-747587886fb2",
    "encryption_secret_uuid": null,
    "disk_bus": "virtio",
    "device_name": "/dev/vda",
    "image_id": null,
    "encryption_format": null,
    "delete_on_termination": false,
    "device_type": "disk",
    "destination_type": "volume",
    "source_type": "snapshot",
    "boot_index": 0,
    "no_device": null,
    "encryption_options": null,
    "volume_size": 1,
    "tag": null
  }
]
```
