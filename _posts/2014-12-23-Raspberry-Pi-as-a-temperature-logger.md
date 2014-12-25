---
Title: Raspberry Pi A+ as a temperature logger
---

I was looking for some time for a good reason to buy a [Raspberry Pi]. Unfortunately
I already have a nice nettop box that is doing its job very well as a media player
with [XBMC] (soon Kodi) plus a few services (irc bot, file sharing, …)
Ok, it is not as silent and more power greedy than I'd wish, but I can't just
throw it away…

But I finally found _the good reason_ I was looking for!

I want to log the temperature of a house several times a day during several weeks.
The house has power but no internet access. Ideally, it would be able to log the
temperature inside and outside the house. I've looked for a ready to use tool
that could do the job at a low cost, but haven't found anything really convincing.

This is the kind of task the Raspberry Pi can do very easily,
with the help of a [DS18B20] sensor.

## Raspberry Pi A+

So I bought:

* a [Raspberry Pi A+] — I don't need any USB port or ethernet connection,
256MB of RAM is more than enough, and the very low power consumption is
always nice to have,
* two [DS18B20] sensors — one is waterproof with a cable and the other is not,
* and a 4700Ω resistors.

I already had:

* a spare micro SD card,
* a (lot of) USB power supply,
* a box,
* four screws,
* an old IDE cable to connect to the GPIO,
* a switch from an old phone headset to use as shutdown button.

Total cost: less than 40€.

The system, a [Raspbian] Wheezy 2014-09-09, has been installed without any problem
— I even been able to configure it with the [bépo] keyboard layout during the setup. Great.

Here is how it looks:

[![open](https://farm9.staticflickr.com/8597/15902472707_2ba3e26b76_n.jpg)](https://flic.kr/p/qefk6V)
[![open](https://farm8.staticflickr.com/7520/15465912444_9048f667aa_n.jpg)](https://flic.kr/p/pyEQWQ)
[![open](https://farm8.staticflickr.com/7472/15902126539_3f640b6350_n.jpg)](https://flic.kr/p/qedycv)

Not a peace of art, but it should do the job!

## Temperature sensors

I've soldered the pull up resistor on the IDE cable between the pin 1 and 7,
and the pin 1, 2 and 3 of the [DS18B20] on the pin 9, 7 and 1 of the GPIO respectively.

The temperatures are read through the `sysfs` interface, that requires to load
two extra modules in the kernel. To do that, I've added these two lines in the
`/etc/modules` file:

~~~
w1-gpio
w1-therm
~~~

A python script run by cron every ten minutes reads the temperature from the
sensors and writes the output to a file:

~~~python
#!/usr/bin/env python

import glob, os
from datetime import datetime

out = open("temp.csv", "a")
date = str(datetime.now())
ds = glob.glob('/sys/bus/w1/devices/28-*/w1_slave')
for d in ds:
    name = d.split("/")[-2]
    temp = str(float(open(d).read().split()[-1][2:])/1000)
    print >> out, ";".join([date, name, temp])
out.close()
~~~

The date is the same for all the sensors, to make easier to compare the temperatures.
When doing fast acquisition or using many sensors, it may be a better idea to get
the real date for each sensor, as each sensor take a second or so to read.

The crontab entry is:

    */10 * * * * /home/pi/temp.py

Here are a few lines from the generated `temp.csv` file:

~~~
2014-12-21 16:20:01.575703;28-001450767aff;21.875
2014-12-21 16:30:02.006573;28-001450767aff;21.625
2014-12-21 16:40:02.116337;28-001450767aff;22.125
~~~

There is a single sensor for now – I'm still waiting for the waterproof sensor. And
here is a chart with the data on a few hours:

![Temperatures]({{ site.url }}/images/RPi-temperatures.png)

The data was manipulated with R:

~~~R
data <- read.table("temp.csv", sep=";", col.names=c("date", "sensor", "temperature"))
data$date <- as.POSIXct(as.character(data$date))
plot(temperature ~ date, data)
~~~

## Shutdown button

Without a screen, keyboard, or any kind of connection, it is not possible
to shutdown the Raspberry Pi properly. Fortunately it has many input on the GPIO
that can be used just for that. I've soldered a switch between the pin 39 (GPIO21)
and the pin 40 (ground).

A python script launched by `/etc/rc.local` just waits for the input to change, and
then initiates the clean system shutdown.

~~~python
#!/usr/bin/env python

import RPi.GPIO as GPIO
import os

GPIO.setmode(GPIO.BCM)
GPIO.setup(21, GPIO.IN, pull_up_down=GPIO.PUD_UP)
GPIO.wait_for_edge(21, GPIO.BOTH)
os.system("logger 'power button pressed'")
os.system("halt")
~~~

Note that it wasn't possible to use the pin number with the RPi.GPIO module
installed by default on my raspbian. It failed with the error:

    ValueError: The channel sent is invalid on a Raspberry Pi

I had to run `setmode(BCM)` instead of `setmode(BOARD)` to
avoid that problem, and use the GPIO number instead of the pin number:

[![Raspberry Pi A+/B+ GPIO]({{ site.url }}/images/GPIO-200px.png)](http://www.element14.com/community/servlet/JiveServlet/previewBody/68203-102-6-294412/GPIO.png)

The switch, once pushed, connects the GPIO21 to the ground, so the GPIO21
has to be configured with a pull up resistor. When reading the value with
`input(21)`, the value returned is `1` when the switch is not pressed and `0`
when it is pressed. This may seem a bit counter intuitive, but only describe
the current state on the GPIO21: `1` for the 3.3V from the pull up resistor,
`0` for the 0V from the ground.

Instead of reading the value from time to time and triggering the shutdown when
the value is `0`, the script just use `wait_for_edge(21, BOTH)`, that detects when the
value change on the pin and exits when it happens.

A message is sent to `syslog` with the `logger` command to keep a track that the
button was pressed.

The line in the file `/etc/rc.local` is just:

    /home/pi/shutdown.py &

The `&` is mandatory to let the process run in the background.

## Lack of clock battery and network

I already knew the Raspberry A+ has no network interface. This is fine for this
application, because the house has no internet connection.

This is not convenient for the setup though, where everything must be done
with the screen and the keyboard, or transfered to the SD card before booting
the Raspberry Pi.

Another problem I overlooked is the lack of battery for the clock. This mean
the Raspberry Pi can't keep its clock running when it is not powered. The [Raspbian]
deals with that by keeping the date unchanged since the last time it went down.
With the network, the date and time are updated with NTP. Without network,
the only way to update the date is to use a command like:

    sudo date -s "december 21 2014 17:29"

Obviously, the only (easy) way to run that is with a screen and a keyboard,
and I won't have any when I'll plug the Raspberry Pi in that house.

Again, a network interface would have helped, as I could have used my laptop
to set the date through SSH.

I could buy a USB WIFI adapter, but the total cost would then be quite close
to the price of the Raspberry B+, and WIFI is a lot more painful to configure
than a simple ethernet connection.

I guess I'll just have to set the approximate date at which it will boot before unplugging
it.

A power failure will also shift the time recorded with the temperature. I'm not
sure how important it can be, but to be sure to know when a power failure
occur, I've added a command that renames the temperature file when the
Raspberry reboots:

    mv temp.csv temp.csv.`date +%Y-%m-%dT%H.%M.%S`

This is run by cron with the special string `@reboot`.

So a Raspberry A+ is fine for the application itself, but the network interface
would have helped quite a lot to configure everything and experiment with the
temperature logging. I think I'll buy a B+ for my next project!

That's it!

Now I wait for the weatherproof sensor to arrive. In the mean time, I let the logger
accumulate some real temperatures that I can use to try to extract some interesting
informations.


[Raspberry Pi]: http://www.raspberrypi.org/products/
[Raspberry Pi A+]: http://www.raspberrypi.org/products/model-a-plus/
[XBMC]: http://kodi.tv/
[DS18B20]: https://learn.adafruit.com/adafruits-raspberry-pi-lesson-11-ds18b20-temperature-sensing?view=all
[Raspbian]: http://www.raspbian.org/
[bépo]: http://bepo.fr
