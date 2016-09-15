---
layout: post
title: Docker for Windows and Windows Containers
date: '2016-09-15T12:00:00.001+10:00'
author: Richard Banks
tags:
 - docker
modified_time: '2016-09-15T12:00:00.001+10:00'
---

Great news! The latest docker for windows application now supports switching between the linux and windows daemons!

This means you can run and manage Windows Containers and Linux containers from the same client on Windows 10 and Windows Server 2016.

If you want to try this out for yourself, you'll need the following:

- The [Docker for Windows](https://docs.docker.com/docker-for-windows/) app, version 1.12.1-beta26 or later.
- The Windows Container feature (install the container using the [Windows Containers Quick Start guide](https://msdn.microsoft.com/en-us/virtualization/windowscontainers/quick_start/quick_start_windows_10), but skip the daemon and client installation)

By default, the Docker client will connect to the Linux daemon, and show Linux based containers, as you can see here:

![docker connected to linux daemon](/assets/images/2016-09-15-linux-daemon.jpg)

To switch the client to use the Windows Containers daemon, select the option from the SysTray icon:

![switching the client](/assets/images/2016-09-15-switch-daemon.jpg)

It'll take a moment for the switch to complete, but when it does you'll then be able to see windows based containers, as you can see here

![docker connected to linux daemon](/assets/images/2016-09-15-windows-daemon.jpg)

Be aware that there's still some teething issues with switching, and you may see timeouts when switching to the Windows daemon, but it works and makes it much simpler for those of us using a mix of windows and linux containers.

Oh, before I forget and before you ask, the answer is yes. Yes, you can switch the docker client between the windows and linux daemons without affecting any running containers. Nice! 