---
title: "Broken Git Ref"
date: 2021-08-25T11:57:02+01:00
categories:
  - git
tags:
  - git
---

Facing this warning in a repo
```
warning: ignoring broken ref refs/remotes/origin/HEAD
````

https://stackoverflow.com/questions/45811971/warning-ignoring-broken-ref-refs-remotes-origin-head

After reading this article, seem the reference of refs/remotes/origin/HEAD is broken.


```
$ git symbolic-ref refs/remotes/origin/HEAD
refs/remotes/origin/master
```

I have changed the branch from master to main before. master branch is no longer exist. I think that is the reason of the warning.

```
git symbolic-ref refs/remotes/origin/HEAD refs/remotes/origin/main
```

Then the warning is fixed.


