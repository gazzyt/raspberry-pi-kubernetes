## Preparing the PI
Having got a 4GB Raspberry Pi 4 with a fan to keep it cool I decided I would need some additional (fast) storage for all those container images. Having seen [Andreas Spiess' video about booting a PI 4 from an SSD](https://www.youtube.com/watch?v=gp6XW-fGVjo) I decided an SSD would be the way to go now that we have USB3 on the PI.

I bought this [Platinum Portable USB 3.0 SSD 120GB 128GB - Macbook / Laptop - Very fast!](https://www.ebay.co.uk/itm/Platinum-Portable-USB-3-0-SSD-120GB-128GB-Macbook-Laptop-Very-fast/362626751785?ssPageName=STRK%3AMEBIDX%3AIT&_trksid=p2057872.m2749.l2649) from Ebay as is promised quick delivery and a few days later it arrived.

The YouTube video uses the instructions from the [following blog post](https://jamesachambers.com/raspberry-pi-4-usb-boot-config-guide-for-ssd-flash-drives/) which I won't repeat verbatim.

Having flashed the SD card and the SSD and added the `ssh` file to the boot partition of both I booted the PI on the wired network with the SSD disconnected. It booted and I could access it via ssh.

Plugging the SSD in to the blue USB3 port the drive was recognised:
```
pi@raspberrypi:~ $ lsusb
Bus 002 Device 002: ID 174c:1153 ASMedia Technology Inc. ASM1153 SATA 3Gb/s bridge
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 002: ID 2109:3431 VIA Labs, Inc. Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
pi@raspberrypi:~ $
```

Happier still the ASMedia controller it uses is listed as "known good" in the blog post.

Following the steps for changing the PARTUUID and /boot/cmdline.txt I rebooted the PI but could not ssh to it. I waited 10 minutes but still no luck although I could ping it.
So I connected a keyboard and screen and rebooted it. Everything booted fine but the ssh daemon had not started. I enabled this manually via raspi-config and ran the test to see if we had really booted from the SSD:
```
pi@raspberrypi:~ $ findmnt -n -o SOURCE /
/dev/sda2
```

Now we are ready to update /etc/fstab and resize the filesystem following the blog post. This went smoothly for me.
```
pi@raspberrypi:~ $ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/root       110G  3.9G  102G   4% /
devtmpfs        1.8G     0  1.8G   0% /dev
tmpfs           2.0G     0  2.0G   0% /dev/shm
tmpfs           2.0G  9.0M  1.9G   1% /run
tmpfs           5.0M  4.0K  5.0M   1% /run/lock
tmpfs           2.0G     0  2.0G   0% /sys/fs/cgroup
/dev/mmcblk0p1  253M   53M  200M  21% /boot
tmpfs           391M     0  391M   0% /run/user/1000
```
Success! It's fast!

Finally:
* Update the OS using apt
* Give it a better hostname
* Change the password of the pi user
