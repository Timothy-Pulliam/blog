---
layout: "../../layouts/BlogPost.astro"
title: "Recover a lost root password"
description: "Recover a lost root password"
pubDate: "Jan 15 2024"
heroImage: "/linux-hero.jpg"
previewText: "Recovering a root password is an essential skill for a budding Linux Administrator. There _will_ be times when you log into a system that either you, or someone else has set up, and you don't have the root password! This post will show you how to recover it and look like a kernel sorcerer while doing so."
---

# Overview

Recovering a root password is an essential skill for a budding Linux Administrator. There _will_ be times when you log into a system that either you, or someone else has set up, and you don't have the root password! This post will show you how to recover it and look like a kernel sorcerer while doing so.

# Resetting the root Password

First, you will need access to the bootloader (e.g. GRUB). This means you will either need physical acces to the system, access to Out-of-Band, iDRAC, IPMI, or Serial interface, or you will need access to the VM console.

Here I am using [GNOME Boxes](https://docs.fedoraproject.org/en-US/quick-docs/installing-virtual-systems-with-gnome-boxes/), which is an application that comes pre-installed on Fedora 39.

Boot into GRUB. Press 'e' to edit the kernel parameters.

![grub.png](/linux-reset-root-passwd/grub.png)

Find the line that starts with `linux`. This command [loads a kernel image from the specified file](https://www.gnu.org/software/grub/manual/grub/html_node/linux.html#linux). The line can also start with `linux16` if it is [loading a kernel in 16 bit mode](https://www.gnu.org/software/grub/manual/grub/html_node/linux16.html).

Append `rd.break` at the end of the line. This will cause the [initramfs to drop into a shell after it has been loaded](https://man7.org/linux/man-pages/man7/dracut.cmdline.7.html).

![boot_options.png](/linux-reset-root-passwd/boot_options.png)

Press `Ctrl+x` to boot the kernel with these options.

You will be dropped into an initramfs root shell, which means the `/` filesystem will not be mounted. We must mount it with the following commands

- `mount -o remount,rw /sysroot`
- `chroot /sysroot`

![chroot.png](/linux-reset-root-passwd/chroot.png)

You can then reset the root passwd using the `passwd` command. However, if you are using SELinux, you will need to relabel the context of the `/etc/shadow` file. The easy way to do this is to `touch /.autorelabel`. The presence of this file at boot time will cause SELinux to automatically relabel all file's contexts on the filesystem.

![chroot2.png](/linux-reset-root-passwd/chroot2.png)

Exit the chroot shell, and exit the initramfs shell and linux will reboot. The autorelabel process may take some time, especially on larger filesystems, since it needs to read every file.

![exit.png](/linux-reset-root-passwd/exit.png)

# Conclusion

And that's pretty much all there is to it. This has saved my bacon several times and saved me from having to reimage a system because of a lost root password.
