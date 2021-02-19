---
layout: post
title: Zephyr - Completely Out of Tree
---

When building an embedded application, there are two routes that get presented
to developers on how to move forward.  The traditional path is to develop what
is known as a "bare metal" application, allowing a product to create it's own
destiny.  Alternatively, a development team can take on the task of enabling an
embedded RTOS allowing the team to focus on the actual application instead of
infrastructure.  But this isn't about bare metal vs an RTOS.  You can do a
search for that very topic and get plenty of reasons for why to choose one over
the other.

I have chosen to use an RTOS.  Specifically, the
[Zephyr](https://zephyrproject.org) RTOS.  Given my history with the project,
that may seem like an obvious choice, but the project has evolved vastly
different from when I was last involved with it.

Before starting a Zephyr application, you have the option of developing your
application with different source code layouts they call "star topologies".  In
the Zephyr 2.5.0 release, the documentation on [star
topology](https://docs.zephyrproject.org/2.5.0/guides/west/workspaces.html#topologies-supported)
goes into some good details about each of the topologies.  I like to think of
them in a slightly different way:

- T1 sets the Zephyr source as the center of the universe through an in-tree
  development model that will be a fork of the Zephyr project.
  - Good: If you believe you will need to modify the Zephyr sources to support
    your application and do not want to make those changes public
  - Bad: Updating the kernel becomes challenging if you are not careful about
    those in-kernel changes
- T2 sets your application out of the tree, making it the center of the
  universe.
  - Good: Draws the line for the separation of church and state pretty clearly
  - Good: You can set the Zephyr version to track an internal fork or the
    external public kernel
  - Bad: The mix of `git` for your application code and `west update`for the
    supporting code can easily confuse many developers
- T3 uses a repo that details the infrastructure only as the center of the
  universe.
  - Good: Draws the line for the separation of church and state pretty clearly
  - Good: You can set the Zephyr version to track an internal fork or the
    external public kernel
  - Good: Once the infrastructure layout is complete, the need to update is
    minimal
  - Bad: The mix of `git` for your application code and `west update`for the
    supporting code can easily confuse many developers
  - Bad: It is easy for developers to forget to update the infrastructure repo
    and think everything is correct.

If you're taking the time to use an RTOS, you likely have already begun thinking
about the possibility of needing to maintain and/or update your RTOS as time
goes on.  In my view, this eliminates the possibility of working with the T1
configuration as it provides the most hassle when upgrading later.

Torsten Tejlmand Rasmussen at Nordic Semiconductor

[Jared Wolff](https://twitter.com/jaredwolff) wrote up his experience on [How to
Build Drivers for
Zephyr](https://interrupt.memfault.com/blog/building-drivers-on-zephyr) on
Interrupt.  

While looking at the documentation, I found the details about keeping 

- Out Of Tree
  - Application
  - Drivers

- What happens when you go completely out of tree?
  - BOARD_ROOT
  - DTS_ROOT
  - SOC_ROOT

https://docs.zephyrproject.org/latest/application/index.html#custom-board-devicetree-and-soc-definitions
