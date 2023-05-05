---
title: "Mistake when updating docker-compose.yml"
date: 2023-05-05
tags:
- docker
---

After updating the `command` or `env` parameters in `docker-compose.yml` file, I restarted the container with `docker-compose restart` to make the change effect.

However, I realize that the changes were not applied, and the container is still using the old config.

To apply the changes, I have to re-create the container:

```
docker-compose down
docker-compose up -d
```

This will stop and remove the existing container and create a new one with the updated config.