+++
title = "Broken VLC related dependencies in CentOS6"
date = "2012-09-15"
tags = ["centos", "repository", "yum"]
type = "post"
+++
[This article on Russian](http://hrafn.me/2012/09/15/vlc-dependencies-centos6/)

From time to time I use [RERemix](http://mirror.yandex.ru/fedora/russianfedora/releases/RERemix/6.2/) on my laptop. As it's known (or maybe it's not known) this distribution is based on [Scientific Linux](https://www.scientificlinux.org/), which, in turn is based on [RHEL](http://www.redhat.com/products/enterprise-linux/). RERemix differs generally avaliable repositories right after installation, patched packages for normal font display and other useful things :)

There is popular multimedia player VLC among the others. With installation of its problems isn't present any, enough, that the necessary repository was connected, for example, [RPMForge (Repoforge)](http://repoforge.org/). Or, as it's made in RERemix - [PUIAS](http://puias.math.ias.edu/).
However after VLC installation from this repositories and after updating some packages from EPEL, thre is  high probability to receive something, like this:

      Error: Package: gstreamer-plugins-bad-0.10.19-3.el6.rf.i686 (@rpmforge)
                 Requires: libmodplug.so.0
                 Removing: libmodplug-0.8.7-1.el6.rf.i686 (@rpmforge)
                     libmodplug.so.0
                 Updated By: 1:libmodplug-0.8.8.3-2.el6.i686 (epel)
                     Not found
      Error: Package: vlc-1.1.13-1.el6.rf.i686 (@rpmforge)
                 Requires: libthreadutil.so.2
                 Removing: libupnp-1.6.6-1.el6.rf.i686 (@rpmforge)
                     libthreadutil.so.2
                 Updated By: libupnp-1.6.13-1.el6.i686 (epel)
                     Not found
      Error: Package: vlc-1.1.13-1.el6.rf.i686 (@rpmforge)
                 Requires: libmatroska.so.2
                 Removing: libmatroska-1.0.0-1.el6.rf.i686 (@rpmforge)
                     libmatroska.so.2
                 Updated By: libmatroska-1.2.0-1.el6.i686 (epel)
                     Not found
      Error: Package: vlc-1.1.13-1.el6.rf.i686 (@rpmforge)
                 Requires: libmodplug.so.0
                 Removing: libmodplug-0.8.7-1.el6.rf.i686 (@rpmforge)
                     libmodplug.so.0
                 Updated By: 1:libmodplug-0.8.8.3-2.el6.i686 (epel)
                     Not found
      Error: Package: vlc-1.1.13-1.el6.rf.i686 (@rpmforge)
                 Requires: libupnp.so.3
                 Removing: libupnp-1.6.6-1.el6.rf.i686 (@rpmforge)
                     libupnp.so.3
                 Updated By: libupnp-1.6.13-1.el6.i686 (epel)
                     Not found
      Error: Package: vlc-1.1.13-1.el6.rf.i686 (@rpmforge)
                 Requires: libebml.so.2
                 Removing: libebml-1.0.0-1.el6.rf.i686 (@rpmforge)
                     libebml.so.2
                 Updated By: libebml-1.2.1-1.el6.i686 (epel)
                     Not found

Problems with dependences is available.

After some reflections I found two ways to use VLC and to try to avoid the similar. First, it is possible to try to be played with preferencies of priorities of repositories and a ban or permission of installation of certain packages from this. But here once it was possible to face a similar problem with other packages.

The second option is a little another. Having used a resource [pkgs.org](http://pkgs.org/) I found VLC also in RPMFusion repository about which, such feeling, recently somehow forgot partially, though for this purpose there are some reasons. For example, an empty repository [rpmfusion](http://mirror.yandex.ru/fedora/rpmfusion/free/el/releases/6/Everything/x86_64/os/), and the packages being in [rpmfusion-updates](http://mirror.yandex.ru/fedora/rpmfusion/free/el/updates/6/x86_64/). But it is a subject of another story.

So I offer such way of a solution.

1. Remove VLC package. And remove all packages which cause the conflicts when updating.
2. Enable RPMFusion. For this purpose it is enough to install two packages: [one](http://mirror.yandex.ru/fedora/rpmfusion/free/el/updates/6/x86_64/rpmfusion-free-release-6-1.noarch.rpm) and [second](http://mirror.yandex.ru/fedora/rpmfusion/nonfree/el/updates/6/x86_64/rpmfusion-nonfree-release-6-1.noarch.rpm).
3. Install VLC. The package will be installed from RPMFusion because there the version is newer.

And after that all should work normally. Checked on itself.
