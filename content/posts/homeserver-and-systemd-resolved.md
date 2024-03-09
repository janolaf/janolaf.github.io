---
title: "Homeserver and Systemd Resolved"
date: 2022-11-22
draft: false
toc: false
images:
categories:
  - fedora
tags:
  - dns
  - systemd-resolved
  - fedora
---

## It is always dns

One thing I always forget to do after installing Fedora Server, is to change systemd-resolved to point to my local dns server.

I have a local authoritative dns server setup to handle dns queries for some of the services I host.

If you are running a DNS server on your home network, most guides recommend that you disable systemd-resolved.
You do not have to do that, and in my honest opinion, should not disable systemd-resolved. The fix is actually quite simple.

First, do not edit 'resolved.conf'. Every time you update systemd-resolved, 'resolved.conf' might be overwritten.
Instead, create a directory '/etc/systemd/resolved.d'.
create a file, and name it `something.conf`. In `something.conf` add your DNS server IP with `DNS=IP`. You can add `FallbackDNS=` to add alternative DNS resolution in cases were your home dns is non-responsive.

Example:
``` systemd
[Resolve]
DNS=10.0.0.53
FallbackDNS=8.8.8.8, 9.9.9.9
Cache=yes
Domains=home.local
```
Save and restart systemd-resolved,
```bash:
systemctl restart systemd-resolved.service
```

That is it, and I always forget to do this. It's probably something I should have added to my ansible scripts.
