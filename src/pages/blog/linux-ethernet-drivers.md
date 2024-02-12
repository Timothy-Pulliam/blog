---
layout: "../../layouts/BlogPost.astro"
title: "Troubleshooting Linux Ethernet Drivers"
description: "Troubleshooting Linux Ethernet Drivers"
pubDate: "Feb 6 2024"
heroImage: "/linux-hero.jpg"
previewText: "Troubleshooting an Intel 10GbE integrated Ethernet Linux driver."
---

Last week I needed to clone a server running on some old Dell hardware. I decided to use [Clonezilla](https://clonezilla.org/). But when I booted it up, I was dismayed to find the live debian environment could not get a DHCP lease.

![dhcp_fail](/linux-ethernet-drivers/cz1.png)

I dropped into a shell and found the OS could not detect the ethernet interface. Usually `NO-CARRIER` indicates no ethernet cable is plugged in, but it also occurs when there is a driver failure.

![no_carrier](/linux-ethernet-drivers/cz2.png)

Working my way up from layer 1, the first thing I did was reseat the ethernet cable. This yielded no change, as I expected. The next thing to check is the ethernet driver in use. Running `lshw -C net` gave the following output:

![lshw](/linux-ethernet-drivers/cz3.png)

The interface is using the `ixgbe` driver, which is used for Intel Ethernet devices. This is expected as this was a 10GbE integrated Intel NIC. Checking the kernel [ring buffer](https://en.wikipedia.org/wiki/Circular_buffer) logs with `sudo dmesg | grep ixgbe`

![dmesg](/linux-ethernet-drivers/cz4.png)

The message `failed to load because an unsupported SFP+ or QSFP module type was detected` corroborates an `ixgbe` driver issue.

After a quick consultation with ChatGPT, I found the following workaround.

```bash
sudo su -
echo "options ixgbe allow_unsupported_sfp=1" > /etc/modprobe.d/ixgbe.conf
# reload ixgbe kernel module to load changes
rmmod ixgbe
modprobe ixgbe
# restart NetworkManager just in case
systemctl restart NetworkManager
ifup eth0
```

After that, `eth0` got an IP address, and I was able to finish the clone process.
