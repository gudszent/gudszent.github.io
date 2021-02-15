---
title: Connecting the Docker container to the external network via VLAN  
author: Gudszent Otto
date: 2020-10-07 14:10:00 +0800
categories: [Linux,Docker]
tags: [Docker,VLAN]
---

## Create Docker network
Get network id: 
`ip addr show`\
Create the Docker network with MACVLAN driver. The part ens192 of parent adapts to the environment.

```bash
 docker network create -d macvlan \
    --subnet=172.6.2.0/24 \
    --gateway=172.6.2.1 \
    -o parent=enp2s0.20 vlan20

 docker network create -d macvlan \
    --subnet=172.6.2.0/24 \
    --gateway=172.6.2.1 \
    -o parent=enp2s0.40 vlan40
```
## Create Docker container

Create a container by specifying the created network and IP address. You can use Ping & Traceroute, so use Alpine. 

```bash
docker run -it -d --rm \
   --net=vlan20 \
    --ip=172.6.2.201 \
    --name container-vlan20 \
    alpine /bin/sh
```

## Check working

Confirm communication from each container.

```bash
 docker exec -it container-vlan20 /bin/sh
 ping 8.8.8.8
```

## Docker compose

cat ./docker-compose.yml

```bash
version: '2.1'

services:
  vlan20:
    image: alpine
    container_name: container-vlan20
    command: ['tail', '-f', '/dev/null']
    networks:
      vlan20:
        ipv4_address: 172.6.2.200
  vlan40:
    image: alpine
    container_name: container-vlan40
    command: ['tail', '-f', '/dev/null']
    networks:
      vlan30:
        ipv4_address: 172.6.4.200


networks:
  vlan20:
    name: vlan20
    driver: macvlan
    driver_opts:
      parent: enp2s0.20
    ipam:
      config:
        - subnet: 172.6.2.0/24
          gateway: 172.6.2.1
  vlan30:
    name: vlan40
    driver: macvlan
    driver_opts:
      parent: enp2s0.40
    ipam:
      config:
        - subnet: 172.6.4.0/24
          gateway: 172.6.4.1
```