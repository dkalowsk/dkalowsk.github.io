---
layout: post
title: Creating a Zephyr-based Product
---

In creating this post, my goal is to create a readable work log for the
decisions and processes used to create a completely custom application module.
This includes a custom board, soc, drivers, and application within the Zephyr
framework.  Some of the decisions in this post reflect upon my experience
setting up a small team to develop those pieces collaboratively, while enabling
the team to scale out as the project progresses.  These assumptions may not
apply to your environment.

# Intro Ramblings

Building an embedded application, there are typically two routes that get
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
that may seem like an obvious choice.  But the project has evolved vastly
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
them in a slightly simplified way:

- T1 sets your fork of the Zephyr source as the center of the universe.
- T2 sets your application git repo as the center of the universe.
- T3 uses a repo that details the infrastructure only as the center of the
  universe and everything else revolves around it.

In my experience, the T2 model is the right solution.

### Why Not T1?

The T1 model is extremely seductive, as it plays to my preference for a monorepo
(temporarily ignore Zephyr modules), while limiting a developer's distractions
from tooling.  The challenging part is in keeping developers out of changing the
Zephyr kernel when they implement a sub-component incorrectly.  For example,
instead of figuring out why a build isn't finding a specific include path, you
can just hack it in with CMake because it is just "right there always".  Also,
this is a post about out of tree and T1 is an in-tree development.

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
State by setting a application repo as the center of the world.  Additionally like
T3, it keeps the process of updating to a newer kernel relatively easy.  Similar
to T1, with a little additional care and grooming, you can easily make your own
application a "monorepo" enabling cross-cutting changes, and limiting the
tooling knowledge, similar to T1.

The biggest downfall I have found to this setup is if you depend upon `ctags` or
`cscope` for your development process.  For developers that use either, I point
out the value of the `compile_commands.json` that can be generated through CMake
and the use of a language server (for example, `clangd` or `ccls`).  Then again,
VSCode does just fine with everything provided you open in the top directory.
Alternatively, if you use a vim varient, you can setup
[gutentags](https://github.com/ludovicchabant/vim-gutentags) to generate from a
custom
[root](https://github.com/dkalowsk/dotfiles/blob/73b682dde8c26ee610b13ca81ccdd6c0fef6c689/vim/.vimrc#L611)
as well (other tooling may do the same).

## Create Custom Application

I tend to want to see results quickly and iterate on them.  To help achieve
this, I like to start projects by picking a Zephyr supported board that uses
the SoC I expect to work with.  For discussion in this post, I will select a
device with the Arm Cortex M4 in it.

An additional great benefit is if it has many of the same devices attached to
it, but this is not a hard requirement.  I expect that over the life of
development these will all be replaced anyways.

Once you have a selected device and board selected, I copy over the Zephyr
hello_world sample application.  Additionally because I am lazy, I setup a
`build.sh` script that will do the job of cleaning up any artifacts while also
calling `west build` with any arguments I need.  It doesn't have to be something
fancy, and looks similar to this:

```bash
#!/usr/bin/env bash

set -o errexit
set -o pipefail
set -o nounset

if [[ -n "${DEBUG_SCRIPT:-}" ]]; then
    set -o xtrace
    export PS4='+ ${BASH_SOURCE:-}:${FUNCNAME[0]:-}:L${LINENO:-}:   '
    export
fi

readonly ZEPHYR_SDK_INSTALL_DIR=${ZEPHYR_SDK_INSTALL_DIR:-"/opt/toolchains/zephyr-sdk-0.12.3"}
readonly ZEPHYR_TOOLCHAIN_VARIANT=${ZEPHYR_TOOLCHAIN_VARIANT:-zephyr}

PROJECT_REPO=$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)
readonly OUTPUT_DIR="${PROJECT_REPO}/output"
TOP_DIR=$(dirname "$PROJECT_REPO")

_project_clean()
{
    pushd "$PROJECT_REPO" > /dev/null

    echo "clean:"
    rm -rf "${OUTPUT_DIR}"
    echo "done"

    popd > /dev/null
}

_project_build()
{

    export ZEPHYR_TOOLCHAIN_VARIANT
    export ZEPHYR_SDK_INSTALL_DIR

    echo "build:"

    pushd "$PROJECT_REPO/applications/hello_world" > /dev/null

    west build -t build \
        -d "${OUTPUT_DIR}/hello_world" -- .

    popd > /dev/null
}

_project_clean

_project_build
```

I can't recall where it was, or who it is attributed to, but there is something
to keep in mind when you start down this path of using a bash script.  Bash is
the language you die with, it never goes away.  The script only grows in
complexity as you find more uses for it.

Use this method at your own risk.  I like this method simply because I can give
all the other developers on a project (or a CI) a single consistent interface to
use.

Now all I need to do is:

```bash
#
# Generally build and clean the project
./build.sh

#
# If you think there is an error in the script:
DEBUG_SCRIPT=1 ./buildh.sh
```

Lastly we need a west manifest file.  Any testing done here is with 0.9.0 and
later versions may not reflect the same ability.  The west manifest should be
relatively simple:

```yaml
manifest:
  remotes:
    - name: github
      url-base: https://github.com
  projects:
#
# Zephyr Specific defines
#
    - name: zephyr
      remote: github
      repo-path: zephyrproject-rtos/zephyr
      import:
        name-whitelist:
          - cmsis
          - mbedtls
      west-commands: scripts/west-commands.yml
      revision: v2.5.0

  self:
    path: my_external_application
```

## Create A Custom Board

Once we have a basic hello world working, next step is to create a custom board.  
  - BOARD_ROOT

## Create A Custom DTS
  - DTS_ROOT

## Create A Custom SOC
  - SOC_ROOT

## Add Custom Drivers

Now it is time to add your custom drivers for the platform.  Instead of trying
to write something useful, I will point you towards [Jared
Wolff's](https://twitter.com/jaredwolff) excellent write up on his experience
titled [How to Build Drivers for
Zephyr](https://interrupt.memfault.com/blog/building-drivers-on-zephyr) on
Interrupt.  He does an excellent job describing what needs to be done.

## Convert To A Module

- What happens when you go completely out of tree?

https://docs.zephyrproject.org/latest/application/index.html#custom-board-devicetree-and-soc-definitions

https://www.diva-portal.org/smash/get/diva2:1352440/FULLTEXT01.pdf

# Thanks To

Really want to thank Torsten Tejlmand Rasmussen at Nordic Semiconductor for his
help on solving some of the bizarre warnings run into while getting the build
environment described here setup.  His help was the inspiration to writing this
entry to document the changes needed to get the world setup.

