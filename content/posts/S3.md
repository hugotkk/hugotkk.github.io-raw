---
title: "S3"
date: 2023-03-29
tags:
- cloud
- aws
- devops
- s3
- iam
---

# Public Access Object Setup with ACL & Bucket Policy:

## Using ACL:

1. Disable block public access ACL.
1. Select objects and make them public.
1. Add "everyone" with read permission in object's ACL section (public-read).
1. Update ACL via CLI.

## Using Bucket Policy:

1. Disable block public access bucket policy.
1. Allow everyone to get objects by updating bucket policy.
1. Specify object tags in bucket policy.

# Additional Notes:

1. Enabling ACL may cause varied object ownership in shared buckets.
1. Bucket policies only apply to objects owned by bucket owner.
1. Bucket ownership feature ensures all objects are owned by bucket owner and ACLs are disabled.
1. S3 Access Points simplify policy management by attaching policy to access point.
1. Access point policy overrides existing bucket policies for granular control.
1. Using S3 Access Points reduces risk of policy errors/inconsistencies.

