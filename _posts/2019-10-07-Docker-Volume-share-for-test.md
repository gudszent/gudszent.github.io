---
title: Share Docker Volume folder in test environment
author: Gudszent Otto
date: 2019-10-07 14:10:00 +0800
categories: [Linux,Docker]
tags: [Docker,VLAN]
---

## Install Samba
...
### Edit smb.conf

```bash
nano /etc/samba/smb.conf
```
```bash
[Volume]
path = /var/lib/docker/volumes
valid users = shareuser
force user = root
browseable = yes
writeable = yes


```
```bash
smbpasswd -a shareuser
```