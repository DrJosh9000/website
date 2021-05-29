+++
title = "Library"
description = "Leverage periph to write Go applications"
+++

The complete API documentation, including examples, are at:

* [periph.io/x/conn/v3](https://periph.io/x/conn/v3)
* [periph.io/x/host/v3](https://periph.io/x/host/v3)
* [periph.io/x/devices/v3](https://periph.io/x/devices/v3)


# Introduction

`periph` is designed to be a package leveraged to abstract OSes and boards
differences to write an app once, run anywhere. It uses a driver registry to
efficiently load the relevant drivers on the board and OS it is running on. It
differentiates between drivers that _enable_ functionality on the host and
drivers for devices connected _to_ the host.

Most micro computers expose at least some of the following:
[I²C bus](https://periph.io/x/conn/v3/i2c#Bus), [SPI
bus](https://periph.io/x/conn/v3/spi#Conn), [GPIO
pins](https://periph.io/x/conn/v3/gpio#PinIO), [analog
pins](https://periph.io/x/conn/v3/analog),
[UART](https://periph.io/x/conn/v3/uart), I2S and PWM.

Note: not all of the above is implemented yet!

- The interfaces are defined in [conn](https://periph.io/x/conn/v3).
- The concrete objects _implementing_ the interfaces are in
  [host](https://periph.io/x/host/v3).
- The device drivers _using_ these interfaces are located in
  [devices](https://periph.io/x/devices/v3).

A device can be connected on a bus, let's say an LED strip connected over SPI.
In this case the application needs to obtain a handle to the SPI bus and then
connect the LED device driver to the SPI bus handle.


# Compatibility guarantee

`periph` follows [SemVer](https://semver.org) compatibility guarantee:

- Major version change (`v1.0` to `v2.0`) may introduce breaking changes.
- Minor version change (`v1.1` to `v1.2`) will be backward compatible.


# Initialization

The function to initialize the drivers registered by default is
[host.Init()](https://periph.io/x/host/v3#Init). It returns a
[driverreg.State](https://periph.io/x/conn/v3/driver/driverreg#State):

~~~go
state, err := host.Init()
~~~

[driverreg.State](https://periph.io/x/conn/v3/driver/driverreg#State) contains
information about:

- The drivers loaded and active.
- The drivers skipped, because the relevant hardware wasn't found.
- The drivers that failed to load due to an error. The app may still run without
  these drivers.

In addition, [host.Init()](https://periph.io/x/host/v3#Init) may return an
error when there's a structural issue, for example two drivers with the same
name were registered. This is a fatal failure. The package
[periph/host](https://periph.io/x/host/v3) registers all the drivers under
it.


# Connection

A connection [conn.Conn](https://periph.io/x/conn/v3#Conn) is a
**point-to-point** connection between the host and a device where the
application is the master driving the I/O.

A `Conn` can be multiplexed over the underlying bus. For example an I²C bus
[i2c.Bus](https://periph.io/x/conn/v3/i2c#Bus) may have multiple connections
(slaves) to the master, each addressed by the device address.


## SPI connection

An [spi.Conn](https://periph.io/x/conn/v3/spi#Conn) **is** a
[conn.Conn](https://periph.io/x/conn/v3#Conn). The reason is that spi.Conn
is locked on a specific CS line, so it is effectively treated as a
point-to-point connection and not as a bus.


## I²C connection

An [i2c.Bus](https://periph.io/x/conn/v3/i2c#Bus) is **not** a
[conn.Conn](https://periph.io/x/conn/v3#Conn). This is because an I²C bus is
**not** a point-to-point connection but instead is a real bus where multiple
devices can be connected simultaneously, like a USB bus. To create a
point-to-point connection to a device which does implement
[conn.Conn](https://periph.io/x/conn/v3#Conn) use
[i2c.Dev](https://periph.io/x/conn/v3/i2c#Dev), which embeds the device's
address:

~~~go
// Open the first available I²C bus:
bus, _ := i2creg.Open("")
// Address the device with address 0x76 on the I²C bus:
dev := i2c.Dev{bus, 0x76}
// This is now a point-to-point connection and implements conn.Conn:
var _ conn.Conn = &dev
~~~

Since many devices have their address hardcoded, it's up to the device driver to
specify the address.


## GPIO

[GPIO pins](https://periph.io/x/conn/v3/gpio#PinIO) can be leveraged for
arbitrary uses, such as buttons, LEDs, relays, etc.


# Examples

See [device/](device/) for various examples.


# Using out-of-tree drivers

Out of tree drivers can be seamlessly loaded by an application. The example
below shows a hypothetical driver located at github.com/example/virtual_i2c that
exposes an I²C bus over a REST API to a remote device.

This driver can be used with periph as if it were built into periph as follows:

~~~go
package main

import (
    "log"

    "github.com/example/virtual_i2c"
    "periph.io/x/conn/v3/driver"
    "periph.io/x/conn/v3/i2c"
    "periph.io/x/conn/v3/i2c/i2creg"
    "periph.io/x/host/v3"
)

type driverImpl struct{}

func (d *driverImpl) String() string          { return "virtual_i2c" }
func (d *driverImpl) Prerequisites() []string { return nil }
func (d *driverImpl) After() []string { return nil }

func (d *driverImpl) Init() (bool, error) {
    // Load the driver. Note that drivers are loaded *concurrently* by periph.
    if err := virtual_i2c.Load(); err != nil {
        return true, err
    }
    err := i2c.Register("foo", 10, func() (i2c.BusCloser, error) {
        // You may have to create a struct to convert the API:
        return virtual_i2c.Open()
    })
    // If this Init() function returns an error, it will be in the state
    // returned by host.Init():
    return true, err
}

func main() {
    // Register your driver in the registry:
    if _, err := driverreg.Register(driver); err != nil {
        log.Fatal(err)
    }
    // Initialize normally. Your driver will be loaded:
    if _, err := host.Init(); err != nil {
        log.Fatal(err)
    }

    bus, err := i2creg.Open("1")
    if err != nil {
        log.Fatal(err)
    }
    defer bus.Close()

    // Use your bus driver like if it had been provided by periph.
}
~~~
