+++
title = "CentOS. Mount cgroups for LXC"
tags = ["centos", "cgroups", "lxc"]
date = "2013-11-13"
type = "post"
+++

A short note :)

We should use following cgroups mount options for LXC on CentOS 6:

	$ mount -t cgroup -o cpuset,memory,cpu,devices,net_cls none /cgroup

Or we can add appropriate line to `/etc/fstab` file.

We have to specify controllers which should be used, because blkio mounts in CentOS6/RHEL6 by default. But this controller doesn't support nested hierarchy that are necessary for LXC. Ubuntu/Debian doesn't have such problem.

