# Guide: Upgrading your Gentoo kernel to 5.17.0

#### // [Project 081](https://p081.github.io) | [Elevendebloater](https://spdgmr.github.io/elevendebloater) | [sfetch](https://spdgmr.github.io/sfetch) | [More Projects](https://spdgmr.github.io/projects) | [spDE](https://spdgmr.github.io/spde) | [Discord server](https://ffdiscord.github.io) | [Twitter](https://nitter.net/spdgmr) | [YouTube](https://invidious.namazso.eu/speedie) | [RSS](https://raw.githubusercontent.com/spdgmr/posts/main/rss.xml)
--------------

## Guide: Upgrading your Gentoo kernel to 5.17.0

This guide will go over how to upgrade your kernel on Gentoo Linux to 5.17.0. If we check the [sys-kernel/gentoo-sources](https://packages.gentoo.org/packages/sys-kernel/gentoo-sources) package we can see that 5.17.0 is now available as of 2022-03-23. However it is not stable so we will need to unmask it.

First, let's use the ``su`` command to change our user to root since many commands will require root permissions. You could use sudo/doas for each command but that's painful.

Now, one pretty unstable way to do this is to simply have ``ACCEPT_KEYWORDS="~amd64"`` in ``/etc/portage/make.conf`` but this is not stable and will ALWAYS allow "untested" packages to be installed on your system. I and others prefer this but it's not something you should use if you care about a stable system.

So instead, let's individually unmask this package. Create ``/etc/portage/package.unmask`` and add ``sys-kernel/gentoo-sources`` to the file

![image](https://user-images.githubusercontent.com/71722170/159775895-da08c9e8-0894-497a-9493-acc134408681.png)

Now, your sources are likely outdated so we will need to update them. To do this, run ``emerge --sync && emerge-webrsync``
Once we've synced our repositories, let's try running ``emerge --ask gentoo-sources``. If the version is 5.17.0 then you've correctly unmasked and synced your repositories. 

![image](https://user-images.githubusercontent.com/71722170/159776932-2d1245e1-e583-43cc-9720-4991c2b7dc7e.png)

Now ``emerge`` this and wait.

![image](https://user-images.githubusercontent.com/71722170/159777036-51ba456f-dd09-477e-8210-09a034d97cf2.png)

Once the emerge is complete it's time to back up our kernel configuration. ``cd /usr/src/linux`` and ``cp <kernel> /home/<user>`` where ``<kernel>`` is the name of your kernel config and ``<user>`` is your user account. You don't have to back it up to this place but you need a backup of your kernel configuration unless you wanna configure a new one. Now ``cd /home/<user>``.

Now that we have a backup of our previous configuration, it's time to change our symlink. Run ``eselect kernel list``

![image](https://user-images.githubusercontent.com/71722170/159777139-b812a338-892b-454e-ad88-c86d2dda68ce.png)

Now take note of all the available kernels and their numbers.

Then run ``eselect kernel set <number>`` where ``<number>`` is the number of the kernel you will be using (should be 5.17.0)

![image](https://user-images.githubusercontent.com/71722170/159777376-eb2f34b6-e4be-4f43-9bb3-de1632a47791.png)

``cd /usr/src/linux`` and ``cp /home/<user>/<kernel> /usr/src/linux/.config && make olddefconfig``. 

This will copy the kernel config to the 5.17.0 kernel directory. The ``make olddefconfig`` will use your old kernel and set the new settings newer kernels offer to the defaults. This should be fine unless you NEED to change them.

Once this is complete, make sure your /boot is mounted and then run ``make -j$(nproc) && make install`` to compile the kernel and copy it over to your /boot partition.

Finally, let's make sure our bootloader can find it. For Grub, you need to ``grub-mkconfig -o /boot/grub/grub.cfg`` but if you're not using Grub then you will have to use [[searx]](https://searx.org) to find out what to do.

**IMPORTANT: If you are using an initramfs then you need to create a new one. To do this you can use either ``genkernel`` or ``dracut``. To use genkernel, run ``genkernel --install --kernel-config=/usr/src/linux/.config initramfs``. Then just repeat the previous step.**

Now you're done. Now reboot and make sure your kernel boots. If you cannot boot, you will have to do some troubleshooting..

Have a good day and good luck with Gentoo.

--------------
