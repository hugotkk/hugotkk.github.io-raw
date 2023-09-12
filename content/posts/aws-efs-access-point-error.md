---
title: "Fixing Access Issues Between ECS Containers and EFS Access Points"
date: 2023-09-11
tags:
- aws
- efs
---

I recently had trouble connecting an ECS container to an EFS access point. Here's an breakdown of what happened and how I fixed it:

**The Problem:**

When I tried to link an ECS container with an EFS access point, I got this error: `"'b'mount.nfs4: access denied by server while mounting 127.0.0.1:/'"`

**First Thoughts:**

I thought the issues could be:
- EFS's security group blocking our connection.
- Not having the right permissions in the task's IAM role.

**What I Tried:**

To figure out what was wrong, I did two things:
1. I tried connecting the EFS without the access point directly from EC2.
2. I changed the volume mount settings to leave out the access point.
Both worked perfectly, so my first thoughts weren't the problem.

After some digging, I found a [helpful article](https://repost.aws/knowledge-center/efs-access-points-directory-access). It explained that if you don't set up the right ownership and permission settings for the root directory of an access point, EFS won't create it. This was causing the "access denied" error.

**The Solution:**

When setting up the access point, I made sure to fill in the `owner id`, `owner group id`, and default file permissions in the `Root directory creation permission` section. This fixed the error.

---