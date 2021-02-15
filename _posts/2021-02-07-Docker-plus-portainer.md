---
title: Ubuntu 20.04 install docker.io than run portainer  
author: Gudszent Otto
date: 2021-02-07 14:10:00 +0800
categories: [Linux,Ubuntu]
tags: [Docker,Portainer]
---

## Install docker

```bash
sudo apt update
sudo apt install docker.io
sudo systemctl enable --now docker
```

## Run Portainer
```bash
docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer-ce
```