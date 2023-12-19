---
title: "Basic Operations in Openstack"
date: 2023-12-19
tags:
- openstack
- linux
---

## adminrc

### Via Horizon

![](https://lh7-us.googleusercontent.com/gsvYcUHaaXeUAEW4g6n0GqBumI7UmHFfHH8rZ4Vb_UDgNDT2oOGiLTHnZdYBkRmjBK5Iff2PNjPuyGo8C09HofM-r821h-oIML-otVPyr32uQHHMtVmBa3XrkOOFYXrTo6BjZgcVwFC5OC-HV1mh6vY)


### Via Command Line

While there isn't a direct way to generate the adminrc file from the command line, you can use the OpenStack CLI to fetch the necessary authentication parameters and convert them into environment variables.

Check Available Options:

```bash
openstack -h
```

| Parameter    | CLI Argument           | Environment Variable                     |
|--------------|------------------------|------------------------------------------|
| API endpoint | `--os-auth-url http://controller:5000` | `export OS_AUTH_URL=http://controller:5000` |
| API Version  | `--os-identity-api-version 3` | `export OS_IDENTITY_API_VERSION=3`       |
| User         | `--os-username admin`  | `export OS_USERNAME=admin`                |
| Password     | `--os-password admin`  | `export OS_PASSWORD=admin`                |
| Project      | `--os-project-name admin` | `export OS_PROJECT_NAME=admin`           |
| Domain       | `--os-domain-name Default` | `export OS_DOMAIN_NAME=Default`         |


Use the converted environment variables to create the adminrc file. eg:

vim adminrc

```bash
export OS_AUTH_URL=http://controller:5000
export OS_IDENTITY_API_VERSION=3
export OS_USERNAME=admin
export OS_PASSWORD=admin
export OS_PROJECT_NAME=admin
export OS_DOMAIN_NAME=Default
```

Once the adminrc file is ready, we can

```bash
source adminrc
```

