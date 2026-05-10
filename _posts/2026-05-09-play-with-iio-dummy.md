---
layout: post
title: "IIO Dummy module Experiment One: Play with iio_dummy "
date: 2026-04-29 10:00:00 -0300
categories: [Linux Kernel, FLUSP]
tags: [IIO, Drivers, ARM64, Cross-Compilation, c]
---

As part of my journey in the Free Software Extension Group (FLUSP) at IME-USP, I recently tackled Tutorial 8, which focuses on the Industrial I/O (IIO) subsystem. The goal was to configure, compile, and interact with the `iio_dummy` driver. While the tutorial seems straightforward, the transition from theory to a functional virtual environment often presents non-trivial challenges.

### The Setup

The experiment was conducted on an Ubuntu host, aiming for an ARM64 virtual machine managed by `libvirt` and `kworkflow`. This architecture mismatch is where most of the learning — and the troubleshooting — happened.

### Challenges Encountered

#### 1. Cross-Compilation Hurdles
The first major obstacle was the architecture difference. My physical machine (x86_64) uses a different instruction set than the VM (ARM64). Simply running `make` results in compiler warnings regarding mismatched architectures.
* **Solution:** I had to explicitly tell the build system to use the cross-compiler. By setting `ARCH=arm64` and `CROSS_COMPILE=aarch64-linux-gnu-`, I ensured the modules were built correctly for the target environment.

#### 2. Symbol Resolution and MODPOST Errors
When compiling only the dummy driver directory using `M=drivers/iio/dummy`, I encountered `undefined!` symbol errors during the `MODPOST` stage. This happened because the dummy driver depends on other IIO components that hadn't been compiled yet.
* **Solution:** Instead of a partial build, I performed a full module compilation (`make modules`). This ensured all dependencies were built and symbols were properly exported, allowing the `MODPOST` stage to link everything correctly.

#### 3. Kernel Version Mismatch
After transferring the modules, I faced `Unknown symbol` errors when trying to load them with `insmod`. This is a classic issue where the modules are "newer" than the kernel currently running in the VM.
* **Solution:** I had to ensure the VM was booting from the exact `Image` file I had just compiled. I used `virsh edit arm64` to update the boot path in the VM configuration to point to my local kernel tree: `/home/lk_dev/iio/arch/arm64/boot/Image`.

#### 4. The Disappearing Device
A common point of confusion is when the `/sys/bus/iio/devices/iio:device0/` directory disappears. In my case, this happened after unloading the module to apply code changes.
* **Solution:** Since the IIO dummy device is a software-instantiated device, it must be recreated every time the module is reloaded. Using `configfs` via `mkdir` in the proper `/mnt/iio_experiments/` directory brought the device back to life.

### Conclusion

This tutorial was a great lesson in environment synchronization. Developing for the kernel is as much about managing your build tree and VM configuration as it is about writing C code. Successfully reading the raw data from `in_magn_x_raw` after these fixes was a rewarding conclusion to the experiment.

