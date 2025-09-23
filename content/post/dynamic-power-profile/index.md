---
title: "Dynamic Power Profiles"
description:
date: 2025-09-22T19:04:49-04:00
image:
math:
license:
hidden: false
comments: true
draft: true
categories:
- Technical
tags:
- linux
- rust
- dbus
- oss
- power-management
---

I recently upgraded my 13" Framework laptop from an Intel Intel i7-1165G7 to an AMD Ryzen AI 9 HX 370. The only thing that's really bothered me about the Framework laptop is its battery life. There are a lot of tools available for optimizing power management -- [tlp](https://linrunner.de/tlp/index.html), [powertop](https://linuxconfig.org/how-to-check-and-tune-power-consumption-with-powertop-on-linux), [power-profile-daemon](https://gitlab.freedesktop.org/upower/power-profiles-daemon), etc -- but I had an idea for a quick and easy win.

When I'm plugged into a power source, I want maximum performance. When I'm on battery, I want maximum battery life. I don't want to have to manually switch between power profiles. I don't want to have to think about it.

So I wrote [dynamic-power-profile](https://github.com/AdamIsrael/dynamic-power-profile), a simple daemon written in Rust that monitors the power state via dbus and changes the power profile accordingly. It's available via [crates.io](https://crates.io/crates/dynamic-power-profile).

Right now it's highly opinionated. It requires dbus. It runs under systemd. When on power, it will set the profile to `performance`, if available, or `balanced`. If you're on battery, it will set the profile to `power-saver`.

It's been tested and works on Bluefin and GNOME but should work on any Linux/x86 system with dbus.

Next steps:
- measure battery life on battery life on different profiles
- setup Github Actions to build tagged releases
  - figure out how to publish new versions to brew
- make the power profile configurable
- add the ability to execute custom commands based on power state, e.g., change the screen brightness automatically
