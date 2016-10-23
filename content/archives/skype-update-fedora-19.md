+++
title = "Skype failed after update. Fedora 19"
date = "2013-05-15"
tags = ["skype", "fedora"]
type = "post"
+++

After yesterday's updates Skype stopped working giving an error.

	Segmentation fault (core dumped)

I have tried to find out the cause of error by strace but since I have used this utility for the first time I couldn't immediately understand the causes malfunction of the program. Roll back the version of a package using `yum downgrade` failed because the previous version in the repository was gone.

So I have decided to just delete that version and then install the previous version from the Fedora 18 repo. And then in the process of removing the package I have got the following error:

	warning: file /usr/lib/libtiff.so.4: remove failed: No such file or directory

In the end, it turned out to solve the problem is very simple. It took only a necessary symbolic link 

	ln -s /usr/lib/libtiff.so.5.2.0 /usr/lib/libtiff.so.4

End.


