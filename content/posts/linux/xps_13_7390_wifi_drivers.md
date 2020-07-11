---
title: "Dell XPS 13 7390 - WiFi drivers on Fedora"
date: 2020-07-11T13:26:57+10:00
draft: false
tags: ['linux', 'xps 7390', 'wifi', 'drivers', 'fedora']
categories: []
---

My WiFi drivers fail from time-to-time, usually after a Kernel upgrade.

For future reference, this is the series of commands I run to fix them.

**Source**: https://gitlab.com/emrose/xps13-7390_debian/-/issues/5#note_240886447

```bash
git clone https://chromium.googlesource.com/chromiumos/third_party/linux-firmware chromiumos-linux-firmware
cd chromiumos-linux-firmware
sudo cp iwlwifi-*  /lib/firmware/
cd /lib/firmware
sudo ln -s iwlwifi-Qu-c0-hr-b0-50.ucode iwlwifi-Qu-b0-hr-b0-50.ucode
```