---
title: "My journal on Openstack"
date: 2023-12-19T0:00:00Z
tags:
- openstack
- kolla-ansible
- cinder
- swift
- networking
---

I've successfully set up OpenStack using Devstack and Kolla-Ansible, and you can find my configuration [here](https://github.com/hugotkk/coa).

This article is part of a series, with other sections located on different pages:
- [Kolla Ansible](/posts/openstack-kolla-ansible)
- [adminrc](/posts/openstack-adminrc)
- [OVN & Provider Network](/posts/openstack-ovn-provider-network)
- [Cinder](/posts/openstack-cinder)
- [Swift](/posts/openstack-swift)
- [Image Volume & Snapshot](/posts/openstack-image-volume-snapshot)
- [Usage & Quota](/posts/openstack-usage-quota)

On this page, I'll be detailing my personal journey through the world of OpenStack.

**Initial Steps:**

My OpenStack adventure began with an older version (Pika). I accessed it through a VirtualBox image from a Udemy course. This phase helped me understand the basics of OpenStack. 

I learned about VM creation and storage management using Horizon, similar to AWS's web console.

**Limitations and Exploring More:**

I soon noticed limitations in the Pika version, especially in networking. It was still using the Linux Bridge instead of OVN. 

My goal was to understand how different OpenStack components like Compute, Storage and Network work together. I realized that focusing on an outdated version wouldn't benefit my learning journey.

**Evolving to a More Sophisticated Setup:**

To deepen my understanding, I moved on to the latest stable release of OpenStack (2023.1), using Devstack. Initially, the installation was challenging due to some unclear steps in the script. 

However, Kolla-Ansible changed my experience. It made the installation process smoother and is recommended for production environments.

After that I did my COA Exam.

**Delving into High Availability:**

I then shifted my focus to understand High Availability (HA) in OpenStack. 

To do this, I set up a home lab with a refurbished HPE DL360p Gen8 server.

This allowed me to experiment and apply my knowledge in a practical setting.

**Engaging with the OpenStack Community:**

Joining the OpenStack community has been a significant part of my journey.

I've gained a lot of insights by observing how community members address OpenStack-related issues and queries. 

This engagement has greatly enhanced my understanding in OpenStack.
