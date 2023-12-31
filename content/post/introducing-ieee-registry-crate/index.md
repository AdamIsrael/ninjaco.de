---
title: "Introducing the IEEE-Registry crate"
description: "IEEE-Registry is a Rust crate that will cache the IEEE Registry CSV files for MAC Addresses, Company IDs, EtherType™, and Manufacturer IDs."
date: 2023-12-30T21:29:16-05:00
draft: false
categories:
- Technical
tags:
- rust
---

I'm doing some quick work on the [arp-scan-rs](https://github.com/AdamIsrael/arp-scan-rs/tree/lib) crate, to convert it from a cli app to cli + library so that its scans can be performed programatically. It basically uses arp to find and identify devices on your local network.

I noticed that its [database of vendors](https://github.com/AdamIsrael/arp-scan-rs/blob/lib/data/ieee-oui.csv) used a very limited version of the IEEE oui database. That got me to thinking, wouldn't it be nice to have a way to _easily_ get a copy of the latest database?

So I sketched out what that might look like. Basically, request the path to the locally cached database. If the file doesn't exist, download it _in situ_. I spent a couple nights putting together [IEEE-Registry](https://crates.io/crates/ieee-registry/0.1.1), which should work as advertised.

```bash
# Perform a scan on the default network interface
$ arp-scan

Selected interface wlp1s0 with IP 192.168.1.21/24
Estimated scan time 2068ms (10752 bytes, 14000 bytes/s)
Sending 256 ARP requests (waiting at least 800ms, 0ms request interval)

| IPv4            | MAC               | Hostname     | Vendor       |
|-----------------|-------------------|--------------|--------------|
| 192.168.1.1     | 91:10:fb:30:06:04 | router.home  | Vendor, Inc. |
| 192.168.1.11    | 45:2e:99:bc:22:b6 | host-a.home  |              |
| 192.168.1.15    | bc:03:c2:92:47:df | host-b.home  | Vendor, Inc. |
| 192.168.1.18    | 8d:eb:56:17:b8:e1 | host-c.home  | Vendor, Inc. |
| 192.168.1.34    | 35:e0:6c:1e:e3:fe |              | Vendor, Inc. |

ARP scan finished, 5 hosts found in 1.623 seconds
7 packets received, 5 ARP packets filtered
```

To get a copy of the database programatically, it's as simple as:

```rust
use ieee_registry::*;

// Get the path to oui.csv, downloading it if necessary.
let oui_path = get_oui_path();
```

or run the included binary:

```
$ ieee-registry
Caching IEEE registry file(s)...
✔ /home/user/.local/share/ieee/cid.csv
✔ /home/user/.local/share/ieee/eth.csv
✔ /home/user/.local/share/ieee/iab.csv
✔ /home/user/.local/share/ieee/mam.csv
✔ /home/user/.local/share/ieee/man.csv
✔ /home/user/.local/share/ieee/opid.csv
✔ /home/user/.local/share/ieee/oui.csv
✔ /home/user/.local/share/ieee/oui36.csv
```

`oui_path` now points to `~/.local/share/ieee/oui.csv`, which can then be parsed and used to lookup the vendor of found ethernet devices. My `arp-scan` went from no vendors detected to almost 90% found.
