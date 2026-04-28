---
title: Linux kernel build configuration and modules
date: 2026-04-27 19:00:00 -0300
categories: [Open Source Software Development, Linux Kernel]
tags: [linux, kernel, c, tutorial, flusp]
---

Following up on my journey with Linux Kernel development, I recently tackled the FLUSP tutorial on how to make new kernel modules, create build configurations, and understand module dependencies. While the C code itself was straightforward, interacting with the build system (`Kconfig` and `Makefile`) and managing the virtual machine state brought up some interesting challenges.

Here is a summary of the main roadblocks I encountered and how I solved them.

### Roadblocks and Solutions

**1. The `libguestfs` Appliance Crash (QCOW2 Locking)**
Early in the tutorial, we needed to mount the VM's disk image to inject our compiled `simple_mod.ko` module. I used `sudo guestmount --rw`, but it immediately crashed with an "appliance closed the connection unexpectedly" error. The issue? My virtual machine was still running in the background. QEMU strictly locks the `.qcow2` image while the VM is active to prevent corruption.
*Solution:* I had to completely power off the VM (`poweroff`) before mounting the filesystem. Later in the tutorial, I learned a much better approach: keeping the VM running and sending the compiled modules over the network using `kw ssh --send`.

**2. Module Dependencies and the `insmod` Trap**
When we created the second module (`simple_mod_part`) that called an exported function from the first one (`simple_mod`), I realized that simply throwing `.ko` files into the kernel memory isn't always enough. If you try to load a dependent module with `insmod`, it fails because it doesn't know how to resolve the exported symbols.
*Solution:* I learned to properly install the modules into `/lib/modules/$(uname -r)/kernel/drivers/misc/` inside the VM, run `depmod --quick` to update the dependency tree, and finally use `modprobe`. `modprobe` is smart enough to automatically load the parent module (`simple_mod`) right before loading the child module, resolving all symbol dependencies gracefully.

### Conclusion

This tutorial was incredibly valuable for understanding the Linux kernel build system. Learning how `Kconfig` links to the `Makefile`, how symbols are exported across the kernel, and how tools like `kworkflow` (`kw`) can drastically speed up the development cycle (like sending patches directly via SSH) gave me a lot more confidence to start writing more complex drivers.