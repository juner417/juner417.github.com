---
layout: post
title: "Let's try out rkt container"
date: 2017-12-30 20:00:00
categories: blog
author: juner
tags: [container, rkt]
share: true
comments: true
---

# Let's try out rkt container

## installation
 - [rkt installation on ubuntu](https://coreos.com/rkt/docs/latest/distributions.html#ubuntu)

## version 확인
```bash
root@k8s-slave-4:~# rkt version
rkt Version: 1.28.1
appc Version: 0.8.10
Go Version: go1.8.3
Go OS/Arch: linux/amd64
Features: -TPM +SDJOURNAL
```

## alpine image 실행
```bash
root@k8s-slave-4:~# rkt run quay.io/coreos/alpine-sh --interactive
pubkey: prefix: "quay.io/coreos/alpine-sh"
key: "https://quay.io/aci-signing-key"
gpg key fingerprint is: BFF3 13CD AA56 0B16 A898  7B8F 72AB F5F6 799D 33BC
 Quay.io ACI Converter (ACI conversion signing key) <support@quay.io>
Are you sure you want to trust this key (yes/no)?
yes
Trusting "https://quay.io/aci-signing-key" for prefix "quay.io/coreos/alpine-sh" after fingerprint review.
Added key for prefix "quay.io/coreos/alpine-sh" at "/etc/rkt/trustedkeys/prefix.d/quay.io/coreos/alpine-sh/bff313cdaa560b16a8987b8f72abf5f6799d33bc"
Downloading signature: [=======================================] 473 B/473 B
Downloading ACI: [=============================================] 2.65 MB/2.65 MB
image: signature verified:
  Quay.io ACI Converter (ACI conversion signing key) <support@quay.io>

# In alpine container
/ # ps auxf
PID   USER     TIME   COMMAND
    1 root       0:00 /usr/lib/systemd/systemd --default-standard-output=tty --log-target=null --show-status=0
    2 root       0:00 /usr/lib/systemd/systemd-journald
    5 root       0:00 /bin/sh -c /bin/sh
   10 root       0:00 /bin/sh
   11 root       0:00 ps auxf
/ # apk
apk-tools 2.6.3, compiled for x86_64.

usage: apk COMMAND [-h|--help] [-p|--root DIR] [-X|--repository REPO] [-q|--quiet] [-v|--verbose] [-i|--interactive] [-V|--version]
           [-f|--force] [-U|--update-cache] [--progress] [--progress-fd FD] [--no-progress] [--purge] [--allow-untrusted] [--wait TIME]
           [--keys-dir KEYSDIR] [--repositories-file REPOFILE] [--no-network] [--arch ARCH] [--print-arch] [ARGS]...

The following commands are available:
  add       Add PACKAGEs to 'world' and install (or upgrade) them, while ensuring that all dependencies are met
  del       Remove PACKAGEs from 'world' and uninstall them
  fix       Repair package or upgrade it without modifying main dependencies
  update    Update repository indexes from all remote repositories
  info      Give detailed information about PACKAGEs or repositores
  search    Search package by PATTERNs or by indexed dependencies
  upgrade   Upgrade currently installed packages to match repositories
  cache     Download missing PACKAGEs to cache and/or delete unneeded files from cache
  version   Compare package versions (in installed database vs. available) or do tests on literal version strings
  index     Create repository index file from FILEs
  fetch     Download PACKAGEs from global repositories to a local directory
  audit     Audit the directories for changes
  verify    Verify package integrity and signature
  dot       Generate graphviz graphs
  policy    Show repository policy for packages
  stats     Show statistics about repositories and installations

Global options:
  -h, --help              Show generic help or applet specific help
  -p, --root DIR          Install packages to DIR
  -X, --repository REPO   Use packages from REPO
  -q, --quiet             Print less information
  -v, --verbose           Print more information (can be doubled)
  -i, --interactive       Ask confirmation for certain operations
  -V, --version           Print program version and exit
  -f, --force             Do what was asked even if it looks dangerous
  -U, --update-cache      Update the repository cache
  --progress              Show a progress bar
  --progress-fd FD        Write progress to fd
  --no-progress           Disable progress bar even for TTYs
  --purge                 Delete also modified configuration files (pkg removal) and uninstalled packages from cache (cache clean)
  --allow-untrusted       Install packages with untrusted signature or no signature
  --wait TIME             Wait for TIME seconds to get an exclusive repository lock before failing
  --keys-dir KEYSDIR      Override directory of trusted keys
  --repositories-file REPOFILE Override repositories file
  --no-network            Do not use network (cache is still used)
  --arch ARCH             Use architecture with --root
  --print-arch            Print default arch and exit

This apk has coffee making abilities.
```

## rkt에서 docker image 실행
 - [running docker image](https://coreos.com/rkt/docs/latest/running-docker-images.html)  

```bash
root@k8s-slave-4:~# rkt run docker://alpine --interactive --insecure-options=image --exec=/bin/sh
run: Get https://auth.docker.io/token?scope=repository%3Alibrary%2Falpine%3Apull&service=registry.docker.io: net/http: TLS handshake timeout
root@k8s-slave-4:~# rkt run docker://alpine --interactive --insecure-options=image --exec=/bin/sh
Downloading sha256:88286f41530 [=============================] 1.99 MB / 1.99 MB
/ # ps auxf
PID   USER     TIME   COMMAND
    1 root       0:00 /usr/lib/systemd/systemd --default-standard-output=tty --log-target=null --show-status=0
    3 root       0:00 /usr/lib/systemd/systemd-journald
    5 root       0:00 /bin/sh
   10 root       0:00 ps auxf
/ # 

root@k8s-slave-4:~# rkt run docker://nginx --interactive --insecure-options=image --exec=/bin/sh
Downloading sha256:afeb2bfd31c [=============================] 22.5 MB / 22.5 MB
Downloading sha256:7ff5d10493d [=============================] 21.9 MB / 21.9 MB
Downloading sha256:d2562f1ae1d [=============================]     201 B / 201 B
# 

root@k8s-slave-4:~# rkt run docker://170.30.120.42:5000/nginx --interactive --insecure-options=all-fetch --exec=/bin/bash
Downloading sha256:a3ed95caeb0 [=============================]       32 B / 32 B
Downloading sha256:a3ed95caeb0 [=============================]       32 B / 32 B
Downloading sha256:a3ed95caeb0 [=============================]       32 B / 32 B
Downloading sha256:a3ed95caeb0 [=============================]       32 B / 32 B
Downloading sha256:0bed9719ddc [=============================]     193 B / 193 B
Downloading sha256:a3ed95caeb0 [=============================]       32 B / 32 B
Downloading sha256:357ea8c3d80 [=============================] 51.4 MB / 51.4 MB
Downloading sha256:7a2b54f3390 [=============================]     20 MB / 20 MB
root@rkt-1e2ccecc-2e7a-4e4f-8eb1-db628e965d1f:/# ls
bin  boot  dev etc  home  lib lib64  media  mnt  opt proc  root  run  sbin  srv  sys  tmp  usr  var
```

## image 목록 보기
```bash
root@k8s-slave-4:~# rkt image list
ID   NAME      SIZE IMPORT TIME LAST USED
sha512-1308db08fa77 coreos.com/rkt/stage1-coreos:1.28.1  206MiB 18 hours ago 18 hours ago
sha512-2222d0a86708 quay.io/coreos/alpine-sh:latest   11MiB 18 hours ago 18 hours ago
sha512-b357ce7cf282 registry-1.docker.io/library/alpine:latest 8.2MiB 18 hours ago 18 hours ago
sha512-c3a617a36429 registry-1.docker.io/library/nginx:latest 209MiB 18 hours ago 18 hours ago
sha512-a607d63e5e28 172.19.136.42_5000/nginx:latest   359MiB 21 seconds ago 19 seconds ago
```
