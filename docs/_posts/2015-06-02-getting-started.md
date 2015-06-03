---
layout: page
title: "Getting Started"
category: tut
date: 2015-06-02 02:20:46
---

### Table of Contents

* [Setting up Your Environment](#environment)
* [Talking to a Networked "Thing": Hello World](#helloworld)



**[Note] This tutorial assumes you have basic Unix knowledge**



### <a name="environment"></a> Setting up Your Environment

There are two ways of establishing a programming environment for the FireStorm:

* installing the environment directly on an Ubuntu system
* using a virtual machine

Regardless of how you set up the environment, you should end up with a folder called `workspace` that contains the following:

* `contrib`: tracks the [contrib repository](https://github.com/SoftwareDefinedBuildings/ioet_contrib) and contains
  library and application code contributed by the members of the IOET class for public use.
* `toolchains/stormport`: tracks the [Storm TinyOS port](https://github.com/SoftwareDefinedBuildings/stormport/tree/kernel0/) repository and
  provides the kernel for the motes
* `toolchains/storm_elua`: tracks the [Storm userland](https://github.com/SoftwareDefinedBuildings/storm_elua) repository

#### Local FireStorm Environment

Bootstrapping the FireStorm programming environment has been tested on 64-bit Ubuntu systems, and while
it may work on other Debian-based Linux distros, it is probably easier in that case to use the VM (see section
below).

To bootstrap a local programming environment:

```
git clone https://github.com/SoftwareDefinedBuildings/firestorm_environment workspace
cd workspace
sudo ./bootstrap.sh
```

#### Virtual Machine


First, download the latest version of [VirtualBox](https://www.virtualbox.org/wiki/Downloads) for your platform, then download
the class VM (LINK COMING). Import the virtual appliance, and make sure to check "Reinitialize the MAC address of all network cards":

**TODO: add link to download the VM**

[{% image 400xAUTO virtualbox_reinitialize.png %}]( {{ site.url }}/assets/virtualbox_reinitialize.png )

The username for the virtual machine is "oski" and the password is "bear".

Once the VM has booted, open up a terminal and verify that you are in the `/home/oski/workspace` directory. 

Unlike working with a local native environment, plugging the FireStorm into the USB port on your
computer will not automatically make it availble to the virtual machine. Whenever a FireStorm is
plugged in, you will need to pass through the USB device to the virtual machine by selecting it from
the drop down list of USB devices. The FireStorm will likely be the only FTDI device in your USB
list, so it should be easy to identify.

[{% image 400xAUTO virtualbox_usbdevice.jpg %}]( {{ site.url }}/assets/virtualbox_usbdevice.jpg )

This needs to be done every time a FireStorm is plugged in, even if you had previously enabled the
passthrough and then disconnected. If you see an error that ends with `pylibftdi._base.FtdiError:
device not found (-3)`, then you probably forgot to enable the passthrough.

In certain cases, we have found that certain other devices plugged into your computer may cause
VirtualBox to lose its mind. Try unplugging any dongles or external devices if you experience
unusual behaviour.

**TODO: does the VM come with `https://github.com/SoftwareDefinedBuildings/ioet_contrib` pre-cloned?**

### <a name="helloworld"></a> Talking to a Networked "Thing": Hello World

The FireStorm attaches via USB Micro to your computer. Plug in your FireStorm (remember to enable
the passthrough if you are on the VM) and type the following into the terminal to check if your
device is attached.

```
$ sload -V ping
Device responded
```

`sload` is the StormLoader tool, which is documented [here](http://storm.rocks/ref/sloader.html),
and should already be installed.

To connect to the FireStorm's serial port, we use the `sload tail` command. There are two
invocations of this command that we will use:

```
sload tail -n # watch the output of the storm but don't reset it
sload tail -i # reset the storm and attach an interactive console
```

We will explain the `build.lua` file in a moment, but for now, let's use it to run a simple "Hello
World" program on our mote. The `build.lua` file should be in your top-level `workspace` directory.
Verify that it contains the following contents:

```lua
#!/usr/bin/env lua

-- Keep the comments in your version of the build file. We omit them here for brevity
autorun = "contrib/app/helloworld.lua" --<< EDIT ME
libs = { --<< EDIT ME
    cord    = "contrib/lib/cord.lua",
}
autoupdate = true
reflash_kernel = true
kernel_opts = {
    quiet = true, -- if set to false, you will see kernel debug messages
    eth_shield = false -- set to true to enable the ethernet shield 
}
dofile("toolchains/storm_elua/build_support.lua")
go_build()
```

and then build your FireStorm by running

```
./build.lua
```

Depending on what was on your FireStorm prior, this may take anywhere from a few seconds to a little
over a minute (don't worry -- it won't always be this long). When the mote is finished flashing, you
should see some output denoting the node ID, IP address and kernel version, and then the string
"hello world" should print out once a second.

```
[SLOADER] Attached
SERIAL NUMBER 0x3007
IPAddress - Setting global address: 2001:470:1234:2:212:6d02::3007
Booting kernel 4.0.3.0 (25b11aab07a0865122330531fe728be541fd499a)
hello world
hello world
```
