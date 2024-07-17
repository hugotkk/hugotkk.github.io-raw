---
title: Azure Data
date: 2024-07-16
tags:
  - azure
  - data
---

## Batch

Azure Batch offers two pool allocation modes: 
- Batch service
- User subscription

Batch service: (prefered)
- Pools are allocated in Batch-managed subscriptions
- Supports dedicated VMs and Azure Spot VMs

User subscription:

- Pools are created directly in subscription
- Supports dedicated VMs and Azure Spot VMs
- VNET support
- We can use Azure Reserved VM Instances or applying Azure Policy to VM Scale Sets

Notes:
- Low-priority VMs are being phased out and replaced by Azure Spot VMs
- Batch service wasn't able to use Spot VMs. Only Low-priority VMs can be use.
- But now both modes now support Azure Spot VMs
- Batch supports Azure Hybrid Benefit in both modes

Use case:
- Render farm - for image rendering

## CycleCloud

It is an enterprise tool for orchestrating and managing HPC environments

It can
- Provision infrastructure for HPC systems
- Deploy familiar HPC schedulers like Slurm
- Create and manage file systems for HPC workloads
- Target at HPC user that need specific scheduler

## Service Fabric

It is a platform for building and managing microservices.

- Support both stateful and stateless microservices
- Open-source software that can be deployed in various environments
	- On-premises
	- Azure
	- Other public cloud
- Container orchestration
- Application lifecycle management

## Service Bus Queue

It is similar to AWS SQS

## Databricks

- Pass-Through: Allows using user credentials to access storage accounts from Databricks.
- Service Principal: Requires granting Databricks access to the storage account via an Entra Enterprise Application.
- Premium SKU is required for the pass-through authentication method

## Data Factory

- Can build data pipelines to ingest data into SQL Server.
- Uses self-hosted integration runtime for on-premises sources.
- Supports connections to various databases, including Oracle, using self-hosted integration runtime.

## Event Hub

- Used for streaming big data and live data processing.
- More powerful than Event Grid for high-throughput scenarios.
- Event Hub Capture doesn't support premium storage accounts but works with Data Lake Storage and Standard Blob Storage.
- Captured data is stored in Avro format.

Use cases:
- For SQL, Diagnostic Settings can only connect to Event Hub, not Event Grid.
- For Azure AD, Audit Logs can be sent to Event Hub, then processed by Functions and stored in Cosmos DB.

## Event Grid

- Used for event-driven architectures, similar to AWS EventBridge.
- Event Grid Domain allows routing to multiple Azure Functions based on event type, using a single endpoint.