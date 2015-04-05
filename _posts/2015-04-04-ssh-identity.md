---
layout: post
title: SSH Identity
categories: [sysadmin]
tags: [ssh]
date: 2015-04-04 22:17:49-04:00

---

Refs:

* http://www.cyberciti.biz/faq/force-ssh-client-to-use-given-private-key-identity-file/

Manually indicate which identity private key to use when connecting:

```bash
ssh -i /backup/home/user/.ssh/myid user@unixserver1.nixcraft.com
```

or specify it in the ssh config to automatically be used when connecting to that specific host.

```bash
nano ~/.ssh/config
```

```text
Host server1.nixcraft.com
  IdentityFile ~/backups/.ssh/id_dsa
Host server2.nixcraft.com
  IdentityFile /backup/home/userName/.ssh/id_rsa
```