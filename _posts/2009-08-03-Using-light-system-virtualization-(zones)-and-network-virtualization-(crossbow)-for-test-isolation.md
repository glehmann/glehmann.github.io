---
title: Using light system virtualization (zones) and network virtualization (crossbow) for test isolation
---

As an [ITK] developer, the main developer of [WrapITK] and the happy user of a powerful workstation, I'm used to run quite a lot of tests every nights. I first began to run those tests on linux, where not much isolation mechanisms are available. It has work quite well for some time, but these days, I'm getting some bad interactions with the installed version of the code. Also, it's a bit difficult to test the installation procedure, because I'm never sure to have perform a really clean uninstall.

The zones in OpenSolaris provide very efficient and light weight virtualization, which should help to fix my problems.

First, I configured a zfs file system where I'll store the zones. I think the compression was activated by default in earlier version in the file system dedicated to each zone. It doesn't seem to be the case anymore in snv_118, so I turn it on on the base file system which will store the file system of all the zones.

    zfs create -o mountpoint=/zones -o compress=on rpool/zones

Then build the virtual network infrastructure.

    dladm create-etherstub switch0
    dladm create-vnic -l switch0 zglobal0
    ifconfig zglobal0 plumb
    ifconfig zglobal0 192.168.0.1
    ifconfig zglobal0 up
    echo 192.168.0.1 > /etc/hostname.zglobal0
    cat <<EOF >> /etc/ipf/ipnat.conf
    map bge0 192.168.0.0/24 -> 0/32  portmap tcp/udp auto
    map bge0 192.168.0.0/24 -> 0/32
    EOF
    svcadm enable ipfilter ipv4-forwarding

Now this part is specific to the zone, and must be done for each zone.

    dladm create-vnic -l switch0 zitk0
    echo 192.168.0.2  itk >> /etc/hosts

We don't plumb the new interface: it will be done in the zone. It would have be possible to forward a port, for example the 80 to host a web server in the zone, by adding a line like

    rdr bge0 0/0 port 80 -> itk port 80 tcp

to /etc/ipf/ipnat.conf — that's not useful for a test zone.

The configuration for the zone is stored in the file "itk.zone" - it makes the editing and the rereading of the configuration a bit easier.

    create
    set zonepath=/zones/itk
    set ip-type=exclusive
    add net
      set physical=zitk0
    end

Ok, everything is there. The zone can be created, installed and booted. The installation is a bit long if it's the first zone that you install, because the files are not yet cached.

    zonecfg -z itk -f itk.zone
    zoneadm -z itk install
    zoneadm -z itk boot

Last steps, log into the zone, and configure it

    zlogin -C itk

This is the standard sysidconfig process. When asked for the configuration of the interface zitk0, I gave the adress 192.168.0.2, 192.168.0.1 as the getaway, and the same DNS server than the one used in the global zone. It must be possible to automate this step with a [sysidcfg] file, but I didn't looked much at sysidconfig yet, and I don't plan to create many zones, so I'll do with this manual configuration step for now.

Then create a new user — me

    mkdir -p /export/home
    useradd -d /export/home/glehmann -m -s /usr/bin/bash -u 101-c "Gaetan Lehmann" glehmann
    passwd glehmann

Once completed, it is possible to quit the console of the zone by typing the escape sequence

    ~.

I can now log into the virtual host with ssh to prepare all my tests.

Next task, give only a few memory to that zone and a very low cpu priority.

[itk]: http://www.itk.org
[WrapITK]: http://wrapitk.googlecode.com
[sysidcfg]: http://docs.sun.com/app/docs/doc/817-5504/6mkv4nh2r?a=view
