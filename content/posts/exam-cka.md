---
title: "Note for CKA Exam"
date: 2021-11-26
tags:
- cert
- exam
- cka
- k8s
---

## Things to turn off before exam

### Bandwidth saving

* google drive sync
* any backgorund updates
* wifi connection in other devices

### Prevent from autoupdate

* software update (mbp, iphone)

### Others

* firewall
* disable all chrome plugins

## backup plan

* sim card - two extra network: EE, giffgaff
* spare computer (MBP)

## Tips

### When use wc -> be careful the header, need `total - 1`

### Store temporary commands / note / result in a fixed location like `/root/tmp`

### Even in remote host; we use that fixed location to store our results then get back the result from it

```
remote$ cat tmp
remote$ exit
```

```
remote$ exit
local$ ssh remote cat tmp
```

### Do not use tmux

hard to copy and paste and scroll

### One question one yml

Name with `q<no>.yml` like `q10.yml` is the yml of 10<sup>th</sup> question

### Write all resources on same yml in same question

### Set .vimrc

```
set sw=2 ts=2 et
set hidden
```

### Use alias

```
export do='--dry-run=client -o yaml'
export now='--sort-by="{.metadata.creationTimestamp}"'
```

### Use variable in command

```
n=namespace
nn=name
i=image
s=server
```

Then you will find that we can reuse many command history. like 

#### Create po yml

```
k run -n $n $nn --image $i $do
```

#### Get po 

```
k get po $nn -n $n
```

#### Test service account

```
k auth can-i --as=system:serviceaccount:$n:$nn
```

### When working on next question, start a new vim (kill the old session)

### Keep a single vim session in bash 

* back and fore from vim and bash many times is needed
* using multiple vim session are so confusing
* that also why working on single yaml per question is encouraged

### Create (`touch /opt/xxxxx/q11/answer`) the answer file first 

* needed to submit anser to specfic file
* just touch the file -> open it with vim -> put it in background (c-z)
