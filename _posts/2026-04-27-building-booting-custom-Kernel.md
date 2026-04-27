---
title: Building and booting a custom Linux kernel for ARM
date: 2026-04-27 15:30:00 -0300
categories: [Open Source Software Development, Linux Kernel]
tags: [linux, kernel, arm, cross-compilation, kworkflow]
---

Following our sequence of setup posts for the Linux Kernel contribution environment, this post covers the next step: compiling the Linux Kernel from source and booting it inside the ARM64 virtual machine we previously set up. 

For this task, we cloned the IIO (Industrial I/O) subsystem tree and used `kworkflow` (`kw`) to manage the cross-compilation process from an x86_64 host to our ARM target. While the process is mostly streamlined, I encountered a few interesting technical roadblocks along the way.

### Roadblocks and Solutions

**1. The Missing `rsync` Package in the Target VM**
To optimize the build process and avoid compiling unnecessary modules, the tutorial suggests using `make localmodconfig` based on a list of loaded modules (`vm_mod_list`) generated inside the VM. However, when I ran `kw ssh --get '~/vm_mod_list'` to fetch this file, the connection dropped with a `bash: rsync: command not found` error. The minimal Debian image we used for the VM doesn't come with `rsync` installed by default, and `kworkflow` relies on it under the hood for file transfers. The fix was simple: SSH into the VM manually and run `apt update && apt install -y rsync`.

**2. Kworkflow Cache Desync with `.config`**
To identify our custom kernel, we used `make nconfig` to append a custom string to the local version and disabled the automatic version appendage. After saving, checking the status with `kw build --info` still displayed the old auto-generated git hash instead of the new custom name. This happens because `kworkflow` caches the configuration for performance. As long as the actual `.config` file was updated correctly via the native `make` command, the kernel build system will use the right name during compilation. You can either clear the `kw` cache or just trust the `.config` file and proceed.

**3. SSH "No route to host" Immediately After VM Creation**
After successfully building the kernel and editing the `activate.sh` script to point to the new `Image` file, I recreated the VM and immediately ran `kw deploy` to install the modules. The command failed with multiple `No route to host` SSH errors. This was just a classic timing issue: destroying and recreating the VM means it takes a few seconds to boot up the OS and start the SSH daemon. Furthermore, the DHCP pool might assign it a new IP address. Waiting a moment, checking the new IP with `sudo virsh net-dhcp-leases default`, and updating the `kw remote` configurations solved the issue.

### Successful Attempt

Thanks to the `localmodconfig` optimization, the cross-compilation using `gcc-aarch64-linux-gnu` was surprisingly fast on my host machine. After resolving the SSH timing issue, the `kw deploy` command successfully transferred the modules to the VM. 

I logged into the virtual machine one last time, ran `uname --kernel-release`, and there it was: my custom kernel version running smoothly. The safe testing environment is now fully operational and ready for actual kernel code contributions!