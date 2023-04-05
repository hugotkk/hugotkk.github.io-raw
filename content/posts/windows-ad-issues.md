---
title: "Active Directory Issues and Solutions"
date: 2023-04-05
tags:
- windows
- ad
---

## DC Cannot Replicate

* Problem: [Target principal name incorrect during AD replication](https://learn.microsoft.com/en-us/troubleshoot/windows-server/identity/replication-error-2146893022)
* Solution ([Demote and Promote AD](https://www.youtube.com/watch?v=sdJdwslWkf4)):
  * Demote the DC at "Remove Roles and Features"
  * Manually clean update the metadata:
    * Remove the DC record at AD Users and Computers
    * Remove the DC-related record at AD Sites and Services & in DNS
  * Promote the DC back
* Result:
  * Replication resumes, syncing with other DCs
  * For "PRC server not operating" error, check DNS issues with [dcdiag](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc731968(v=ws.11)) and try [dns scavenging](https://www.youtube.com/watch?v=V-ODqxpY8vA)

## PDC Gone, Operation Master Role not Transferred

* Solution: Forceful Takeover
  * Use [Move-ADDirectoryServerOperationMasterRole](https://learn.microsoft.com/en-us/powershell/module/activedirectory/move-addirectoryserveroperationmasterrole?view=windowsserver2022-ps) to move the Operation Master.

## Removing an Orphaned Domain
* Problem: Trusted domain removed without demoting DC, resulting in ghost domain/DC
* Solution:
  * Use ntdsutil metadata cleanup for both domain controller and domain
  * Delete domain controller with [How to remove a domain controller that no longer exists?](https://www.manageengine.com/products/active-directory-audit/kb/how-to/how-to-remove-a-domain-controller-that-no-longer-exists.html) guide
  * Delete domain with [remove orphan domain](https://learn.microsoft.com/en-GB/troubleshoot/windows-server/identity/remove-orphaned-domains) guide

  