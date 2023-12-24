---
title: "Announcing Smartrg808ac_exporter"
description: "I wired up a Prometheus endpoint for my cable modem so I could monitor its uptime."
date: 2021-02-23T21:36:24-05:00
draft: false
categories:
- Technical
tags:
- Prometheus
- Grafana
- Monitoring
- Open Source
image: "img/teksavvy.svg"
---

A while back, I switched from DSL to Cable Internet. My awesome ISP, TekSavvy, sent me a loaner cable modem -- a SmartRG 808AC. It worked well, despite a few quirks, but lately I've noted that the internet would drop out for a few minutes at a time. I was able to deduce that the modem seems to be rebooting itself, but I had no idea how often it was happening.

Sidetrack: I'd been meaning to setup Prometheus + Grafana for monitoring my home network. It's one of those things that look awesome but I never got around to figuring out the details. I had the itch one day recently, and after reading up on how Prometheus worked, I realized it would be a great way to monitor my modem and find out exactly how often it's rebooting itself.

So I fired up a fresh LXD container, installed Prometheus and Grafana, and forwarded the appropriate ports. Ready, set, go.

Prometheus works by scraping a `/metrics` endpoint, either baked in to an application/device or provided by an "exporter", of which there are hundreds already written.

The SmartRG 808AC doesn't have a `/metrics` endpoint, nor was there an exporter written for it, but luckily there's a [handy guide](https://prometheus.io/docs/guides/go-application/) to writing one. I also borrowed from [uptimerobot-prometheus-exporter](https://github.com/masaruhoshi/uptimerobot-prometheus-exporter) and [speedtest_exporter](https://github.com/nlamirault/speedtest_exporter) while looking for examples.

After a night or two of hacking, I'm proud to announce the first release of the [smartrg808ac_exporter](https://github.com/AdamIsrael/smartrg808ac_exporter).

Right now, it only exposes two metrics: `up`, which will return a 0/1 if the device is up and `uptime_seconds`, which tells you how long the device has been online. Next steps are to add interface metrics.

Back to my original problem; I now have a way to measure how often my cable modem reboots.

![Uptime over 24 hours](/images/smartrg808ac_uptime.png "SmartRG 808AC Uptime")

Over the course of the next few days, I should have enough information to see if there's a pattern to the restarts, and how consistent it is. Plenty enough evidence to contact TekSavvy and see about getting a replacement modem.

