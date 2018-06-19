---
title: "Building affordable Self-service Kiosks with Raspberry Pi"
date: 2018-06-19T18:34:07+10:00
draft: false
---

Recently, I was tasked with designing and setting up a simple, cost-effective, touchscreen-based self-service Kiosk which users can intuitively use to navigate a predefined website (and nothing else!)

After spending some time researching the options (and realising just how expensive commercial touchscreens really are!), I settled on the Dell P2418HT 24" touchscreen monitor and a Raspberry Pi. Coming in at a total cost of just under $600 per Kiosk, it was a bargain when compared to the other options ($1,200+)!

Unfortunately, I wasn't able to find much online about running the Dell P2418HT with a Raspberry Pi and was advised by Dell that it was not supported. Determined to save money, I purchased the monitor anyway with the view of being able to upgrade to an Intel Atom-based NUC at a later date, if the monitor wasn't compatible. It helped that I had a spare Raspberry Pi lying around, too!

To my surprise, both Alpine Linux (ARMHF) and Raspbian detect the touchscreen as a generic Touch-input HID device, so the touchscreen functionality worked out of the box!

As with anything, security should be of paramount importance, so I needed a way to lock users down to only the one particular website, and prevent them from being able to change browser/OS configuration. A combination of Getty, an unprivileged user, and Chromium-browser's `kiosk` mode seemed to fit these requirements perfectly, without impacting on the user's ability to use the desired self-service website.

To do this, I first needed to create a new, unprivileged user. e.g. `kioskuser`.

```bash
useradd -m kioskuser
```

I then needed to configure Getty to automatically login as this user, on tty1. On Alpine Linux, you can do this by updating `/etc/inittab` and changing the tty's `respawn` entry, so that it looks like this:
  ```
  tty1::respawn::/bin/login -f kioskuser
  ```
If you're using Alpine Linux, don't forget to commit your changes to disk:
```bash
lbu_commit -d
```

Next time you reboot your computer (or terminate your session, if using `tty1`), you should automatically be logged in as `kioskuser`. Next, we need to configure Xorg to automatically run the browser when a new X session is initiated.

You can do this by logging in as the unprivileged kiosk user that we recently created (e.g. `kioskuser`), and creating a `~/.xinitrc` file, with the following content.
```bash
#!/bin/sh
exec chromium-browser --kiosk http://myselfserviceurl.com/path
```

Once you've saved this file, login to a new session on tty1 as the kiosk user and you should hopefully be greeted with a full-screen Chrome browser with `http://myselfserviceurl.com/path` loaded! If the browser crashes, or if you restart the Raspberry Pi, Chrome will automatically start up again and load the desired URL. Awesome! :)