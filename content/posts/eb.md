---
title: "AWS Elastic Beanstalk"
date: 2022-02-21
tags:
- aws
- devops
- eb
---

## eb

```
eb init
```

```
eb use web_dev
```

```
eb list
```

deploy code in repo

```
eb deploy web_dev
```

edit config

```
eb config web_dev
```

save config

```
eb config save
```

apply save config

```
eb config put .elasticbeanstalk/saved_configs/Web-env-sc.cfg.yml
```

Folder Structure

```
▾ .ebextensions/
    app.config
▾ .elasticbeanstalk/
  ▸ saved_configs/
    config.yml
▸ .git/
  .gitignore
  index.php
  README
```

swap name, configuration with not swap

