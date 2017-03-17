
# Part 3: Root Filesystem

The root filesystem of a Linux operating system contains all the files you need to run Linux. It is typically placed on an extra partition and should generally be pretty small to keep the system fast and secure.

At the [Hypriot project](https://blog.hypriot.com) we based HypriotOS on the Linux distro Debian/Jessie and we already built a complete set of root filesystems for different CPU architectures. One of the basic goals of HypriotOS is to keep the Linux system itself as small as possible - so we just added the basic packages to the rootfs and did some basic customization.


## Building a Root filesystem

Normally one would use the Debian tool "debootstrap" to create a new root filesystem for Debian/Jessie and there are some detailed documentation about [Debootstrap](https://wiki.debian.org/Debootstrap). It will take some time to get familiar with this tools and after some time you can create your own customized rootfs from scratch as well.

At Hypriot we did this work already and there is a complete GitHub repo to build all these root filesystems at https://github.com/hypriot/os-rootfs. We also provide ready-to-download tarballs at the [GitHub releases](https://github.com/hypriot/os-rootfs/releases) of this project.

And luckily we do have a ready tarball for ARMv8/ARM64/AARCH64 for Debian/Jessie, so it's just easy to take this one for our workshop to build a 64bit Docker-enabled Linux operating system for the Raspberry Pi 3.


## Contents of a Root Filesystem

So lets download the latest version of this rootfs of HypriotOS and analyze it.
```
$ wget https://github.com/hypriot/os-rootfs/releases/download/v1.1.1/rootfs-arm64-debian-v1.1.1.tar.gz
```

The first thing we could see, it is really small in size with 116 MBytes only.
```
$ ls -alh rootfs-arm64-debian-v1.1.1.tar.gz
-rw-r--r--  1 dieter  staff   116M Feb 27 17:05 rootfs-arm64-debian-v1.1.1.tar.gz
```

But yes, believe me, there are all necessary Debian packages included to boot into a full Linux system.
```
$ tar vtf rootfs-arm64-debian-v1.1.1.tar.gz | head -20
drwxr-xr-x  0 root   root        0 Feb 27 17:03 ./
drwxr-xr-x  0 root   root        0 Feb 27 17:04 ./sbin/
lrwxrwxrwx  0 root   root        0 Nov  8  2014 ./sbin/ip6tables -> xtables-multi
-rwxr-xr-x  0 root   root    14480 Nov  8  2014 ./sbin/ipmaddr
lrwxrwxrwx  0 root   root        0 Nov  8  2014 ./sbin/iptables-save -> xtables-multi
-rwxr-xr-x  0 root   root    74384 May 28  2016 ./sbin/dmsetup
-rwxr-xr-x  0 root   root      885 Feb 24 09:09 ./sbin/shadowconfig
lrwxrwxrwx  0 root   root        0 Sep 27  2014 ./sbin/modinfo -> /bin/kmod
-rwxr-xr-x  0 root   root    14504 Nov 13 16:15 ./sbin/pam_tally2
-rwxr-xr-x  0 root   root    10328 Mar 30  2015 ./sbin/fsfreeze
-rwxr-xr-x  0 root   root   102096 Dec 31 22:14 ./sbin/mke2fs
-rwxr-xr-x  0 root   root     6080 Apr  6  2015 ./sbin/fstab-decode
lrwxrwxrwx  0 root   root        0 Sep 27  2014 ./sbin/insmod -> /bin/kmod
-rwxr-xr-x  0 root   root   233968 Mar 30  2015 ./sbin/fdisk
-rwxr-xr-x  0 root   root    73088 Dec 31 22:14 ./sbin/tune2fs
lrwxrwxrwx  0 root   root        0 Jan  7 04:35 ./sbin/init -> /lib/systemd/systemd
lrwxrwxrwx  0 root   root        0 Sep 27  2014 ./sbin/rmmod -> /bin/kmod
lrwxrwxrwx  0 root   root        0 Dec 31 22:14 ./sbin/mkfs.ext2 -> mke2fs
lrwxrwxrwx  0 root   root        0 Sep 27  2014 ./sbin/depmod -> /bin/kmod
-rwxr-xr-x  0 root   root    27040 Mar 30  2015 ./sbin/blockdev
```


## Recap

This part of the workshop was really easy because we already have a ready-to-use tarball with the latest version of a pre-configured Debian/Jessie-based HypriotOS for our 64bit Raspberry Pi 3.

Lets sum up what components we do have now:

1. Bootloader - specific for the Raspberry Pi
2. Kernel - a Linux kernel in 64bit with the current LTS version 4.9.13 compiled and configured for the Raspberry Pi 3
3. Root Filesystem - from HypriotOS based on Debian/Jessie for AARCH64

So, these are finally all the components we need to build our complete operating system. The final construction will be covered in [Part 4 - SD Card Image](/part4-sd-card-image.md) where we'll create a bootable SD card and assemble all of the components we've built so far.

--

![bee42-logo.jpg](/images/bee42-logo.jpg)

--
MIT License

Copyright (c) 2017 Dieter Reuter
