---
title: "Understanding Prometheus - Metrics, Data Types, and Querying"
date: 2023-04-26
tags:
- monitor
- prometheus
---

An example of Prometheus data:

```
http_request_total{method="GET", endpoint="/contact-us", status="200"} 1 2 3 4 5 
http_request_total{method="POST", endpoint="/auth", status="400"} 1 6 8 10 15
```

## Key Concepts

* Metric: Quantity measurement (e.g.: `http_request_total`)
* Metric label: Metadata for the measurement (e.g.: `method="GET"`)
* Sample: Data point at a certain time (e.g.: `5`) - float64
* Series: Unique combination of metric labels (e.g.: `http_request_total{method="GET", endpoint="/contact-us", status="200"}` and `http_request_total{method="POST", endpoint="/auth", status="400"}`)
* Time series: Samples over time (e.g.: `1 2 3 4 5`)

## Data Types

* Instant vector: `http_request_total{method="GET"}`
* Range vector: `http_request_total{method="GET"}[5m]`
* Scalar: `numbers`

## Metric Types

Prometheus supports four metric types:

1. Gauge: Values can go up and down (e.g.: `logged_users`)
2. Counter: Values can only increase (e.g.: `http_request_total`)
3. Histogram: Provides `<metric_name>_bucket`, `<metric_name>_sum`, `<metric_name>_count`. Use `histogram_quantile()` for server-side quantile calculation (e.g.: `http_request_duration_seconds`)
4. Summary: Similar to Histogram, but quantiles are calculated client-side (application). Thus, it cannot be further aggregated.

## promql

### Operator Precedence

Prometheus supports a range of binary operators with different precedence levels. From highest to lowest precedence:

1. ^
2. *, /, %, atan2
3. +, -
4. ==, !=, &lt;=, &lt;, >=, >
5. and, unless
6. or

[Reference](https://prometheus.io/docs/prometheus/latest/querying/operators/#binary-operator-precedence)

### Modifiers

* @ 1609746000 - pretend the query time is 1609746000
* offset 5m - pretend the query time is 5 minutes ago

Have to use right after the select (before any function call)

### Vector Matching

- Vector <ops> scalar:
  - Example: http_request_total / 2

- Vector <ops> Vector:
  - Types of matching:
    - One-to-One
    - One-to-Many
    - Many-to-One
  - Matches vectors using labels by default
  - Customize matching key with ignore() or in()
  - Use group_right() or group_left() for many side
  - Use group_left(labels) to bring labels from one to many side

[Example](https://prometheus.io/docs/prometheus/latest/querying/operators/#many-to-one-and-one-to-many-vector-matches):

```
method_code:http_errors:rate5m{method="get", code="500"}  24
method_code:http_errors:rate5m{method="get", code="404"}  30
method_code:http_errors:rate5m{method="put", code="501"}  3
method_code:http_errors:rate5m{method="post", code="500"} 6
method_code:http_errors:rate5m{method="post", code="404"} 21

method:http_requests:rate5m{method="get", foo="bar"}  600
method:http_requests:rate5m{method="del", foo="bar1"}  34
method:http_requests:rate5m{method="post", foo="bar2"} 12
```

```
method_code:http_errors:rate5m{code="500"} / ignoring(code) group_left(foo) method:http_requests:rate5m
```

```
{method="get", code="500", foo="bar"} 0.04 
{method="get", code="404", foo="bar"} 0.05 
{method="post", code="500", foo="bar2"} 0.05 
{method="post", code="404", foo="bar2"} 0.175
```

If no group_left(foo), `foo=”bar”` will gone

[Reference](https://prometheus.io/docs/prometheus/latest/querying/basics/)

## Common Prometheus Functions

* `changes()`: Number of changes over time
* `time()`: Current timestamp
* `timestamp()`: Timestamp of the sample
* Derivative and Rate:
    * `deriv()`: gauge; 
    * `rate()`, `irate()`: counter
* Delta and Increase:
    * `delta()`, `idelta()`: gauge
    * `increase()`: counter
* `irate()` vs `rate()`: 
    * `irate()`: (last - first datapoint)/time range
    * `rate()`: (projected end - start time datapoint)/time range
* Aggregration:
    * `<aggregation>`: sum, count, max, min, avg, etc: Aggregates across dimensions (group by labels)
    * `<aggregation>_over_time()`: Aggregates across time (group by time)

Examples:

```
sum(http_request_total)
```

Result:

```
{} 9
```

```
sum_over_time(http_request_total{method="GET"}[5m])
```

Result:

```
{method="GET", endpoint="/contact-us", status="200"} 10 # 1+2+3+4+5
{method="POST", endpoint="/auth", status="400"} 25 #1+6+8+10+15
```

[Reference](https://prometheus.io/docs/prometheus/latest/querying/functions/#delta)

## Prometheus Client Library Usage

1. Instrumentation
2. Writing exporters
3. Pushing metrics to Pushgateway

[Gist reference](https://gist.github.com/hugotkk/85c8b71bd25b01ca44c6de97d4b293cf)

## Storage

* Not recommended to use NFS for storage: [reference](https://prometheus.io/docs/prometheus/latest/storage/#:~:text=and%20data%20durability.-,CAUTION,-%3A%20Non%2DPOSIX) for storage

## Agent Mode

* Disables query, alert, and recording rule functions
* Scrapes metrics from target and remotely writes to other instances
* [Reference](https://prometheus.io/blog/2021/11/16/agent/)

## Service Discovery

1. Static: Define target servers in the config file
2. *_sd_config: Use built-in configurations (e.g.: EC2, Kubernetes, file)
3. Custom: Use file_sd_config. Update the file periodically.

Each scrape config can have:

* interval
* timeout
* proxy
* metrics_path

## Relabeling

1. relabel_configs: Modify scrape parameters before scraping (e.g.: Blackbox exporter)
2. metrics_relabel_configs: Modify data collected after scraping (e.g.: remove unwanted metrics)

## Alerting in Prometheus

* Evaluates rules, fires alerts, routes to destination
* Does not handle notifications
* Routes by matching rules with labels
* Labels: alert identity
* Annotations: longer-form description
* Annotations support templating with go lang syntax
* Reference labels in annotations can be done by `{{ $labels.foo }}`

## Alertmanager

Silencing alerts use cases:
- Provisioning new servers
- Decommissioning servers
- Maintenance

Inhibiting:
* Stop a group of alerts when another alert is triggered
* Example: Cluster down alert inhibits memory or disk check alerts
