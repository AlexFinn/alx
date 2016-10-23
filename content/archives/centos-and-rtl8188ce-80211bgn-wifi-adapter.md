+++
title = "CentOS and RTL8188CE 802.11b/g/n WiFi Adapter"
date = "2013-11-20"
tags = ["centos", "wifi", "realtek"]
type = "post"
+++

I use CentOS6 on my Lenovo Thinkpad X220. This laptop has RTL8188CE 802.11b/g/n WiFi Adapter:

	03:00.0 Network controller: Realtek Semiconductor Co., Ltd. RTL8188CE 802.11b/g/n WiFi Adapter (rev 01)
	Subsystem: Realtek Semiconductor Co., Ltd. Device 8195
	Flags: bus master, fast devsel, latency 0, IRQ 17
	I/O ports at 5000 [size=256]
	Memory at f2400000 (64-bit, non-prefetchable) [size=16K]
	Capabilities: <access denied>
	Kernel driver in use: rtl8192ce
	Kernel modules: rtl8192ce

<!--more-->

To this device to work you must install the following package:

	[alexfinn@x220 ~]$ rpm -qa | grep 8192
	kmod-r8192ce-0006.0321.2011-1.el6.elrepo.x86_64

from [ELRepo](http://elrepo.org/) repository.

And I found a useful [tip](https://bbs.archlinux.org/viewtopic.php?id=132931) for the adequate functioning of the adapter.
I created `rtl8192ce.conf` in `/etc/modprobe.d`. And I added this to this file:

	[alexfinn@x220 ~]$ cat /etc/modprobe.d/rtl8192ce.conf 
	# Disable powersaving 
	options rtl8192ce ips=0

	# WARNING! Do not enable this shit. 
	#          It causes bugs. 
	options rtl8192ce fwlps=0

	# Use software control instead 
	options rtl8192ce swlps=1

Then you must restart a laptop. 

Prior to the creation of the file and add these lines there connection broke off often and was not restored.
