---
title: OpenMQTTGateway
author: Gudszent Otto
date: 2021-05-04 14:10:00 +0800
categories: [openHAB, OpenMQTTGateway]
tags: [openHAB, OpenMQTTGateway]
---

/main/User_config.h
```
#ifndef MQTT_USER
#  define MQTT_USER "user"
#endif
#ifndef MQTT_PASS
#  define MQTT_PASS "pass"
#endif
#ifndef MQTT_SERVER
#  define MQTT_SERVER "IP"
```
/platformio.ini:
Select environment
```
default_envs = esp32-olimex-gtw-ble-eth
```
OTA command in terminal
```
pio run -t upload --upload-port 192.168.0.100
```
Releases:
https://github.com/1technophile/OpenMQTTGateway/releases/
