---
layout: post
title: Setting up Home Assistant
---

Decided it was time to try making my home more "smart", and really wanted to
try out the newer Matter and Thread standards.  The goal was to own the data
connection in the house and not depend upon Apple, Google, Amazon, or Samsung
for the features.  Thus I decided to use Home Assistant.

These are the adventures of a simple guy trying to set up Home Assistant for a
first time.

# Equipment

A list of equipment used for this:

- A Raspberry Pi 4 - I had one laying around
- 64 GD SD Card
- makerdiary nrf52540 USB dongle
- eve door sensor Matter enabled

## Setting up RPi

Setting up the Raspberry Pi wasn't too difficult.  The `rpimager` has options
for flashing a ready-built Home Assistant OS install.

Once done flashing the OS, add the SD card to the RPi and boot.  The OS is
designed to not really need you to do anything on the console, which is a nice
little change from what I was expecting.  All the work is done through the web
UI.  

Please note, if you try to run the system without an wired connection, it's
going to take some additional work on the console for you.  From reading the
forums, most uses of HA-OS find wireless to be too inconsistent, and thus the
default is for a wired only connection.  Even asking on the forums seems to get
a bit of RTFM attitude.  

## Setting up makerdiary

Getting the makerdiary up and working took a bit more work.  Thankfully there are
some pretty easy cheats on this.

### Getting the latest firmware

Getting a recent firmware can be grabbed from https://github.com/ArthFink/nrf52840-OpenThread

Why do I, a Zephyr developer, resort to this?  I honestly didn't want to deal with installing
the nRF SDK.  I likely could have figured out the native Zephyr build requirements, but this
project was about making a "smart home" not rolling my own everything.

This file though does not generate a UF2 boot image, which is what is needed by
the makerdiary. Instead following the directions at
https://wiki.makerdiary.com/nrf52840-mdk-usb-dongle/programming/uf2boot/#updating-the-application-firmware
 leads to some nice steps on how to convert the HEX file into a UF2 file.

### Getting uf2conf

In case the steps change:

```
python3 -m pip install --pre -U git+https://github.com/makerdiary/uf2utils.git@main
```

### Converting the Firmware image

The GitHub firmware downloads as a ZIP with several files in it:

```
ot-cli-ftd-UART 
ot-cli-ftd-USB
ot-rcp-USB.hex
ot-cli-ftd-UART.hex
ot-cli-ftd-USB.hex
ot-rcp-USB
```

None of these are UF2 files, which is what the makerdiary needs. Running the following
command seems to work:

```
uf2conv -f 0xADA52840 -c -o application.uf2 ot-cli-ftd-USB.hex
```


