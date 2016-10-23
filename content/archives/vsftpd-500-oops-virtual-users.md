+++
title = "Vsftpd. 500 OOPS. Virtual users"
date = "2013-11-29"
tags = ["linux", "vsftpd"]
type = "post"
+++

The following error may occur on the client side: Fatal error: 500 OOPS: priv_sock_get_cmd if running an amd64 kernel with vsftpd-3.0.x.

<!--more-->

To solve this bug you will need to add the following to your /etc/vsftpd/vsftpd.conf:

	seccomp_sandbox=NO

Refer to [https://bugzilla.redhat.com/show_bug.cgi?id=845980](https://bugzilla.redhat.com/show_bug.cgi?id=845980) and [Gentoo Wiki](https://wiki.gentoo.org/wiki/Vsftpd)

P.S. 
And just in case add next line to config file

	allow_writeable_chroot=YES

for adequate working of virtual users.


