---
title: Mikrotik minimum config with capsman
author: Gudszent Otto
date: 2020-08-03 14:10:00 +0800
categories: [Mikrotik, Capsman]
tags: [Mikrotik, Capsman]
---

Eth2 Wan port  

```
/caps-man channel
add band=2ghz-g/n control-channel-width=20mhz extension-channel=eC name=2g reselect-interval=8h
add band=5ghz-a/n/ac control-channel-width=20mhz extension-channel=eeCe name=5g reselect-interval=8h skip-dfs-channels=yes
/interface bridge
add admin-mac=C4:AD:34:11:4F:13 auto-mac=no comment=defconf name=bridge
/interface wireless
set [ find default-name=wlan1 ] antenna-gain=0 band=2ghz-b/g/n channel-width=20/40mhz-XX country=no_country_set disabled=no distance=indoors frequency=auto frequency-mode=manual-txpower installation=indoor mode=ap-bridge ssid=MikroTik-114F19 station-roaming=enabled wireless-protocol=802.11
set [ find default-name=wlan2 ] antenna-gain=0 band=5ghz-a/n/ac channel-width=20/40/80mhz-XXXX country=no_country_set disabled=no distance=indoors frequency=auto frequency-mode=manual-txpower installation=indoor mode=ap-bridge ssid=MikroTik-114F18 station-roaming=enabled wireless-protocol=802.11
/caps-man datapath
add bridge=bridge client-to-client-forwarding=yes local-forwarding=yes name=datapath1
/caps-man security
add authentication-types=wpa2-psk encryption=aes-ccm name=Wifi_pass passphrase="vanpasswordecske"
/caps-man configuration
add channel=2g channel.frequency="" country=hungary datapath=datapath1 mode=ap name=2g_config rates.basic=12Mbps,18Mbps,24Mbps,36Mbps,48Mbps,54Mbps rates.supported=12Mbps,18Mbps,24Mbps,36Mbps,48Mbps,54Mbps security=Wifi_pass ssid=Mikrotik
add channel=5g country=hungary datapath=datapath1 mode=ap name=5g_config security=Wifi_pass ssid=Mikrotik
/interface list
add comment=defconf name=WAN
add comment=defconf name=LAN
/interface wireless security-profiles
set [ find default=yes ] supplicant-identity=MikroTik
/ip hotspot profile
set [ find default=yes ] html-directory=flash/hotspot
/ip pool
add name=default-dhcp ranges=192.168.88.10-192.168.88.254
/ip dhcp-server
add address-pool=default-dhcp disabled=no interface=bridge name=defconf
/user group
set full policy="local,telnet,ssh,ftp,reboot,read,write,policy,test,winbox,password,web,sniff,sensitive,api,romon,dude,tikapp"
/caps-man access-list
add action=accept allow-signal-out-of-range=10s disabled=no signal-range=-70..120 ssid-regexp=""
add action=reject allow-signal-out-of-range=10s disabled=no ssid-regexp=""
/caps-man manager
set enabled=yes upgrade-policy=suggest-same-version
/caps-man manager interface
add disabled=no interface=bridge
/caps-man provisioning
add action=create-dynamic-enabled hw-supported-modes=gn master-configuration=2g_config name-format=identity
add action=create-dynamic-enabled hw-supported-modes=ac master-configuration=5g_config name-format=identity
/interface bridge port
add bridge=bridge comment=defconf interface=ether4
add bridge=bridge comment=defconf interface=ether5
add bridge=bridge comment=defconf interface=sfp1
add bridge=bridge comment=defconf interface=wlan1
add bridge=bridge comment=defconf interface=wlan2
add bridge=bridge interface=ether1
add bridge=bridge interface=ether3
/ip neighbor discovery-settings
set discover-interface-list=LAN
/interface list member
add comment=defconf interface=bridge list=LAN
add comment=defconf interface=ether2 list=WAN
/interface wireless cap
set bridge=bridge caps-man-addresses=127.0.0.1 discovery-interfaces=bridge enabled=yes interfaces=wlan1,wlan2
/ip address
add address=192.168.88.1/24 comment=defconf interface=bridge network=192.168.88.0
/ip dhcp-client
add comment=defconf disabled=no interface=ether2
/ip dhcp-server network
add address=192.168.88.0/24 comment=defconf gateway=192.168.88.1
/ip dns
set allow-remote-requests=yes
/ip firewall filter
add action=accept chain=input comment="defconf: accept established,related,untracked" connection-state=established,related,untracked
add action=drop chain=input comment="defconf: drop invalid" connection-state=invalid
add action=accept chain=input comment="defconf: accept ICMP" protocol=icmp
add action=accept chain=input comment="defconf: accept to local loopback (for CAPsMAN)" dst-address=127.0.0.1
add action=drop chain=input comment="defconf: drop all not coming from LAN" in-interface-list=!LAN
add action=accept chain=forward comment="defconf: accept in ipsec policy" ipsec-policy=in,ipsec
add action=accept chain=forward comment="defconf: accept out ipsec policy" ipsec-policy=out,ipsec
add action=fasttrack-connection chain=forward comment="defconf: fasttrack" connection-state=established,related
add action=accept chain=forward comment="defconf: accept established,related, untracked" connection-state=established,related,untracked
add action=drop chain=forward comment="defconf: drop invalid" connection-state=invalid
add action=drop chain=forward comment="defconf: drop all from WAN not DSTNATed" connection-nat-state=!dstnat connection-state=new in-interface-list=WAN
/ip firewall nat
add action=masquerade chain=srcnat comment="defconf: masquerade" ipsec-policy=out,none out-interface-list=WAN
/ip firewall service-port
set ftp disabled=yes
set irc disabled=yes
set h323 disabled=yes
set dccp disabled=yes
/ip service
set telnet disabled=yes
set ftp disabled=yes
set www disabled=yes
set ssh disabled=yes
set api disabled=yes
set winbox address=192.168.88.0/24
set api-ssl disabled=yes
/system clock
set time-zone-name=Europe/Budapest
/system identity
set name=Router
/tool mac-server
set allowed-interface-list=LAN
/tool mac-server mac-winbox
set allowed-interface-list=LAN
```