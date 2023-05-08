---
title: "Brief Overview of Scan, Query, Select, and Filter of DyanmoDB"
tags:
- cloud
- aws
- db
- ddb
---

Scan and Query:

- Amazon DynamoDB has two operations for retrieving data: Scan and Query.
- Scan retrieves all items in a table, making it inefficient for larger tables.
- Query is designed to retrieve specific items based on their partition key values, allowing filtering of results.

Indexing:

- The primary key consists of the partition key and the sort key (optional), which uniquely identify each item in a table.
- While the primary key is essential, it may not always be sufficient for accessing data in the way needed.
- Global Secondary Index (GSI) is a separate index with its partition key and sort key that can be used for querying on non-primary key attributes.
- GSI can significantly reduce the number of items that need to be scanned and improve query speed.
- Primary key has to be unique, but GSI can be duplicated.

Select and Filter:

- Select and Filter expressions can also be used to specify attributes and conditions for retrieving data.
- Query is generally recommended for specific and efficient data retrieval, especially when used with a GSI.
- The choice of which operation to use depends on the specific use case and how data needs to be accessed and queried.
