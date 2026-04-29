---
title: Writing and Testing a Character Device Driver in Linux
date: 2026-04-28 20:00:00 -0300
categories: [Open Source Software Development, Linux Kernel]
tags: [linux, kernel, c, drivers, tutorial]
---

Continuing my journey with the Linux Kernel through the FLUSP tutorials, I recently tackled the development of a Character Device Driver. This tutorial was a fantastic hands-on experience in understanding how user-space applications communicate with kernel-space modules through file operations and device nodes. 

While the theory behind major/minor numbers and the `file_operations` struct is straightforward, executing the cross-compilation and deployment process on a virtualized ARM64 environment brought up some excellent learning opportunities. Here are the main roadblocks I faced and how I solved them.

### Roadblocks and Solutions

**1. Architecture Mismatch and Silent Kernel Panics**
When updating the kernel configuration with `make olddefconfig`, I initially forgot to explicitly pass the `ARCH=arm64` flag. This caused the build system to overwrite my carefully crafted QEMU configuration with a default `x86_64` one. As a result, the kernel compiled fine, but when booting the VM, it resulted in a silent black screen because it lacked the ARM64 VirtIO drivers. 
* **Solution:** I had to force-stop the VM (`virsh destroy`), regenerate the configuration properly using `make ARCH=arm64 CROSS_COMPILE=aarch64-linux-gnu- defconfig`, and trigger a full recompilation.

**2. Cross-Compilation Quirks with `kw deploy`**
After successfully compiling my `simple_char.ko` module, I attempted to send it to the VM using the `kworkflow` deployment tool (`kw deploy`). However, the process failed during the module preparation phase with an error from the `strip` utility. The host's native `strip` command was trying to optimize the module but couldn't recognize the `arm64` binary format.
* **Solution:** To bypass the host's default packaging tools, I opted for a manual deployment. I used `scp` to send the `.ko` file directly into the VM's root directory and loaded it into the kernel manually using `insmod simple_char.ko`.

**3. Volatile Device Nodes and Dynamic Major Numbers**
To test the driver, I created a device node using `mknod simple_char_node c <MAJOR> 0`. Since the driver uses `alloc_chrdev_region()`, the major number is dynamically assigned by the kernel. Everything worked perfectly until I rebooted the VM. Upon restart, my user-space test programs failed to interact with the module.
* **Solution:** I realized two things: first, the loaded module state lives in RAM and is lost on reboot; second, while my node file persisted on the disk, the kernel assigned a *different* major number to the module upon reloading it. I had to read `/proc/devices`, delete the stale node, and recreate it with the newly assigned major number to restore communication.

### Successful Attempt

Once these infrastructure and deployment quirks were ironed out, the actual driver logic worked flawlessly. Writing user-space C programs (`read_prog.c` and `write_prog.c`) and watching them successfully pass strings back and forth into the kernel's protected memory space was incredibly rewarding. It solidifies the Unix philosophy that "everything is a file" and provides a great foundation for diving into more complex subsystems like the Industrial I/O (IIO) framework next.