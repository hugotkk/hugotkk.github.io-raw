---
title: Azure Functions
date: 2024-07-11
tags:
  - azure
  - function
---

## Tiers

### Consumption Plan
- Event-driven and automatically scales
- Limited to 10 minutes execution time by default (can be extended to 60 minutes with configuration)
- Cold starts
- Pay only for the time your code runs
- Does not support VNet integration

### Premium Plan
- No cold starts (always warm)
- Unlimited execution time
- Pay for pre-warmed instances
- Supports VNet integration

### Dedicated (App Service) Plan
- Runs on App Service VMs
- Can run continuously
- Supports VNet integration