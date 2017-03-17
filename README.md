
## Workshop:
# Building a 64bit Docker OS for the Raspberry Pi 3

In this "short" workshop I'm going through all the steps to build a complete 64bit operating system for the Raspberry Pi 3. This 64bit HypriotOS is mainly focused on running Docker containers easily on this popular DIY and IoT device.

You'll see that I'm using Docker heavily for each and every build step, because it really helps us a lot to run a local build with [Docker-for-Mac](https://docs.docker.com/docker-for-mac/) or later on the cloud build servers at [Travis CI](https://travis-ci.org).

![bee42-workshop.jpg](/images/bee42-workshop.jpg)

This workshop is sponsored by [bee24 solutions](http://bee42.com) <a href="https://twitter.com/bee42solutions" class="twitter-follow-button" data-show-count="false">Follow @bee42solutions</a><script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>


### Background and Contents

As you may already know, a complete Linux operating system consists of several important building blocks and I'd like to show you how to build all of these step by step.

For this purpose I'm not going to reinvent the wheel and so I'll reuse all of the existing parts which are already published by the Raspberry Pi Organisation, by the Debian Linux project and will add some more glue code.

At the end you should have learned how to build all the necessary parts and how to automatically create a finished SD card image which could be flashed on an empty SD card. Then boot your Raspberry Pi 3 with and in less than a minute you have a fully running Docker Engine on a Debian-based 64bit Linux OS.

Here is a short overview of the contents of the workshop:

* [Part 1: Bootloader](/part1-bootloader.md)

* [Part 2: Kernel](/part2-kernel.md)

* [Part 3: Root Filesystem](/part3-root-filesystem.md)

* [Part 4: SD Card Image](/part4-sd-card-image.md)

So, that's enough for explaining the background of this workshop. For more informations
please follow along the details I'll publish on the coming tweets at Twitter and in the
GitHub repos until all the parts of this workshop are finished and available as OpenSource.

Please be aware that this OS is in a development phase and far from being perfect.
So you are encouraged to help me with reporting issues and filing pull-requests to
add features and fix bugs.


### Raspberry Pi 3 Model B

**Specification**

![Raspi3-Image](https://upload.wikimedia.org/wikipedia/commons/e/e6/Raspberry-Pi-3-Flat-Top.jpg)

Even more technical specs can be found at [Wikipedia](https://en.wikipedia.org/wiki/Raspberry_Pi) or in this detailed [MagPi article](https://www.raspberrypi.org/magpi/raspberry-pi-3-specs-benchmarks/).

--

![bee42-logo.jpg](/images/bee42-logo.jpg)

--
MIT License

Copyright (c) 2017 Dieter Reuter
