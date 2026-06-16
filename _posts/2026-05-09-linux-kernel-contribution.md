---
layout: post
title: "Cleaning up includes in the IIO Subsystem: My First Linux Kernel Contribution"
date: 2026-04-30 10:00:00 -0300
categories: [Linux Kernel, FLUSP]
tags: [IIO, Drivers, linux, kernel-linux, c, IWYU]
---

Alongside my colleague Felipe Khoury Dayoub, I have just had my first contribution accepted into the Linux Kernel. This was done within the Industrial I/O (IIO) subsystem, specifically targeting the `stk3310` ambient light and proximity sensor driver. 

This contribution was part of our coursework as undergraduate students at the Institute of Mathematics and Statistics of the University of São Paulo (IME-USP). It quickly taught me that contributing to a massive open-source project is as much about understanding community workflows and software architecture as it is about writing code.

### The Goal 

Our mission was to refactor the included headers of the `drivers/iio/light/stk3310.c` file. Historically, many older drivers relied on catch-all headers like `<linux/kernel.h>` or `<linux/device.h>`, which hurts compilation times and tangles the dependency tree. The modern kernel standard is strict: include exactly what you use. To map this, we used the Include-What-You-Use (IWYU) tool.


### The Architecture Lesson [V1]

The Linux Kernel community is incredibly responsive. Shortly after sending V1, the automatism ended, and we received a masterclass in architecture from reviewer Joshua Crofts and the IIO subsystem maintainer, Jonathan Cameron.

**1. IWYU generates noise:**
While IWYU is a fantastic tool, it is just a machine. It told us to include headers like `<linux/kconfig.h>` and `<linux/stddef.h>`, which the reviewer immediately asked us to remove because they were redundant. 

**2. The `<linux/device.h>` debate:**
The most interesting discussion was regarding the device header. Jonathan intervened to explain that `<linux/device.h>` is becoming a new "catch-all", and the community is actively trying to stop including it directly. His guideline was clear: try to compile without it. If the compiler breaks because it doesn't recognize pointers to the device structure, don't include the entire library—use a forward declaration (`struct device;`) at the top of the file instead. We ran `make`, and it compiled perfectly in silence. The maintainer was right.

### Structural Feedback: The V2 Patchset

Besides filtering the noise, we received a structural critique. Our V1 modified and organized everything in a single go. The kernel community does not accept this; they demand **Atomic Commits**.

We were asked to split our work into a patchset. To understand the impact, here is how the includes looked **originally** in the driver:

```c
/* 1. The Original State */
#include <linux/i2c.h>
#include <linux/interrupt.h>
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/mod_devicetable.h>
#include <linux/regmap.h>

#include <linux/iio/events.h>
#include <linux/iio/iio.h>
#include <linux/iio/sysfs.h>
```
To achieve atomicity, we divided our changes:
* **Patch 1/2 (Visual Style):** This patch couldn't add or remove dependencies. It exclusively handled grouping the headers logically. We separated the generic `<linux/*>` headers from the subsystem-specific `<linux/iio/*>` headers with a blank line, keeping both blocks strictly alphabetized. Notice how module.h and mod_devicetable.h swap places below:

```c
/* 2. Patch 1/2: Sorting and Grouping */
#include <linux/i2c.h>
#include <linux/interrupt.h>
#include <linux/kernel.h>
#include <linux/mod_devicetable.h>
#include <linux/module.h>
#include <linux/regmap.h>

#include <linux/iio/events.h>
#include <linux/iio/iio.h>
#include <linux/iio/sysfs.h>
```

* **Patch 2/2 (The Cleanup):** Only in the second patch did we apply the actual cleanup, removing the generic headers and adding the explicit, noise-free dependencies.

### The Final Tweaks and a Rite of Passage
With V2 divided and lean, we resent the code. Jonathan Cameron approved it and applied it to the testing branch, but the final code still went through some meticulous late-stage refinements directly on the mailing list.

First, Jonathan made two manual tweaks: he added `<asm/byteorder.h>` back because he noticed our code used endianness conversions, and he opted to keep `<linux/sysfs.h>` explicitly (subtly disagreeing with the first reviewer) for the attribute_group struct.

Then, a third highly active maintainer, Andy Shevchenko, saw our thread and suggested we drop two more libraries (`irqreturn.h` and `errno.h`) since they were already implied by other included headers. Jonathan agreed, cleaned it up, and did the final merge.

Here is the Final Merged State, reflecting our Patch 2 combined with the maintainers' final architectural decisions:

```c
/* 3. The Final Merged State (Patch 2 + Maintainers' Tweaks) */
#include <linux/array_size.h>
#include <linux/bits.h>
#include <linux/dev_printk.h>
#include <linux/err.h>
#include <linux/i2c.h>
#include <linux/interrupt.h>
#include <linux/mod_devicetable.h>
#include <linux/module.h>
#include <linux/mutex.h>
#include <linux/pm.h>
#include <linux/property.h>
#include <linux/regmap.h>
#include <linux/sprintf.h>
#include <linux/sysfs.h>
#include <linux/types.h>

#include <linux/iio/events.h>
#include <linux/iio/iio.h>
#include <linux/iio/sysfs.h>
#include <linux/iio/types.h>

#include <asm/byteorder.h>
```


I went to send a quick "Thank You" email to the maintainers, only to immediately receive a bounce-back error from the `mlmmj` program managing the `vger.kernel.org` mailing list. The reason? My email client was set to send HTML messages. The kernel mailing lists strictly enforce text/plain emails to preserve bandwidth. Switching my client to Plain Text Mode finally let my message through.
### Conclusion: A Lesson in Culture

The final lesson wasn't about code; it was about culture. Because we sent V2 just a few hours after V1, Jonathan gave us a friendly piece of advice: "Slow down a bit". In the kernel workflow, you should wait a few days before sending updated versions to give developers in different time zones a chance to read and review. He was lenient because we were students submitting a simple patch.

Even though this was a cleanup patch, the experience was invaluable. It demystified the patch-sending pipeline, taught us how to filter automated tools, and showed us that Linux Kernel development is a living, incremental, and deeply collaborative process.