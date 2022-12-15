---
title: "Notes about Prometheus & Grafana"
date: 2022-12-15
tags:
- monitor
- grafana
- prometheus
---

## Installation

* [Prometheus & the related components](https://prometheus.io/download/) (eg: pushgateway, blackbox_exporter, alertmanager) - scrape engine - it's better to look at the `README.md` and  `Dockerfile` on their github repo to get the information about the [docker](https://github.com/prometheus/prometheus#docker-images) setup
* [Grafana](https://grafana.com/docs/grafana/latest/setup-grafana/installation/) - for visualization; drawing graphs
* Node Exporter (via package manager) - [Centos7](https://rhel.pkgs.org/7/epel-x86_64/golang-github-prometheus-node-exporter-1.2.2-1.el7.x86_64.rpm.html) (need [epel](https://docs.fedoraproject.org/en-US/epel/#_el7) repo); [Ubuntu20.04](https://packages.ubuntu.com/search?keywords=node-exporter&searchon=names&suite=focal&section=all) - monitor VM's states
* [cAdvisor](https://github.com/google/cadvisor) - monitor docker host's states
* [Others exporters](https://prometheus.io/docs/instrumenting/exporters/)

* [kube-prometheus-stack](https://artifacthub.io/packages/helm/prometheus-community/kube-prometheus-stack) (eks) - included grafana and prometheus

## References

* [docker-compose.yml](https://grafana.com/docs/grafana-cloud/quickstart/docker-compose-linux/) for prometheus and node exporter
* [grafana.ini](https://grafana.com/docs/grafana/latest/setup-grafana/configure-grafana/)
* [prometheus.yml](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#configuration-file)
* [alertmanager.yml](https://prometheus.io/docs/alerting/latest/configuration/)
* [web-config.yml](https://prometheus.io/docs/prometheus/latest/configuration/https/) - tls and basic auth settings - pass `--web.config.file` when starting the binary. This works on any prometheus related products eg: prometheus, node_exporter, alertmanager
* [prometheus query functions](https://prometheus.io/docs/prometheus/latest/querying/functions/)

## Grafana Dashboards

* [cadvisor](https://grafana.com/grafana/dashboards/893-main/)
* [node-exporter](https://grafana.com/grafana/dashboards/1860-node-exporter-full/)

## Prometheus

* Customise the [storage](https://prometheus.io/docs/prometheus/latest/storage/#operational-aspects) config
* [Setup an alert](https://prometheus.io/docs/tutorials/alerting_based_on_metrics/)
* [Setup federation](https://prometheus.io/docs/prometheus/latest/federation/#configuring-federation) - a primary prometheus will work as an aggregator to collect metrics from other prometheus instances. This 
can reduce the loading on a single prometheus server.
* [Setup recording rules](https://prometheus.io/docs/prometheus/latest/configuration/recording_rules/)
* [Setup node-exporter with push gatewaty](https://gist.github.com/elritsch/60cbbdcec4d3f1519ce35dc23242ac1f) - but this isn't recommended. The reasons are in [here](https://prometheus.io/docs/practices/pushing/#should-i-be-using-the-pushgateway)
* [Snapshot](https://prometheus.io/docs/prometheus/latest/querying/api/#snapshot) (backup)
* [EC2 host discovery](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#ec2_sd_config) - use for dynamically adding the ec2 instances as the scrape targets

### Thanos

To archive the HA, third-party solutions like thanos will be needed. 

These two articles are very good introduction of the concept of thanos
* [Deep Dive into Thanos-Part I](https://medium.com/nerd-for-tech/deep-dive-into-thanos-part-i-f72ecba39f76)
* [Deep Dive into Thanos-Part II](https://medium.com/nerd-for-tech/deep-dive-into-thanos-part-ii-8f48b8bba132)

Also, there is a [hand-on lab](https://killercoda.com/thanos) provided by killercoda which is recommended by [thanos](https://thanos.io/tip/thanos/quick-tutorial.md/#quick-tutorial)

[This](https://thanos.io/tip/thanos/quick-tutorial.md/#components) shows the architecture of thanos.

The thanos gateway works as a proxy on each prometheus instance (sidecar) to communicate to other thanos components. Also, data can be optionally saved to object stores (like aws s3 / mino) for long-term storage as well.

Queries will add an additional layer to the system. Instead of obtaining the data directly from prometheus, the client (grafana / end user) will now talk to the thanos querier instead. The queries will
* deduplicate the data they get from the replicas
* combine the data from the long term object store and shards

Thus, with the queries, prometheus instances can logically group into different shards / replicas.

## Grafana

* [Provision dashboards](https://grafana.com/docs/grafana/latest/administration/provisioning/#dashboards) and [Data sources](https://grafana.com/docs/grafana/latest/administration/provisioning/#data-sources)
* But we can't provision the users, orgs, alerts
* [Add prometheus to grafana ](https://prometheus.io/docs/visualization/grafana/)
* [Import or export a dashboard](https://grafana.com/docs/grafana/v9.0/dashboards/export-import/)
* [Backup grafana](https://grafana.com/docs/grafana/v9.0/setup-grafana/upgrade-grafana/#backup)
* [Create repeated row / panel](https://grafana.com/docs/grafana/latest/panels-visualizations/configure-panel-options/#configure-repeating-panels)
* Instead of creating a [field overriding](https://grafana.com/docs/grafana/latest/panels-visualizations/configure-overrides/#configure-field-overrides), we can [choose the series color](https://grafana.com/docs/grafana/latest/panels-visualizations/visualizations/configure-legend/#change-a-series-color) quickly by just clicking the color bar next to the series
* HA in Grafana - [setup database as a backend](https://grafana.com/docs/grafana/latest/setup-grafana/set-up-for-high-availability/) to share the persistent data




