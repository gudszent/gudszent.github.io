---
title: Proper run Ryzen CPU amd nvme ssd under ubuntu 20.04  
author: Gudszent Otto
date: 2021-02-07 14:10:00 +0800
categories: [Linux,Ubuntu]
tags: [Ryzen,Nvme]
---

## Allow Ubuntu HWE kernel to avoid Kingston A2000 reboot issue

sudo apt install linux-generic-hwe-20.04 \
sudo apt install linux-image-extra-virtual-hwe-20.04 

nvme ssd issue:\
<https://bugzilla.kernel.org/show_bug.cgi?id=195039>
\
Work around:
```bash
sudo nano /etc/default/grub
```

In the GRUB_CMDLINE_LINUX part, GRUB_CMDLINE_LINUX=””, to the end of it. \
`nvme_core.default_ps_max_latency_us=0`
```bash
sudo update-grub
```
\
\
Note\
chmod -R ug+rwX /dev/dri


