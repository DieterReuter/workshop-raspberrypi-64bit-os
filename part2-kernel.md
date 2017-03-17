
# Part 2: Kernel

## Using the Linux kernel

At the end of the boot process the bootloader will automatically detect the CPU model and tries to load and run a pre-defined Linux kernel. As the first Raspberry Pi uses an ARMv6 CPU and later on with the Raspberry Pi 2 an ARMv7 CPU, the bootloader can load both different kernels. Here are the default filenames we could use for these different kernel versions:

* **kernel.img** - for ARMv6 (Raspberry 1, Zero, Zero W)
* **kernel7.img** - for ARMv7 (Raspberry 2, 3 in 32bit mode)
* **kernel8.img** - for ARMv8,ARM64,AARCH64 (Raspberry 3 in 64bit mode)

As we're going right into a 64bit Linux system, we're using "kernel8.img" as our user code to be started directly through the bootloader. To be honest there are some more additional configuration files which will be needed to start and configure our Linux kernel in the right way.

* **kernel8.img** - the 64bit kernel for the Raspberry Pi 3
* **bcm2710-rpi-3-b.dtb** - device tree binary Linux kernel configuration
* **config.txt** - configuration file read by the bootloader
* **cmdline.txt** - kernel commandline parameters

Now we do have these minimal bootfiles which consists on the bootloader itself, the Linux kernel as user code and some configuration files. As soon as we put these on a SD card with FAT32 format, our Raspberry Pi 3 can be booted from and will show detailed kernel logs while booting on the serial UART console.
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

One important part to use the serial UART console is the configuration of `enable_uart=1` in "config.txt".
```
$ cat config.txt
# enable UART console on GPIO pins
enable_uart=1
```

And there are some more parameters to set in the kernel commandline like `console=tty1`, `console=ttyAMA0,115200` and `kgdboc=ttyAMA0,115200`.
```
$ cat cmdline.txt
+dwc_otg.lpm_enable=0 console=tty1 root=/dev/mmcblk0p2 rootfstype=ext4 cgroup_enable=cpuset cgroup_enable=memory swapaccount=1 elevator=deadline fsck.repair=yes rootwait console=ttyAMA0,115200 kgdboc=ttyAMA0,115200
```

Here are the kernel logs (seen on serial UART console) of a boot attempt with such a 64bit Linux kernel but without having a root filesystem available on the SD card.
```
[    0.000000] Booting Linux on physical CPU 0x0
[    0.000000] Linux version 4.9.13-bee42-v8 (root@1aca9db9a114) (gcc version 6.2.1 20161016 (Linaro GCC 6.2-2016.11) ) #1 SMP PREEMPT Wed Mar 1 17:38:31 UTC 2017
[    0.000000] Boot CPU: AArch64 Processor [410fd034]
[    0.000000] efi: Getting EFI parameters from FDT:
[    0.000000] efi: UEFI not found.
[    0.000000] cma: Reserved 8 MiB at 0x0000000007400000
[    0.000000] percpu: Embedded 22 pages/cpu @fffffffc07f7d000 s52632 r8192 d29288 u90112
[    0.000000] Detected VIPT I-cache on CPU0
[    0.000000] CPU features: enabling workaround for ARM erratum 845719
[    0.000000] Built 1 zonelists in Zone order, mobility grouping on.  Total pages: 32256
[    0.000000] Kernel command line: 8250.nr_uarts=1 bcm2708_fb.fbwidth=656 bcm2708_fb.fbheight=416 bcm2708_fb.fbswap=1 vc_mem.mem_base=0xec00000 vc_mem.mem_size=0x10000000  +dwc_otg.lpm_enable=0 console=tty1 root=/dev/mmcblk0p2 rootfstype=ext4 cgroup_enable=cpuset cgroup_enable=memory swapaccount=1 elevator=deadline fsck.repair=yes rootwait console=ttyS0,115200 kgdboc=ttyS0,115200
[    0.000000] PID hash table entries: 512 (order: 0, 4096 bytes)
[    0.000000] Dentry cache hash table entries: 16384 (order: 5, 131072 bytes)
[    0.000000] Inode-cache hash table entries: 8192 (order: 4, 65536 bytes)
[    0.000000] Memory: 92532K/131072K available (6588K kernel code, 758K rwdata, 2384K rodata, 2624K init, 830K bss, 30348K reserved, 8192K cma-reserved)
[    0.000000] Virtual kernel memory layout:
[    0.000000]     modules : 0xffffff8000000000 - 0xffffff8008000000   (   128 MB)
[    0.000000]     vmalloc : 0xffffff8008000000 - 0xffffffbebfff0000   (   250 GB)
[    0.000000]       .text : 0xffffff99ee680000 - 0xffffff99eecf0000   (  6592 KB)
[    0.000000]     .rodata : 0xffffff99eecf0000 - 0xffffff99eef50000   (  2432 KB)
[    0.000000]       .init : 0xffffff99eef50000 - 0xffffff99ef1e0000   (  2624 KB)
[    0.000000]       .data : 0xffffff99ef1e0000 - 0xffffff99ef29da00   (   759 KB)
[    0.000000]        .bss : 0xffffff99ef29da00 - 0xffffff99ef36d214   (   831 KB)
[    0.000000]     fixed   : 0xffffffbefe7fd000 - 0xffffffbefec00000   (  4108 KB)
[    0.000000]     PCI I/O : 0xffffffbefee00000 - 0xffffffbeffe00000   (    16 MB)
[    0.000000]     vmemmap : 0xffffffbf00000000 - 0xffffffc000000000   (     4 GB maximum)
[    0.000000]               0xffffffbff0000000 - 0xffffffbff0200000   (     2 MB actual)
[    0.000000]     memory  : 0xfffffffc00000000 - 0xfffffffc08000000   (   128 MB)
[    0.000000] SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=4, Nodes=1
[    0.000000] Preemptible hierarchical RCU implementation.
[    0.000000]  Build-time adjustment of leaf fanout to 64.
[    0.000000] NR_IRQS:64 nr_irqs:64 0
[    0.000000] Failed to get local register map. FIQ is disabled for cpus > 1
[    0.000000] arm_arch_timer: Architected cp15 timer(s) running at 19.20MHz (phys).
[    0.000000] clocksource: arch_sys_counter: mask: 0xffffffffffffff max_cycles: 0x46d987e47, max_idle_ns: 440795202767 ns
[    0.000005] sched_clock: 56 bits at 19MHz, resolution 52ns, wraps every 4398046511078ns
[    0.000229] Console: colour dummy device 80x25
[    0.001448] console [tty1] enabled
[    0.001494] Calibrating delay loop (skipped), value calculated using timer frequency.. 38.40 BogoMIPS (lpj=19200)
[    0.001557] pid_max: default: 32768 minimum: 301
[    0.001879] Mount-cache hash table entries: 512 (order: 0, 4096 bytes)
[    0.001936] Mountpoint-cache hash table entries: 512 (order: 0, 4096 bytes)
[    0.003015] ftrace: allocating 22746 entries in 89 pages
[    0.061760] ASID allocator initialised with 65536 entries
[    0.069681] EFI services will not be available.
[    0.078673] Detected VIPT I-cache on CPU1
[    0.078736] CPU1: Booted secondary processor [410fd034]
[    0.085780] Detected VIPT I-cache on CPU2
[    0.085825] CPU2: Booted secondary processor [410fd034]
[    0.092897] Detected VIPT I-cache on CPU3
[    0.092939] CPU3: Booted secondary processor [410fd034]
[    0.093043] Brought up 4 CPUs
[    0.093235] SMP: Total of 4 processors activated.
[    0.093274] CPU features: detected feature: 32-bit EL0 Support
[    0.093375] CPU: All CPU(s) started at EL2
[    0.093436] alternatives: patching kernel code
[    0.094492] devtmpfs: initialized
[    0.111145] DMI not present or invalid.
[    0.111535] clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 1911260446275000 ns
[    0.111608] futex hash table entries: 1024 (order: 5, 131072 bytes)
[    0.112284] pinctrl core: initialized pinctrl subsystem
[    0.113030] NET: Registered protocol family 16
[    0.124301] cpuidle: using governor menu
[    0.124751] vdso: 2 pages (1 code @ ffffff99eecf7000, 1 data @ ffffff99ef1e4000)
[    0.124819] hw-breakpoint: found 6 breakpoint and 4 watchpoint registers.
[    0.126863] DMA: preallocated 256 KiB pool for atomic allocations
[    0.126996] Serial: AMBA PL011 UART driver
[    0.131843] bcm2835-mbox 3f00b880.mailbox: mailbox enabled
[    0.132710] uart-pl011 3f201000.serial: could not find pctldev for node /soc/gpio@7e200000/uart0_pins, deferring probe
[    0.185768] HugeTLB registered 2 MB page size, pre-allocated 0 pages
[    0.186779] bcm2835-dma 3f007000.dma: DMA legacy API manager at ffffff800804b000, dmachans=0x1
[    0.189271] SCSI subsystem initialized
[    0.189495] usbcore: registered new interface driver usbfs
[    0.189625] usbcore: registered new interface driver hub
[    0.189792] usbcore: registered new device driver usb
[    0.190077] dmi: Firmware registration failed.
[    0.191243] raspberrypi-firmware soc:firmware: Attached to firmware from 2017-02-26 20:30
[    0.192990] clocksource: Switched to clocksource arch_sys_counter
[    0.268406] VFS: Disk quotas dquot_6.6.0
[    0.268537] VFS: Dquot-cache hash table entries: 512 (order 0, 4096 bytes)
[    0.268826] FS-Cache: Loaded
[    0.269259] CacheFiles: Loaded
[    0.284203] NET: Registered protocol family 2
[    0.285250] TCP established hash table entries: 1024 (order: 1, 8192 bytes)
[    0.285321] TCP bind hash table entries: 1024 (order: 2, 16384 bytes)
[    0.285390] TCP: Hash tables configured (established 1024 bind 1024)
[    0.285523] UDP hash table entries: 256 (order: 1, 8192 bytes)
[    0.285583] UDP-Lite hash table entries: 256 (order: 1, 8192 bytes)
[    0.285905] NET: Registered protocol family 1
[    0.286524] RPC: Registered named UNIX socket transport module.
[    0.286561] RPC: Registered udp transport module.
[    0.286592] RPC: Registered tcp transport module.
[    0.286622] RPC: Registered tcp NFSv4.1 backchannel transport module.
[    0.287909] hw perfevents: enabled with armv8_pmuv3 PMU driver, 7 counters available
[    0.290869] workingset: timestamp_bits=46 max_order=15 bucket_order=0
[    0.312215] FS-Cache: Netfs 'nfs' registered for caching
[    0.313373] NFS: Registering the id_resolver key type
[    0.313442] Key type id_resolver registered
[    0.313477] Key type id_legacy registered
[    0.316468] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 251)
[    0.316691] io scheduler noop registered
[    0.316729] io scheduler deadline registered (default)
[    0.316813] io scheduler cfq registered
[    0.322871] BCM2708FB: allocated DMA memory c7450000
[    0.322946] BCM2708FB: allocated DMA channel 0 @ ffffff800804b000
[    0.329911] Console: switching to colour frame buffer device 82x26
[    0.334635] Serial: 8250/16550 driver, 1 ports, IRQ sharing enabled
[    0.337232] console [ttyS0] disabled
[    0.338686] 3f215040.serial: ttyS0 at MMIO 0x0 (irq = 61, base_baud = 31224999) is a 16550
[    1.028053] console [ttyS0] enabled
[    1.033775] KGDB: Registered I/O driver kgdboc
[    1.045837] bcm2835-rng 3f104000.rng: hwrng registered
[    1.052882] Unable to detect cache hierarchy from DT for CPU 0
[    1.079034] brd: module loaded
[    1.096254] loop: module loaded
[    1.100870] Loading iSCSI transport class v2.0-870.
[    1.107931] usbcore: registered new interface driver smsc95xx
[    1.115233] dwc_otg: version 3.00a 10-AUG-2012 (platform bus)
[    1.348566] Core Release: 2.80a
[    1.353150] Setting default values for core params
[    1.359424] Finished setting default values for core params
[    1.566921] Using Buffer DMA mode
[    1.571695] Periodic Transfer Interrupt Enhancement - disabled
[    1.579074] Multiprocessor Interrupt Enhancement - disabled
[    1.586210] OTG VER PARAM: 0, OTG VER FLAG: 0
[    1.592098] Dedicated Tx FIFOs mode
[    1.597506] WARN::dwc_otg_hcd_init:1053: FIQ DMA bounce buffers: virt = 0x08111000 dma = 0xc7444000 len=9024
[    1.610400] FIQ FSM acceleration enabled for :
[    1.610400] Non-periodic Split Transactions
[    1.610400] Periodic Split Transactions
[    1.610400] High-Speed Isochronous Endpoints
[    1.610400] Interrupt/Control Split Transaction hack enabled
[    1.640038] WARN::hcd_init_fiq:486: MPHI regs_base at 0x080fa000
[    1.647629] dwc_otg 3f980000.usb: DWC OTG Controller
[    1.654128] dwc_otg 3f980000.usb: new USB bus registered, assigned bus number 1
[    1.663064] dwc_otg 3f980000.usb: irq 15, io mem 0x00000000
[    1.670198] Init: Port Power? op_state=1
[    1.675593] Init: Power Port (0)
[    1.680489] usb usb1: New USB device found, idVendor=1d6b, idProduct=0002
[    1.688830] usb usb1: New USB device strings: Mfr=3, Product=2, SerialNumber=1
[    1.697611] usb usb1: Product: DWC OTG Controller
[    1.703805] usb usb1: Manufacturer: Linux 4.9.13-bee42-v8 dwc_otg_hcd
[    1.711787] usb usb1: SerialNumber: 3f980000.usb
[    1.718827] hub 1-0:1.0: USB hub found
[    1.724060] hub 1-0:1.0: 1 port detected
[    1.730839] usbcore: registered new interface driver usb-storage
[    1.738653] mousedev: PS/2 mouse device common for all mice
[    1.746937] bcm2835-wdt 3f100000.watchdog: Broadcom BCM2835 watchdog timer
[    1.755655] bcm2835-cpufreq: min=600000 max=1200000
[    1.762551] sdhci: Secure Digital Host Controller Interface driver
[    1.770269] sdhci: Copyright(c) Pierre Ossman
[    1.776641] sdhost: log_buf @ ffffff800811b000 (c7447000)
[    1.830029] mmc0: sdhost-bcm2835 loaded - DMA enabled (>1)
[    1.859487] mmc-bcm2835 3f300000.mmc: mmc_debug:0 mmc_debug2:0
[    1.867014] mmc-bcm2835 3f300000.mmc: DMA channel allocated
[    1.898170] sdhci-pltfm: SDHCI platform and OF driver helper
[    1.906804] ledtrig-cpu: registered to indicate activity on CPUs
[    1.914506] hidraw: raw HID events driver (C) Jiri Kosina
[    1.921676] usbcore: registered new interface driver usbhid
[    1.928801] usbhid: USB HID core driver
[    1.934435] Initializing XFRM netlink socket
[    1.940234] NET: Registered protocol family 17
[    1.941315] Indeed it is in host mode hprt0 = 00021501
[    1.952802] mmc0: host does not support reading read-only switch, assuming write-enable
[    1.952957] Key type dns_resolver registered
[    1.963780] Registered cp15_barrier emulation handler
[    1.963807] Registered setend emulation handler
[    1.964921] registered taskstats version 1
[    1.972958] 3f201000.serial: ttyAMA0 at MMIO 0x3f201000 (irq = 72, base_baud = 0) is a PL011 rev2
[    1.973510] of_cfs_init
[    1.973712] of_cfs_init: OK
[    1.976953] mmc1: queuing unknown CIS tuple 0x80 (2 bytes)
[    1.983313] mmc1: queuing unknown CIS tuple 0x80 (3 bytes)
[    1.999764] mmc1: queuing unknown CIS tuple 0x80 (3 bytes)
[    2.028960] mmc1: queuing unknown CIS tuple 0x80 (7 bytes)
[    2.029346] Waiting for root device /dev/mmcblk0p2...
[    2.042281] mmc0: new high speed SDHC card at address aaaa
[    2.050102] mmcblk0: mmc0:aaaa SU16G 14.8 GiB
[    2.057653]  mmcblk0: p1
[    2.069660] random: fast init done
[    2.105028] usb 1-1: new high-speed USB device number 2 using dwc_otg
[    2.113023] Indeed it is in host mode hprt0 = 00001101
[    2.132302] VFS: Cannot open root device "mmcblk0p2" or unknown-block(179,2): error -6
[    2.142782] Please append a correct "root=" boot option; here are the available partitions:
[    2.153872] 0100            4096 ram0 [    2.157614]  (driver?)
[    2.161381] 0101            4096 ram1 [    2.165121]  (driver?)
[    2.168816] 0102            4096 ram2 [    2.172548]  (driver?)
[    2.176165] 0103            4096 ram3 [    2.179898]  (driver?)
[    2.183500] 0104            4096 ram4 [    2.187232]  (driver?)
[    2.190834] 0105            4096 ram5 [    2.194567]  (driver?)
[    2.198135] 0106            4096 ram6 [    2.201883]  (driver?)
[    2.205474] 0107            4096 ram7 [    2.209205]  (driver?)
[    2.212793] 0108            4096 ram8 [    2.216538]  (driver?)
[    2.220130] 0109            4096 ram9 [    2.223877]  (driver?)
[    2.227446] 010a            4096 ram10 [    2.231264]  (driver?)
[    2.234793] 010b            4096 ram11 [    2.238614]  (driver?)
[    2.242099] 010c            4096 ram12 [    2.245926]  (driver?)
[    2.249406] 010d            4096 ram13 [    2.253227]  (driver?)
[    2.256667] 010e            4096 ram14 [    2.260488]  (driver?)
[    2.263872] 010f            4096 ram15 [    2.267692]  (driver?)
[    2.271042] b300        15558144 mmcblk0 [    2.275044]  driver: mmcblk
[    2.278801]   b301           65536 mmcblk0p1 00000000-01[    2.284164]
[    2.286557] Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(179,2)
```

We can see the kernel is starting in 64bit mode using a `AArch64 Processor`
```
[    0.000000] Booting Linux on physical CPU 0x0
[    0.000000] Linux version 4.9.13-bee42-v8 (root@1aca9db9a114) (gcc version 6.2.1 20161016 (Linaro GCC 6.2-2016.11) ) #1 SMP PREEMPT Wed Mar 1 17:38:31 UTC 2017
[    0.000000] Boot CPU: AArch64 Processor [410fd034]
```

and is initializing all four available CPU cores
```
[    0.093043] Brought up 4 CPUs
[    0.093235] SMP: Total of 4 processors activated.
```

but finally it ends up in a kernel panic because of the missing root filesystem.
```
[    2.029346] Waiting for root device /dev/mmcblk0p2...
...
[    2.132302] VFS: Cannot open root device "mmcblk0p2" or unknown-block(179,2): error -6
...
[    2.286557] Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(179,2)
```

So, this was just for demonstrating the boot process until the Linux kernel will be loaded and started correctly.


## Building the Linux kernel

But as you already guessed we have to build this 64bit Linux kernel first. And this will take some time to be explained and to run the kernel build itself.

Luckily the Raspberry Pi Foundation provides a GitHub repo https://github.com/raspberrypi/linux with all the Linux kernel sources, and there is also a branch "rpi-4.9.y" which supports building a 64bit kernel. OK, this branch doesn't have a proper kernel configuration which is already optimized for Docker containers - but this part was just easy for me to fix.

So I've already prepared a GitHub repo https://github.com/dieterreuter/rpi64-kernel to create a true 64bit Linux kernel for the Raspberry Pi 3. And of course with a Docker-optimized kernel configuration, where I have enable absolutely all necessary and important kernel modules and settings which helps to run Linux containers in an optimal way.

The build process itself uses Docker Compose to build a Docker Image first with a complete build environment based upon Debian/Jessie. So you can see, this build always run in a well defined environment. It's running in an isolated Linux with Debian/Jessie inside of a Docker container. In this case we don't have to install anything (except Docker itself or on macOS Docker-for-Mac) on our host machine.

As you can see, I'm using the command `docker-compose build` to build the Docker Image and later on I'm issuing a single build run with the command `docker-compose run builder`. And as a build parameter I can define a specific kernel default configuration file with `DEFCONFIG=docker_rpi3_defconfig`. That's all.
```
$ cat build.sh
#!/bin/bash
set -e

export DEFCONFIG=docker_rpi3_defconfig
docker-compose build
docker-compose run builder
```

For easier usage I created a script "build.sh" with these commands, so we can just invoke a new build with a single command only.
```
$ ./build.sh
```

The build results can be found in a sub-folder `./builds/`, for each new build the script `build-kernel.sh` is creating a new build folder with an individual version based upon the date+time of the current build. So we can run subsequent builds and keep all the old build artefacts.
```
$ tree builds
builds
└── 20170303-152804
    ├── 4.9.13-bee42-v8.config
    ├── 4.9.13-bee42-v8.tar.gz
    ├── 4.9.13-bee42-v8.tar.gz.sha256
    ├── bootfiles.tar.gz
    └── bootfiles.tar.gz.sha256

1 directory, 5 files
```
```
$ ls -al ./builds/20170303-152804/
-rw-r--r--  1 dieter  staff    140641 Mar  3 16:28 4.9.13-bee42-v8.config
-rw-r--r--  1 dieter  staff  21863866 Mar  3 16:40 4.9.13-bee42-v8.tar.gz
-rw-r--r--  1 dieter  staff        89 Mar  3 16:40 4.9.13-bee42-v8.tar.gz.sha256
-rw-r--r--  1 dieter  staff   5092332 Mar  3 16:40 bootfiles.tar.gz
-rw-r--r--  1 dieter  staff        83 Mar  3 16:40 bootfiles.tar.gz.sha256
```

From the build script `build-kernel.sh` we'll get a couple of build artefacts. In "bootfiles.tar.gz" are all the files which have to be placed in the "/boot" folder on the FAT32 partition. These are namely the 64bit Linux kernel "kernel8.img", the device tree binary Linux kernel configuration file "bcm2710-rpi-3-b.dtb" and the kernel overlay files in a folder called "overlays/".
```
$ tar vtf builds/20170303-152804/bootfiles.tar.gz
-rw-r--r--  0 root   root 12704256 Mar  3 16:40 ./kernel8.img
drwxr-xr-x  0 root   root        0 Mar  3 16:40 ./overlays/
...
-rw-r--r--  0 root   root    17714 Mar  3 16:40 ./bcm2710-rpi-3-b.dtb
```

But more important is the file "4.9.13-bee42-v8.tar.gz". Here you'll find all kernel files we really need to install on "/boot" and in the root filesystem like "/lib/firmware" and the kernel modules "/lib/modules/4.9.13-bee42-v8/".
```
$ tar vtf builds/20170303-152804/4.9.13-bee42-v8.tar.gz
drwxr-xr-x  0 root   root        0 Mar  3 16:40 ./lib/modules/4.9.13-bee42-v8/
...
drwxr-xr-x  0 root   root        0 Mar  3 16:40 ./lib/firmware/
...
drwxr-xr-x  0 root   root        0 Mar  3 16:40 ./boot/
-rw-r--r--  0 root   root 12704256 Mar  3 16:40 ./boot/kernel8.img
drwxr-xr-x  0 root   root        0 Mar  3 16:40 ./boot/overlays/
...
-rw-r--r--  0 root   root    17714 Mar  3 16:40 ./boot/bcm2710-rpi-3-b.dtb
```

And if you're interested in the generated and used kernel configuration file, this can be found in "4.9.13-bee42-v8.config" as a build artefact.


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

With the help of an extra Travis build script `travis-build.sh` I'm now able to upload the build artefacts of every successful build as a new GitHub release at https://github.com/DieterReuter/rpi64-kernel/releases. This makes it very easy to consume these build artefacts later in another build process and I do have all the complete and detailed build logs available for future reference.


## Recap

First we discussed how the Linux kernel will be used in the boot process and which necessary files we need to install on the "/boot" section of the FAT32 partiton on the SD card. Then we were able to boot a 64bit Linux kernel successfully on a Raspberry Pi 3 and seen the kernel boot logs appear on the serial UART console.

With the help of a newly created GitHub repo we're now able to compile and package all necessary Linux kernel files in tarballs, so we could copy them over onto a SD card.

From the boot attempt we know that the Linux kernel ends up in a "kernel panic", because we didn't provide a valid root filesystem on our SD card. So for the next part we'll covering how to create and use such a root filesystem
in [Part 3 - Root filesystem](/part3-root-filesystem.md).

--

![bee42-logo.jpg](/images/bee42-logo.jpg)

--
MIT License

Copyright (c) 2017 Dieter Reuter
