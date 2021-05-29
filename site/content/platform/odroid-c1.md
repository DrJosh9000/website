+++
title = "ODROID-C1"
description = "HardKernel board"
picture = "/img/odroid-c1.jpg"
+++

# Overview

The ODROID-C1, ODROID-C1+, and ODROID-C0 boards are well designed boards
originally designed for Android OS but works well with Linux based
distributions. These boards use an Amlogic S805 processor (called
"meson_8b" in the linux kernel). The functionality supported is:

- 2x I²C buses
- 1x SPI bus with 1x chip-enable
- 25x GPIO pins on the main J2 header


# Support

The ODROID-C1, ODROID-C1+, and ODROID-C0 boards are supported when running a
linux based distribution.

- `periph` is tested with a minimal (head-less) [Ubuntu 16.04
  build](https://odroid.in/ubuntu_16.04lts/)


## Drivers

- Headers driver lives in
  [periph.io/x/host/v3/odroidc1](https://periph.io/x/host/v3/odroidc1).
  It exports the main `J2` header, which is rPi compatible except for a couple
  of analog pins (which are not currently supported).
- sysfs driver lives in
  [periph.io/x/host/v3/sysfs](https://periph.io/x/host/v3/sysfs).


# Tips and tricks

The ODROID-C1+ is described on Hardkernel's web site:
[ODroid-C1+](https://www.hardkernel.com/shop/odroid-c1/).
The best reference for actually using the various I/O buses is the wiki:
[Hardware wiki](https://odroid.com/dokuwiki/doku.php?id=en:c1_hardware).

The drivers and some of their peculiarities are described in the Ubuntu section
of this [wiki page](https://odroid.com/dokuwiki/doku.php?id=en:odroid-c1#ubuntu).


# Configuration

## GPIO

Currently no package for memory-mapped I/O has been written for the Amlogic S805
processor, thus all gpio functions are implemented via
[sysfs](https://periph.io/x/host/v3/sysfs).

Interrupts on GPIO pins are limited to 8 pins when using rising or falling edges
and 4 pins when using both edges.


## I²C

By default, I²C is not enabled. Enable with: `modprobe aml_i2c`

I²C #1 is on pins 3&5, I²C #2 on pins 27&28. I²C #0 is not usable.

This needs to be done at each boot. A good location is to add the above into
`/etc/rc.local` before the `exit 0` statement.


## SPI

By default, SPI is not enabled. Enable with: `modprobe spicc`

Bus on pins 19, 21, 23 and chip enable pin 24.

This needs to be done at each boot. A good location is to add the above into
`/etc/rc.local` before the `exit 0` statement.


## 1-wire

1-wire driver is loaded by default. To free up GPIO #83 from 1-wire: `rmmod
w1-gpio`


# Buying

- The ODROID-C1 is directly distributed by [HardKernel](https://hardkernel.com).

_The periph authors do not endorse any specific seller. These are only provided
for your convenience._
