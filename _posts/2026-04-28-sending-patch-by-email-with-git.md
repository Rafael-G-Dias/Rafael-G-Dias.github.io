---
title: Setting Up Git and OAuth for Linux Kernel Patch Submission
date: 2026-04-28 21:00:00 -0300
categories: [Open Source Software Development, Environment Setup]
tags: [linux, kernel, git, tutorial, email, oauth]
---

After getting the virtual machine and kernel compilation tools ready, the next crucial step in kernel development is configuring the communication channel. 

Through FLUSP's Tutorials 5 and 6, I configured my environment to use my university (`@usp.br`) email to send patches. What seemed like a straightforward configuration quickly turned into a great lesson on modern authentication protocols and local proxies.

### Tutorial 5: The Basics
Tutorial 5 was quite brief. It covered the standard Git setup: defining the global `user.name` and `user.email`, and pointing Git to the correct SMTP server (`smtp.gmail.com` with port 587). The tricky part, however, arises when you actually try to authenticate. Since basic App Passwords are often restricted by institutional Google Workspace policies (like USP's), falling back to standard password authentication wasn't an option. This naturally led me to Tutorial 6.

### Tutorial 6: Tackling OAuth2 Authentication
To bypass the App Password restriction, we needed to use OAuth 2.0. The tutorial provided two main approaches, and I ended up exploring the technical hurdles of both.

#### Difficulty 1: The "Testing Phase" 403 Error (Option A)
I initially tried using the `git-credential-gmail` helper. After setting up the client IDs and firing the authentication command, my browser opened the Google login screen, but immediately threw an **Error 403: Access Blocked**.
* **The Root Cause:** The custom OAuth proxy application created by the course mentors was still labeled as "In Testing" on the Google Cloud Console. Google aggressively blocks unauthorized users from testing apps.
* **The Solution:** I had already added my email to the shared course document, but the synchronization isn't automatic. I bypassed the wait by directly emailing the proxy maintainer (`dsl26oauthproxy@gmail.com`) to get my email explicitly whitelisted as an approved tester.

#### Difficulty 2: TLS Clashes with Local Docker Proxies (Option B)
While dealing with the OAuth whitelisting, I explored the alternative method: spinning up a local email proxy using Docker. After successfully building the container and running it on `localhost:2587`, my `kw send-patch` command failed miserably with a `STARTTLS failed!` error.
* **The Root Cause:** Earlier, I had globally configured Git to enforce TLS encryption (`git config --global sendemail.smtpencryption tls`). However, the Git client was now talking to a local Docker container (`127.0.0.1`), which expects an unencrypted local handshake before wrapping the message and sending it securely to Google.
* **The Solution:** I needed to override the global setting just for this specific local repository without breaking my main Git config. Running `git config --local sendemail.smtpencryption ""` cleared the local TLS requirement, allowing Git to successfully handshake with the Docker proxy.

### Conclusion
Setting up the email environment was surprisingly intricate, mainly due to modern security constraints and institutional policies. Overcoming these proxy and encryption mismatches was a great debugging exercise. Now, with the `git send-email` pipeline fully operational and authenticated, I am finally ready to start sending actual code contributions and dive into the Industrial I/O (IIO) subsystem.