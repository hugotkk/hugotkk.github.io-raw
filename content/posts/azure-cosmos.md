---
title: Azure Cosmos
date: 2024-07-16
tags:
  - azure
  - cosmos
---

Cosmos DB is the only DB service supports multi-master writes across multiple regions

This feature allows for write operations in any region, providing low-latency writes globally

Cosmos DB supports multiple APIs, including 
- SQL
- MongoDB
- Cassandra
- Gremlin
- Table

## Cosmos for Postresql

- It is a relational database, not a NoSQL database like the core Cosmos DB
- It supports high availability through read replicas
- The primary node handles write operations
- Read replicas can be used to scale out read operations and provide failover capabilities

## Synapse Link

- Enables near real-time analytics on operational data in Cosmos DB
- Allows querying Cosmos DB data from Azure Synapse Analytics without impacting the performance of transactional workloads

## Change Feed

- Similar to AWS DynamoDB streams
- Provides a log of all changes to data in Cosmos DB
- Useful for event-driven architectures and real-time data processing

## Multi-Master and Scalability

- Supports multi-master writes across multiple regions
- Offers unlimited read and write scalability

## Query Language

- NoSQL but supports SQL-like queries for data retrieval

## Authentication

- Primary/Secondary Key: Similar to account keys in Storage Account
- Resource Tokens: Fine-grained, time-limited access tokens
- Cosmos DB Users: Self-managed users (similar to MySQL users)
- RBAC: with Azure AD identity