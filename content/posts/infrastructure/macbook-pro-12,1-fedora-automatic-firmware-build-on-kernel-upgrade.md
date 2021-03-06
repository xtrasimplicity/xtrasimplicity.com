---
title: "Late 2015 Macbook Pro 12,1 + Fedora: Getting the Webcam to work (and automating the process after Kernel upgrades)"
date: 2018-11-17T09:49:44+10:00
draft: false
---

**Update (4/12/2018): The `master` branch of `Patjak`'s `bcwc_pcie` repository is not compatible with Linux kernel 4.19.2+. Thankfully, the `mainline` branch _is_ compatible.**

**The code below has been updated, but if you experience issues please try switching back to the `master` branch by changing the `repository_branch` variable at the top of the script.**

When upgrading the Kernel on a Fedora workstation running on a Late 2015 MacBook Pro 13", the Webcam ceases to function (as the drivers are compiled against the current kernel).

To get around this, I like to use the following script to automatically rebuild the firmware drivers against the new kernel, on kernel installation.

Hint: Make sure to set the following script as executable (i.e. `chmod o+x /etc/kernel/postinst.d/install-facetimehd.sh`).

Note: This makes use of [Patjak's reverse-engineered Broadcom 1570 drivers](https://github.com/patjak/bcwc_pcie), which he has generously published on Git.

```bash
# /etc/kernel/postinst.d/install-facetimehd.sh
#!/bin/sh
firmware_source_path=/usr/src/bcwc_pcie
repository_url=https://github.com/patjak/bcwc_pcie.git
repository_branch=mainline

echo "Git: Cloning source into ${firmware_source_path}..."
git clone "${repository_url}" "${firmware_source_path}" -b "${repository_branch}"

cd "${firmware_source_path}"

echo "Git: Ensuring source is clean of any build files..."
git clean -fdX "${firmware_source_path}"

echo "Building firmware..."
cd firmware && \
   make && \
   sudo make install

cd .. # Change back to the repo's root directory

echo "Building module..."
make && make install

echo "Loading modules..."
depmod && modprobe facetimehd && echo 'All done!'
```