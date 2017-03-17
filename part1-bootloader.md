
# Part 1: Bootloader

In order to know which essential files are needed to boot the Raspberry Pi from a SD card we have to dig a little bit deeper into the boot process itself. The boot process is almost the same for each and every Raspberry Pi model, so let's read this as a basic primer.


## Raspberry Pi Boot Process

Booting a Raspberry Pi is a little bit different because the CPU is offline from the beginning and the SoC (system on a chip) starts the GPU first which initializes the important system components and controls or better starts the low level boot process.

* **First Stage Bootloader** -
The first stage bootloader is located in the ROM section of the SoC, so it is baked-in into the hardware chip. It mounts the FAT32 boot section/partition of the SD card in order to have access to load the necessary files like the second stage bootloader.

* **Second Stage Bootloader** -
The second stage bootloader will be loaded from the file "bootcode.bin" which contains the GPU code to start the GPU and loading the GPU firmware. So you can see that "bootcode.bin" contains just GPU specific code and will be provided as a closed-source binary blob.

* **GPU Firmware** -
The GPU firmware itself is loaded from the file "start.elf" and enables the GPU to load and run ARM specific user code for the CPU like our Linux kernel which we need to start our Linux operating system.

* **User Code** -
The user code will be run on the CPU and is therefore compiled for the specific ARM processor. Normally this is the Linux kernel which is compiled for ARMv6 or ARMv7 architecture for running Raspbian, but we're going to use a Linux kernel for ARMv8 (or ARM64) which is also known as the AARCH64 architecture.

I'm just touching the basic parts here and if you're interested in all the details you can find the boot process explained in more detail at http://elinux.org/RPi_Software or read the latest documentation about the [Raspberry Pi Boot Modes](https://www.raspberrypi.org/documentation/hardware/raspberrypi/bootmodes/).


## Minimum Boot Files

As we learned about the boot process, we do have only a few requirements to successfully boot a Raspberry Pi from a SD card:

* **FAT32 filesystem** -
will be mounted later as "/boot" in Linux
* **File "bootcode.bin"** - Second Stage Bootloader
* **File "start.elf"** - GPU Firmware
* **File "fixup.dat"** - (optional) used to configure the SDRAM partition between GPU and CPU

And next the user code like the Linux kernel to bring our system to life.

> These are really the minimum required bootfiles to boot a Raspberry Pi from SD card:

>     "bootcode.bin", "start.elf", "fixup.dat", "kernel.img"

To be honest, we do need a few more files for starting and configuring the Linux kernel and to access the serial UART console. So I'm listing here the bare minimum bootfiles to really boot into our Linux kernel and so we can see the kernel logs appear on the serial UART console.
```
$ ls -al
total 15284
drwxr-xr-x  2 root root    16384 Mar  3 11:12 .
drwxr-xr-x 21 root root     4096 Jul  3  2014 ..
-rwxr-xr-x  1 root root    17253 Mar  1 17:42 bcm2710-rpi-3-b.dtb
-rwxr-xr-x  1 root root    50268 Mar  1 16:58 bootcode.bin
-rwxr-xr-x  1 root root      215 Mar  1 18:14 cmdline.txt
-rwxr-xr-x  1 root root       50 Mar  3 11:12 config.txt
-rwxr-xr-x  1 root root     6640 Mar  1 16:58 fixup.dat
-rwxr-xr-x  1 root root 12704256 Mar  1 17:42 kernel8.img
-rwxr-xr-x  1 root root  2841412 Mar  1 16:58 start.elf
```


## Where to get the bootloader files?

As I already mentioned before, these bootloader files are provided as closed-source binary blobs directly from the Raspberry Pi Foundation and can be downloaded from the "boot" section/folder of the Raspberry Pi Firmware repo [raspberrypi/firmware on GitHub](https://github.com/raspberrypi/firmware/tree/master/boot).


## Let's build a bootloader package

Now it's time to build our first component, the "rpi-bootloader" which contains just the bootfiles for the Raspberry Pi. And for easy use we're just going to create a tarball with the essential bootfiles.
> As these bootfiles are not specific to a Raspberry Pi 3 only, we could possibly reuse this package later for all the other Raspberry Pi models as well.

I've already prepared a GitHub repo to create a tarball https://github.com/dieterreuter/rpi-bootloader.


### How the build process works

The build process uses Docker Compose to build a Docker Image first with a complete build environment based upon Debian/Jessie. So you can see, this build always run in a well defined environment. It's running in an isolated Linux with Debian/Jessie inside of a Docker container. In this case we don't have to install anything (except Docker itself or on macOS Docker-for-Mac) on our host machine.

As you can see, I'm using the command `docker-compose build` to build the Docker Image and later on I'm issuing a single build run with the command `docker-compose run builder`. That's all.
```
$ cat build.sh
#!/bin/bash
set -e

docker-compose build
docker-compose run builder
```

For easier usage I created a script "build.sh" with these two commands, so we can just invoke a new build with a single command only.
```
$ ./build.sh
```

The build results can be found in a sub-folder `./builds/`, for each new build the script `build-tarball.sh` is creating a new build folder with an individual version based upon the date+time of the current build. So we can run subsequent builds and keep all the old build artefacts.
```
$ tree builds/
builds/
└── 20170303-121340
    ├── rpi-bootloader.tar.gz
    └── rpi-bootloader.tar.gz.sha256

1 directory, 2 files
```


### Create a Travis CI build job

Most of the times it is easier and more reliable to use a CI (continous integration) system to run the build process on a cloud server, so I'm going to use [Travis CI](https://travis-ci.org).

I just created the necessary Travis configuration file `.travis.yml` with some special settings.
```
$ cat .travis.yml
sudo: required
services:
  - docker
language: bash
script:
  - ./travis-build.sh
after_success:
  - ls -al builds/$BUILD_NR/*
branches:
  only:
    - master
  except:
    - /^v\d.*$/
```

With the help of an extra Travis build script `travis-build.sh` I'm now able to upload the build artefacts of every successful build as a new GitHub release at https://github.com/DieterReuter/rpi-bootloader/releases. This makes it very easy to consume these build artefacts later in another build process and I do have all the complete and detailed build logs available for future reference.


## Recap

We talked about the Raspberry Pi boot process in detail and have shown the minimum required bootfiles.

Then we've built a GitHub repo with a complete automated build process to fetch all files from the original Raspberry Pi Firmware repo and then building a versioned tarball. We can run the build process for development locally on macOS with the help of [Docker-for-Mac](https://docs.docker.com/docker-for-mac/) and later on the cloud build servers at [Travis CI](https://travis-ci.org).

At the end we can just use the produced tarball "rpi-bootloader.tar.gz" in order to copy all the bootfiles to a SD card to get the latest version.

```
$ tar vtf builds/20170303-121340/rpi-bootloader.tar.gz
-rw-r--r--  0 root   root     1494 Mar  3 13:14 boot/LICENCE.broadcom
-rw-r--r--  0 root   root    50268 Mar  3 13:14 boot/bootcode.bin
-rw-r--r--  0 root   root     6646 Mar  3 13:14 boot/fixup.dat
-rw-r--r--  0 root   root     2558 Mar  3 13:14 boot/fixup_cd.dat
-rw-r--r--  0 root   root     9777 Mar  3 13:14 boot/fixup_db.dat
-rw-r--r--  0 root   root     9775 Mar  3 13:14 boot/fixup_x.dat
-rw-r--r--  0 root   root  2846692 Mar  3 13:14 boot/start.elf
-rw-r--r--  0 root   root   654980 Mar  3 13:14 boot/start_cd.elf
-rw-r--r--  0 root   root  4983140 Mar  3 13:14 boot/start_db.elf
-rw-r--r--  0 root   root  3929604 Mar  3 13:14 boot/start_x.elf
```

To create a bootable SD card we now have to format a SD card with FAT32 and have to copy all the files from the "/boot" folder into the root directory of the SD card. But as we don't have any meaningful user code embedded yet, like "kernel8.img", we won't see anything on an attached HDMI monitor nor on the serial UART console. This will be covered in detail in [Part 2 - Kernel](/part2-kernel.md).

--

![bee42-logo.jpg](/images/bee42-logo.jpg)

--
MIT License

Copyright (c) 2017 Dieter Reuter
