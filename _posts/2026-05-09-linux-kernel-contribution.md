---
layout: post
title: "Cleaning up includes in the IIO Subsystem: My First Linux Kernel Contribution"
date: 2026-04-30 10:00:00 -0300
categories: [Linux Kernel, FLUSP]
tags: [IIO, Drivers, linux, kernel-linux, c, IWYU]
---

Alongside my colleague Felipe Khoury Dayoub, I have just had my first contribution accepted into the Linux Kernel. This was done within the Industrial I/O (IIO) subsystem, specifically targeting the `stk3310` ambient light and proximity sensor driver. 

This contribution was part of our coursework at the Institute of Mathematics and Statistics of the University of São Paulo (IME-USP), and it taught me that contributing to a massive open-source project is as much about understanding community workflows as it is about writing code.

### The Contribution [V1]

Our goal was to refactor the included headers of the `drivers/iio/light/stk3310.c` file. Historically, many drivers relied on catch-all headers like `<linux/kernel.h>` or `<linux/device.h>`. The modern kernel standard encourages explicit dependencies, and a popular way to map these is by using the Include-What-You-Use (IWYU) tool.

We ran the tool, removed `<linux/kernel.h>`, added the explicitly required headers, and sent our first patch.

### First Hurdle: The Git Send-Email Workflow

Before even getting feedback, simply *sending* the patch was my first technical hurdle. As a CS student used to GitHub pull requests, the kernel's email-based workflow is a paradigm shift. 

When trying to use `git send-email` with Gmail's SMTP, I faced authentication failures. I quickly learned that standard passwords don't work; you need to configure an App Password via OAuth2. Furthermore, my Ubuntu environment threw a `No SASL mechanism found` error, which required installing the `libauthen-sasl-perl` library to allow Git to authenticate with Google's servers. Once configured, the patch finally reached the mailing list.

### Feedback and Changes: The V2 Patchset

The Linux Kernel community is incredibly responsive. Shortly after sending V1, we received feedback from reviewer Joshua Crofts and the IIO subsystem maintainer, Jonathan Cameron. Their feedback highlighted three major lessons:

**1. IWYU generates noise:**
While IWYU is a fantastic tool, it isn't perfect. It suggested headers like `<linux/kconfig.h>` and `<linux/sysfs.h>` which weren't strictly necessary. More importantly, Jonathan pointed out that the community is actively trying to stop including `<linux/device.h>` directly unless absolutely needed, encouraging forward declarations (`struct device;`) instead.

**2. Visual Style Matters:**
We were instructed to group the headers logically, separating the generic `<linux/*>` headers from the subsystem-specific `<linux/iio/*>` headers with a blank line, and keeping both blocks strictly alphabetized.

```c
/* Generic headers */
#include <linux/i2c.h>
#include <linux/interrupt.h>
#include <linux/module.h>
#include <linux/regmap.h>

/* IIO specific headers */
#include <linux/iio/events.h>
#include <linux/iio/iio.h>
#include <linux/iio/sysfs.h>
```

**3. Atomic Commits:**
Instead of doing everything in one commit, the maintainers asked us to split our work into a patchset. The first patch should exclusively handle the alphabetical sorting of the existing headers, and the second patch should handle the addition/removal of headers dictated by IWYU. This makes the reviewing process much easier.


### The Final Hurdle: Plain Text Emails

After applying the changes, testing the compilation, and sending the V2 patchset, Jonathan Cameron accepted our code into the iio.git testing branch!

I went to send a quick "Thank You" email to the maintainers, only to immediately receive a bounce-back error from the mlmmj program managing the vger.kernel.org mailing list. The reason? My email client was set to send HTML messages. The kernel mailing lists strictly enforce text/plain emails to preserve bandwidth and ensure compatibility with terminal-based email clients.

Switching my email client to Plain Text Mode finally let my message through—a true rite of passage.

### Conclusion

Even though this was a simple cleanup patch, the experience was invaluable. It demystified the patch-sending pipeline, taught me the importance of atomic commits, and showed me how welcoming and meticulous the kernel maintainers are when guiding new submitters.