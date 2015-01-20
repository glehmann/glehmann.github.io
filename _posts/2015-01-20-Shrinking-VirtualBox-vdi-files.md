---
title: Shrinking VirtualBox vdi files
---

It is quite easy now to reduce a vdi file, by using the TRIM feature introduced
with the SSD. Here is what I've done on my virtual machines.

I've used this command, with the virtual host down:

    VBoxManage storageattach ubuntu --storagectl "SATA" --port 0 --discard on --nonrotational on

then, inside the ubuntu virtual host, I've run:

    sudo fstrim -v /

Done! This simple operation has freed 5GB!

The `VBoxManage` command was identical for a Microsoft Windows 7 virtual host, and
then I'v run [ForceTRIM] with administrator privileges from the virtual host - 7 more
GB freed on my hard drive!

Of course removing some blocks from the vdi file must have some performance impact.
This is not really a problem with linux, because the trim is only done manually
(unless the drive is mounted with the `discard` option). On window it can be disabled
with `fsutil behavior set DisableDeleteNotify 1`, but I don't know if that's really
worth it. I think I'll continue to use it with the TRIM enable and see how it
works.


[ForceTRIM]: http://www.mediafire.com/download/1cd8dh0msw2jq1w/ForceTrim.zip
