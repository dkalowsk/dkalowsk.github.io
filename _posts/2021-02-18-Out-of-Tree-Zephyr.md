---
layout: post
title: Creating a Zephyr-based Product
---

In creating this post, my goal is to create a readable work log for the
decisions and processes used to create a completely custom board, soc, and
application within the Zephyr framework.  A lot of this post is based upon my
experience setting up a team to develop those pieces.

## Intro Ramblings

When building an embedded application, there are typically two routes that get
presented to developers on how to move forward.  The traditional path is to
develop what is known as a "bare metal" application, allowing a product to
create it's own destiny.  Usually this is easily identified by the traditional
big while loop, although it is not required.  Alternatively, a development team
can take on the task of enabling an embedded RTOS allowing the team to focus on
the actual application instead of infrastructure.  But this isn't about bare
metal vs an RTOS.  You can do a search for that very topic and get plenty of
reasons for why to choose one over the other.

I have chosen to use an RTOS.  Specifically, the
[Zephyr](https://zephyrproject.org) RTOS.  Given my history with the project,
that may seem like an obvious choice, but the project has evolved vastly
different from when I was last involved with it, and I have found embracing some
of the changes challenging.

One area I greatly appreciate is the effort the Zephyr developers take into
enabling completely out of tree builds.  When you are building an entire
application (drivers, boards, SoC, etc) this very easily becomes an essential
requirement for development.

## What is Out Of Tree?

Before starting a Zephyr application, you have the option of developing your
application with different source code layouts they call "star topologies".  In
the Zephyr 2.5.0 release, the documentation on [star
topology](https://docs.zephyrproject.org/2.5.0/guides/west/workspaces.html#topologies-supported)
goes into some good details about each of the topologies.  I like to think of
them in a slightly different way:

- T1 sets your fork of the Zephyr source as the center of the universe.
- T2 sets your application git repo as the center of the universe.
- T3 uses a repo that details the infrastructure only as the center of the
  universe and everything else revolves around it.

In my experience, the T2 model is the right solution.

### Why Not T1?

The T1 model is extremely seductive, as it plays to my preference for a monorepo
(essentially, temporarily ignore Zephyr modules), while limiting a developer's
distractions from tooling.  The challenging part is in keeping developers out of
changing the Zephyr kernel when they implement a sub-component incorrectly.  For
example, instead of figuring out why a build isn't finding a specific include
path, you can just hack it in with CMake because it is just "right there
always".  Also, this is a post about out of tree and T1 is an in-tree
development.

### Why Not T3?

The T3 model is the model I want to be the right solution for everything.  It
keeps the layout clean, with a clear division of Church (Zephyr kernel) and
State (Application), and an easy way to expand your project as needed.
Depending upon how granular you make it, the T3 topology can make cross-cutting
changes really difficult.  Working with other developers to sustain  development
velocity becomes another challenge.  Ideally, you always want to limit the
additional cognitive load on development teams, allowing them to stay focused on
the deliverables.  In my experience, the T3 model suddenly becomes unsustainable
when developers forget the proper update flow (does this need a `west update` or
`git pull`?), and they will forget at some point.

Basically this comes down to there are too many moving parts for me in this
topology.  Those moving parts are going to need support, and I don't have the
ability to support that.

### Why T2?

For me, T2 solves most of the limitations from T1 and T3.  Similar to T3, it is
possible to setup the world while maintaining a clear division of Church and
State by setting a application repo as the center of the world.  Additional like
T3, it keeps the process of updating to a newer kernel relatively easy.  Similar
to T1, with a little additional care and grooming, you can easily make your own
application a "monorepo" enabling cross-cutting changes, and limiting the
tooling knowledge, similar to T1.

The biggest downfall I have found to this setup is if you depend upon `ctags` or
`cscope` for your development process, as the tags .  For developers that use either, I point
out the value of the `compile_commands.json` that can be generated through
CMake.  Then again, VSCode does just fine with everything provided you open in
the top directory.

## Create Custom Application

If you are like me, you tend to want to see results quickly and iterate on them.
To help achieve this, I like to start by picking a Zephyr supported device that
uses the SoC I am expected to work with.  Although a great benefit, it does not
need to have the same devices attached.

For the greater clarity in this post, I will select a device with the Arm Cortex
M4 in it.

Once you have a device selected, I copy over the Zephyr hello_world sample
application.  Additionally because I am lazy, I setup a `build.sh` script that
will do the job of cleaning up any artifacts while also calling `west build`
with any arguments I need


Create a git repo
Copy over Zephyrs hello world
Create a west.yml
Create a build.sh

Test

## Create A Custom Board
  - BOARD_ROOT

## Create A Custom SOC
  - SOC_ROOT

## Create A Custom DTS
  - DTS_ROOT

## Add Custom Drivers

[Jared Wolff](https://twitter.com/jaredwolff) wrote up his experience on [How to
Build Drivers for
Zephyr](https://interrupt.memfault.com/blog/building-drivers-on-zephyr) on
Interrupt.

## Convert To A Module

Torsten Tejlmand Rasmussen at Nordic Semiconductor


- What happens when you go completely out of tree?

https://docs.zephyrproject.org/latest/application/index.html#custom-board-devicetree-and-soc-definitions

https://www.diva-portal.org/smash/get/diva2:1352440/FULLTEXT01.pdf
