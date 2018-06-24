---
title: "Building affordable Self-service Kiosks with Raspberry Pi"
date: 2018-06-19T18:34:07+10:00
draft: false
tags: ['raspberry', 'pi', 'dell', 'touchscreen', 'P2418HT', 'kiosk']
categories: ['Projects', 'Linux']
---

Recently, I was tasked with designing and setting up a simple, cost-effective, touchscreen-based self-service Kiosk which users can intuitively use to navigate a predefined website (and nothing else).

After spending some time researching the options (and realising just how expensive commercial touchscreens are!), I settled on the Dell P2418HT 24" touchscreen monitor and a Raspberry Pi. At a total cost of under $600 per Kiosk, it was a bargain when compared to the other options I'd come across ($1,200+ for the display alone)!

Unfortunately, I wasn't able to find much online about running the Dell P2418HT with a Raspberry Pi and was advised by Dell that it was not supported. Determined to save money, I purchased the monitor anyway with the view of being able to upgrade to an Intel Atom-based NUC at a later date, if the monitor wasn't compatible. It helped that I had a spare Raspberry Pi lying around, too!

To my surprise, both Alpine Linux (ARMHF) and Raspbian detect the touchscreen as a generic Touch-input HID device, so the touchscreen functionality worked out of the box!

As with anything, security should be of paramount importance, so I needed a way to lock users down to only the one particular website, and prevent them from being able to change browser/OS configuration. 
A combination of Getty, an unprivileged user, and Chromium-browser's `kiosk` mode seemed to fit these requirements perfectly, without impacting on the user's ability to use the desired self-service website. I've documented the steps required to do this, below.


First, you'll need to create a new, unprivileged user. e.g. `kioskuser`.

```bash
useradd -m kioskuser
```

You'll then need to configure Getty to automatically login as this user, on tty1. The process for doing this will differ depending on whether you use Systemd or SysVinit.

**With SysVinit**:

  1. Update the `/etc/inittab` file and change the tty's `respawn` config, so that it looks like this:

    ```
    tty1::respawn::/bin/login -f kioskuser
    ```
  2. If you're using Alpine Linux, don't forget to commit your changes to disk:
  ```bash
  lbu_commit -d
  ```

**With Systemd**:

  1. Create a file at `/etc/systemd/system/getty@tty1.service.d/override.conf` with the following content:
  
    ```
    [Service]
    ExecStart=
    ExecStart=-/sbin/agetty -a kioskuser %I $TERM
    ```
  2. Reload the daemon and enable the `getty@tty1` service, with:
    ```bash
    sudo systemctl daemon-reload
    sudo systemctl enable getty@tty1.service
    ```

Next time you reboot your computer (or terminate your session, if using `tty1`), you should automatically be logged in as `kioskuser`. 

Next, we need to configure Xorg to automatically run the browser when a new X session is initiated.

You can do this by logging in as the unprivileged kiosk user that we recently created (e.g. `kioskuser`), and creating a `~/.xinitrc` file, with the following content.
```bash
#!/bin/sh
exec chromium-browser --kiosk http://myselfserviceurl.com/path
```

You may also need to update your `~/.bash_profile` file so that it starts the X server on login (as long as a display is present):
  ```bash
  #!/bin/bash
  [[ -z $DISPLAY ]] && startx # Start X server if DISPLAY env var is set. 
  ```

Once you've saved this file, reboot the raspberry Pi and you should hopefully be greeted with a full-screen Chrome browser with `http://myselfserviceurl.com/path` loaded!

If the browser crashes, or if you restart the Raspberry Pi, Chrome will automatically start up again and load the desired URL, running as a user with minimal privileges.

If Chrome doesn't launch in full-screen mode, you may need to disable `Overscan` in your display preferences, or force Chromium to start with a specific resolution. To simplify the latter, you can update your `~/.xinitrc` file to retrieve the current display resolution automatically, as follows:
```bash
#!/bin/sh
# Get the resolution
resolution=`xrandr --current | grep 'default connected' | cut -f3 -d ' '`
width=`echo $resolution | cut -f1 -d 'x'`
height=`echo $resolution | cut -f2 -d 'x' | cut -f1 -d '+'`

exec chromium-browser --kiosk http://myselfserviceurl.com/path --window-size=$width,$height
```