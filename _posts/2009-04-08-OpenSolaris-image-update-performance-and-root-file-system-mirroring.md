---
title: OpenSolaris image-update performance and root file system mirroring
---

My new hosts are not there yet, but I'm practicing OpenSolaris on my old Athlon 64 2800+, with 767MB of RAM, two hard drive of 80 GB and one hard drive of 40 GB. The installation of OpenSolaris 2008.11 on one on the 80 GB hard drive goes fine. The only problem: the network adapter on the motherboard is not detected. Fortunately, I have a second one which is detected. The default network configuration with NWAM works out of the box — great.

Just after the install, I've updated the OS to the last development release — quite easy, with just a few commands:

    pkg set-authority -P -O http://pkg.opensolaris.org/dev opensolaris.org-dev
    pkg install SUNWipkg
    pkg image-update

Easy but a bit long: 96 minutes to run `pkg image-update`. The problem seems to be the low bandwidth between `pkg.opensolaris.org` and me. The network load was roughly at 90-100 KB/s during the download time. `pkg` puts everything in a cache (in `/var/pkg/download`) so a second run, with 21 minutes, gives the time spent waiting for the network transfer — almost 75 minutes. For a better user experience, OpenSolaris community should think to setup mirrors closer to the users. It would be even better if the mirrors used can be automatically configured, based on the location given during the install process, for example.

`pkg` operations is known to be quite I/O demanding. Can we enhance the performance with a mirrored root file system?

Installing a second boot disk as a mirror of the first one is still tricky on OpenSolaris. I found many informations in many places, but unfortunately, most of them don't describe the usage of the `format` command required to prepare the new drive. It's not that difficult, but it's not easy to figure out how to use it the first time. Here is the procedure:

    format -e
    1  (select the disk number)
    fdisk
    3  (delete a partition)
    1  (the partition number)
    y
    5  (exit)

Running again `fdisk` will propose to use a default partition — the great news is that it's exactly what we want:

    fdisk
    y
    label
    y
    quit

Then a cryptic command to copy the partition table from the first disk to new one, or something of that kind:

    prtvtoc /dev/rdsk/c3d0s2 | fmthard -s - /dev/rdsk/c3d1s2

Actually create the mirror:

    zpool attach rpool c1d0s0 c0d0s0

And, to be able to reboot on the new mirrored drive, install grub on the new drive:

    installgrub -m /boot/grub/stage1 /boot/grub/stage2 /dev/rdsk/c3d1s0
  
It would be great to have a simple command to make all those step in an easy way. Maybe its there, but I didn't found it.

I waited for the resilvering to complete (20 minutes) and ran the image-update again: 21 minutes again. No luck, `pkg image-update` doesn't seem to be I/O bounded on that host. But at least I now have the data protection in place :-)
