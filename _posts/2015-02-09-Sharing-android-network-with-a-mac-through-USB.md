---
title: Sharing android network with a mac through USB
---

I'm often sharing my network on my android smartphone with my mac, and I'm often
plugin my phone to my mac to charge the phone. It is quite obvious to me that
the network should go through the USB connection, but unfortunately, I've never
been able to make it work that way, and was always relying on the wifi to share
the network.

I finally found that Mac OS X does not have the required drivers — which is a shame
my opinion. Fortunately, there is a nice open source project to fix that lack:
[HoRNDIS].

I've installed it on the mac, restarted the mac, turned on the USB network sharing
feature on the android phone and plugged the phone in USB. It works!

[HoRNDIS]: http://joshuawise.com/horndis
