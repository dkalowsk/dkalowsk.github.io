---
layout: post
title: Setting up Quantum Fiber
---

Moving houses always introduces challenges, one of those challenges is updates
to home internet providers. After having happily used CenturyLink internet for
years, the time has come to replace the service with the newer Quantum Fiber.

This means changes to the home networking!

# CenturyLink Setup

I never liked the idea of using CenturyLink's provided WiFi/router.  Something
about not being able to control my home networking really frustrated me.  Early
on the home network used an earlier version of the Synology RT2600ac
(https://www.synology.com/en-us/products/RT2600ac) to replace everything but
the ONT from CenturyLink.  And life was good with it for a long time.  Right up
until the COVID pandemic hit.  At that point, it was clear the router was choking
on the higher demand for video calls.

As it seemed Synology wasn't going to release another WiFi router soon, I looked at
using something different.  The Firewalla Purple
(https://firewalla.com/products/firewalla-purple) caught my eye as it was in Kickstarter
mode and likely to finish.  I joined up got the little purple device and had it replace
our Synology mixed with some WiFi mesh routers.  

This setup was later changed to using a Firewalla Gold
(https://firewalla.com/products/firewalla-gold-plus) for several years. Why the change?
I began taking the purple with me when I'd need to travel to setup a VPN back to home,
or enable my family to use a single WiFi point.

On the CenturyLink system, both of these were able to directly connect to the
ONT and use PPPoE credentials to control the network.  The key was to enable
VLAN tagging to 201.


# Quantum Fiber Setup

Quantum Fiber changed their interface to the C5500XK which is a mix of an
ONT and Router.  This means a couple of things:

  1. That it can and does auto tag everything for the VLAN.  On the Firewalla
     side this means no longer tagging the VLAN as the C5500XK can do this
     automatically.
  1. The system no longer uses PPPoE, so moving to DHCP is required.

Since it can be used as a router though we run into an issue of possibly having
a double NAT device, meaning unable to use the Firewalla device.

## Enabling Bridge Mode

Thankfully, other folks have figured out the secret here is enabling the C5500XK
into Transparent Bridge Mode.

The process for enabling Transparent Bride Mode should be easy, but took a few
tries to get the two devices talking.   The steps I used:

  1. Reset the power to the C5500XK
  1. Set your laptop to use ethernet and DCHP
  1. Plug an ethernet into the WHITE port on the C5500XK
  1. Plug the ethernet into your laptop
  1. Connect a browser to 192.160.0.1
  1. Use the credentials provided on the C5500XK to login
  1. Navigate to "Advanced Setup" -> "WAN Settings" -> "Select ISP Protocol"
     1. Change this from IPOE to "Transparent Bridging"
  1. Press Apply and reboot the C5500XK

Once these steps were done, any device plugged into the C5500XK would get a
public network IP.  Moving the Firewalla over to using this new network was
pretty simple.
