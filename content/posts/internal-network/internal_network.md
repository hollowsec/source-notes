---
title: "Malware Analysis Lab: Internal Network"
subtitle: "Set up a truly isolated lab network for malware analysis"
date: 2022-09-27T04:43:17-03:00
draft: false
author: "HollowSec"
authorLink: "https://twitter.com/hollowsec_"
authorEmail: ""
description: ""
keywords: "network, internal network, malware, lab"
license: ""
comment: false
weight: 0

tags:
- network
- malware
- lab 
categories:
- lab

hiddenFromHomePage: false
hiddenFromSearch: false

summary: ""
resources:
- name: featured-image
  src: featured-image.png
- name: featured-image-preview
  src: featured-image.png 

toc:
  enable: true
math:
  enable: false
lightgallery: false
seo:
  images: []

repost:
  enable: true
  url: ""

# See details front matter: https://fixit.lruihao.cn/theme-documentation-content/#front-matter
---


The Internal Network provides an additional level of security that removes the reliance on a host firewell to protect our physical host.

I will use VirtualBox for the configuration of Windows 10 and REMnux (Linux Toolkit for Malware Analysis) for my Internal Network.

<!--more-->
## REMnux Configuration
Perform the following steps to create an Internal Network:
- In VirtualBox, right-click on the REMnux VM and go to settings
- Network
- In the Attached To dropdown menu, select Internal Network
- Add a name to the new Internal Network
- Click the drop down arrow next to Advanced  and change the Promiscuous Mode to "Allow VMs"
- Click Ok and Power on the REMnux

![remnux-network.png](/img/remnux-network.png)
### Static IP Configuration

There is no DHCP server in the Internal Network, we have to manually define a static IP adress for the REMnux host. For this we gonna use <b>netplan</b> utility. <b>Netplan</b> utility uses a YAML config file to control the host's adapter settings. 

Open the netplan config file in vim with sudo:
```bash
$ sudo vim /etc/netplan/01-netcfg.yaml
```

```yaml
# This file describes the network interfaces available on your system
# For more information, see netplan(5).
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:
      dhcp4: yes
```

We want to remove the DHCP and assign all network information statically.

Change `dhcp4` to `no`, add the `addresses` and `gateway4` objects.


```yaml
# This file describes the network interfaces available on your system
# For more information, see netplan(5).
network:
  version: 2
  renderer: networkd
ethernets:
    enp0s3:
     dhcp4: no
     addresses: [10.10.0.3/24]
     gateway4: 10.0.0.1
```

```bash
$ sudo netplan apply
```

``` bash
$ ip -br -c a
lo               UNKNOWN        127.0.0.1/8 ::1/128 
enp0s3           UP             10.0.0.3/24 fe80::a00:27ff:fe0f:f449/64 
```

### iNetSim Configuration
iNetSim is going to be our Internet Simulator.

(iNetSim always gonna give `HTTP 200 OK` and give fake binaries for request, is awesome to capture network indicators of malwares.)

```bash
$ sudo vim /etc/inetsim/inetsim.conf 
```

We want to enable `start_service dns` and change our `dns_default_ip`


``` 
# echo_udp, discard_tcp, discard_udp, quotd_tcp,
# quotd_udp, chargen_tcp, chargen_udp, finger,
# ident, syslog, dummy_tcp, dummy_udp, smtps, pop3s,
# ftps, irc, https
#
start_service dns
start_service http
start_service https
start_service smtp
start_service smtps
start_service pop3
start_service pop3s
start_service ftp
start_service ftps
#start_service tftp
#start_service irc
#start_service ntp

```

``` 
#########################################
# dns_default_ip
#
# Default IP address to return with DNS replies
#
# Syntax: dns_default_ip <IP address>
#
# Default: 127.0.0.1
#
dns_default_ip          10.0.0.3

```


## Windows Configuration
Windows is more straightforward than REMnux for this setup.
- In VirtualBox, right-click on the Windows VM and select settings
-  Network
- Associate Windows to the same Internal Network that we made earlier
- Click the drop down arrow next to Advanced  and change the Promiscuous Mode to "Allow VMs"
- Click Ok and Power on the Windows

![windows-network.png](/img/windows-network.png)
On Windows search for <b>View Network Connections </b>, to get to the Network Connection Adapter Settings.

Right-click on the <b>Ethernet Adapter</b> and select <b>Properties → IPv4 Properties</b>.

We will now statically define our IPv4 settings:

-   The IPv4 address it doesn’t matter what it’s set to as long as it’s a valid address in the `10.0.0.0/24` network.
-   The Subnet Mask to configure a `/24` network.
-   Our Default Gateway to be the REMnux host so we can make use of INetSim.
-   Our DNS server to be the REMnux host so we can make use of INetSim.


![windows-ip.png](/img/windows-ip.png)

``` 
c:\Users\hollow\Desktop>ipconfig /all

Windows IP Configuration

   Host Name . . . . . . . . . . . . : DESKTOP-0RVV361
   Primary Dns Suffix  . . . . . . . :
   Node Type . . . . . . . . . . . . : Hybrid
   IP Routing Enabled. . . . . . . . : No
   WINS Proxy Enabled. . . . . . . . : No

Ethernet adapter Ethernet:

   Connection-specific DNS Suffix  . :
   Description . . . . . . . . . . . : Intel(R) PRO/1000 MT Desktop Adapter
   Physical Address. . . . . . . . . : 08-00-27-CA-87-0F
   DHCP Enabled. . . . . . . . . . . : No
   Autoconfiguration Enabled . . . . : Yes
   Link-local IPv6 Address . . . . . : fe80::8cab:59ec:da25:aebd%4(Preferred)
   IPv4 Address. . . . . . . . . . . : 10.0.0.22(Preferred)
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 10.0.0.3
   DHCPv6 IAID . . . . . . . . . . . : 101187623
   DHCPv6 Client DUID. . . . . . . . : 00-01-00-01-2A-77-59-DC-08-00-27-CA-87-0F
   DNS Servers . . . . . . . . . . . : 10.0.0.3
   NetBIOS over Tcpip. . . . . . . . : Enabled

Ethernet adapter Npcap Loopback Adapter:

   Connection-specific DNS Suffix  . :
   Description . . . . . . . . . . . : Npcap Loopback Adapter
   Physical Address. . . . . . . . . : 02-00-4C-4F-4F-50
   DHCP Enabled. . . . . . . . . . . : Yes
   Autoconfiguration Enabled . . . . : Yes
   Link-local IPv6 Address . . . . . : fe80::31fb:9a74:650a:82df%5(Preferred)
   Autoconfiguration IPv4 Address. . : 169.254.130.223(Preferred)
   Subnet Mask . . . . . . . . . . . : 255.255.0.0
   Default Gateway . . . . . . . . . :
   DHCPv6 IAID . . . . . . . . . . . : 184680524
   DHCPv6 Client DUID. . . . . . . . : 00-01-00-01-2A-77-59-DC-08-00-27-CA-87-0F
   DNS Servers . . . . . . . . . . . : fec0:0:0:ffff::1%1
                                       fec0:0:0:ffff::2%1
                                       fec0:0:0:ffff::3%1
   NetBIOS over Tcpip. . . . . . . . : Enabled
```

# Done. Internal Network is configured
Now we are ready to detonate some samples of malware on our Windows machine and use our internet simulator(REMnux) to capture `Network Indicators.`


# That's All Folks



