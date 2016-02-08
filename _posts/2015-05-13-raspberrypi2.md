---
layout: post
title: raspberrypi2
categories: []
tags: []
date: 2015-05-13 19:53:08-04:00

---

Refs:

* http://www.praj.com.au/post/44076969942/bunzip2-gives-input-file-has-1-other-link
* https://ubuntu-mate.org/raspberry-pi/
* https://www.raspberrypi.org/downloads/
* http://buyapi.ca

## The board

Finally picked up a raspberry pi 2.  Got it at http://buyapi.ca which is a reseller of a bunch of items and accessories for electronic gadettry :]

buyapi is great! if you live in Ottawa you can pick up and save the delivery charges.  I picked up:

* raspberry pi 2
* raspberry pi 2 case (black) cost around 10$
* wirless N usb dongle with extender antenna (300mps) ~15$
* mopi - a battery mobile support board for the raspberry header (has i2c support, etc...)


## The ubuntu image

With the pi alot faster (4core), and beastier in RAM (1GB) - there is an ubuntu image offering supported by Rohith Madhavan.

Installing the image was pretty straight forward - had to use the bzcat instead of bunzip2 to decompress the image (for some reason).  Thanks to www.praj.com.au for the pointer against the strange "has 1 other link" coming out of bunzip.  see Refs above for the instructions.
