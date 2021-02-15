---
title: Create Raid5 with mdadm under Ubuntu  
author: Gudszent Otto
date: 2021-02-07 14:10:00 +0800
categories: [Linux,Ubuntu]
tags: [Raid5, Ubuntu, Linux]
---

## Check current status

```bash
cat /proc/mdstat
```

## Remove curent raid

```bash
sudo umount /dev/md0
sudo mdadm --stop /dev/md0
sudo mdadm --remove /dev/md0
```

## Show existed linux raid

```bash
lsblk -o NAME,SIZE,FSTYPE,TYPE,MOUNTPOINT
```

## Remove superblock
```bash
sudo mdadm --zero-superblock /dev/sda
sudo mdadm --zero-superblock /dev/sdb
```

## Remove partition table
```bash
sudo gdisk /dev/sda
GPT fdisk (gdisk) version 0.8.10

Partition table scan:
  MBR: protective
  BSD: not present
  APM: not present
  GPT: present

Found valid GPT with protective MBR; using GPT.

Command (? for help): x

Expert command (? for help): z
About to wipe out GPT on /dev/sda. Proceed? (Y/N): Y
GPT data structures destroyed! You may now partition the disk using fdisk or
other utilities.
Blank out MBR? (Y/N): Y
```

Create Raid5

```bash
sudo mdadm --create --verbose /dev/md0 --level=5 --raid-devices=4 /dev/sda /dev/sdb /dev/sdc /dev/sdd
```

Wait a until end, no restart! Check command:
`cat /proc/mdstat`

Create FileSystem

```bash
sudo mkfs.ext4 -F /dev/md0
sudo mkdir -p /mnt/raid5
sudo mount /dev/md0 /mnt/raid5
echo '/dev/md0 /mnt/raid5 ext4 defaults,nofail,discard 0 0' | sudo tee -a /etc/fstab

```

Check space
```bash
df -h -x devtmpfs -x tmpfs
```

Save the Array Layout
```bash
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
```

To available in early boot process
```bash
sudo update-initramfs -u
```