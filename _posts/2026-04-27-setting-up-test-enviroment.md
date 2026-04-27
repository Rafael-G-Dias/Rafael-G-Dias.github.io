---
title: Setting up a test environment for Linux Kernel
date: 2026-04-27 14:00:00 -0300
categories: [Open Source Software Development, Linux Kernel]
tags: [linux, kernel, c, tutorial]
---

As a first step into open-source software development and my journey with the Linux Kernel, I recently followed a FLUSP tutorial to set up a test environment using `qemu` and `libvirt`. Configuring a virtual machine from scratch to compile and test kernel modules is a great learning experience. Here is a summary of my setup process, the main roadblocks I faced, and how I solved them.

### Roadblocks and Solutions

**1. Minimal Image Lacking SSH Server**
When configuring SSH access, I noticed the `/etc/ssh` directory didn't exist inside the VM. It turns out the minimal Debian `nocloud` image I was using was extremely stripped down and didn't come with an SSH server pre-installed. I had to access the VM directly via `virsh console` and run `apt update && apt install openssh-server procps -y` before continuing the tutorial.

**2. The Brazilian Keyboard Trap in `virsh console`**
To detach from a running VM console without shutting it down, the default shortcut is `Ctrl + ]`. However, because I use a Brazilian ABNT2 keyboard layout, the terminal emulator didn't properly map the keystroke, leaving me stuck in the VM's screen. The quick workaround was opening a new terminal tab on the host and running `sudo pkill virsh` to forcefully close the console viewer.

**3. Missing `virtiofsd` Daemon**
The final step was setting up a shared directory between the host and the VM using `virtiofs`. After editing the VM's XML configuration with `virsh edit`, the VM refused to start, throwing an `Unable to find a satisfying virtiofsd` error. The issue was simply that my host machine was missing the background daemon required to bridge the file system. Running `sudo apt install virtiofsd` on the host resolved it immediately.

### Successful Attempt

Solving these infrastructure issues provided a deeper understanding of how `libvirt` and `qemu` handle networking and file systems behind the scenes. With the daemon installed and the SSH server running, I successfully started the `arm64` VM and mounted the shared folder. 

The environment is now fully active, with files syncing seamlessly between my host and the VM, leaving me ready to start compiling modules!