---
title: "Working with DateTime in AWS Athena"
date: 2023-05-05
tags:
- cloud
- aws
- athena
---

Comparing varchar DateTime:

- Use '>' and '<' operators
Example:
```
time >= '2022-07-20T00:00:00Z'
```

Comparing int64 DateTime:

1. Convert int64 and varchar to timestamp:
```
FROM_UNIXTIME(time/1000) < TIMESTAMP '2022-07-20 00:00:00'
```

2. Convert int64 to varchar:
```
DATE_FORMAT(FROM_UNIXTIME(time/1000), '%Y-%m-%dT%H:%i:%sZ') <= '2022-02-13T00:00:00Z'
```