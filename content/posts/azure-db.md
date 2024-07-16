---
title: Azure Flexible Server (MySQL)
date: 2024-07-14
tags:
  - azure
  - mysql
---

- HA is not supported in the Burstable compute tier
- HA is available for General Purpose and Business Critical tiers

HA Options:
- Zone-redundant HA: Provides redundancy across AZs.
- Same-zone HA: Provides redundancy within AZ.

Business Continuity Options:
- Geo-redundant backup: Allows restoring the database in a different region.
- Cross-region read replica: Enables replicating data to a different region for read operations and potential failover.

Replication can minimizes downtime during regional failures or disasters.


