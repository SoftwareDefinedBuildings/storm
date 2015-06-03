---
layout: page
title: "Getting Started"
category: tut
date: 2015-06-02 02:20:46
---

### Table of Contents

* [Setting up Your Environment](#environment)
* [Hello World](#helloworld)
* [Talking to a Networked "Thing"](#talkingtoit)



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

### <a name="helloworld"></a> Hello World

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
autorun = "contrib/app/helloworld.lua"
libs = {
    cord    = "contrib/lib/cord.lua",
}
autoupdate = true
reflash_kernel = true
kernel_opts = {
    quiet = true,
    eth_shield = false
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

Press Ctl-C to exit.

### <a name="talkingtoit"></a> Talking to a Networked "Thing"

The `build.lua` file is fairly well self-documented, but there a few options worth pointing out:

* `autorun = "relative/path/to/file.lua"`: 
    This specifies the main file that is run when the FireStorm boots. Often this file will not end,
    as it enters the cord scheduling loop. If you omit this parameter, the mote will boot into a
    primitive Lua shell (without coroutines).
* `libs = {libname="relative/path/to/library.lua", libname2=...}`:
    This table specifies the files that will be programmed into ROM on the device. You can `require`
    these libraries later to use them
* `reflash_kernel = true|false`:
    This specifies whether or not the kernel should be updated when `build.lua` is run. Enabling
    this slows down the build a little, so it is recommended to set this to true every now and then
    so that the kernel stays up to date.
* `quiet = true|false`:
    This should be `true` by default, but disabling quiet-mode will give you all of the kernel
    messages that can be helpful for more complex debugging tasks.

Now, let's edit the `build.lua` file so that we can get an interactive prompt that supports
coroutines. First, change the build file so that the application that runs when the FireStorm boots
is `shell.lua`, in the `contrib/app` directory. Next, add the `stormsh.lua` library from
`contrib/lib` lua`. 

Your build file should now look like the following (excluding comments):

```lua
#!/usr/bin/env lua
autorun = "contrib/app/shell.lua" -- we added this line
libs = {
    cord    = "contrib/lib/cord.lua",
    stormsh = "contrib/lib/stormsh.lua", -- we added this line
}
autoupdate = true
reflash_kernel = true
kernel_opts = {
    quiet = true,
    eth_shield = false
}
dofile("toolchains/storm_elua/build_support.lua")
go_build()
```

Now, run `build.lua` and you should be dropped into an interactive Lua shell. 

```
[SLOADER] Attached
SERIAL NUMBER 0x3007
IPAddress - Setting global address: 2001:470:1234:2:212:6d02::3007
Booting kernel 4.0.3.0 (25b11aab07a0865122330531fe728be541fd499a)
stormsh> 
```

Lua is a very simple language, but provides nice mechanisms for concurrency and can also be embedded
in C or have C embedded within it. The Lua shell is a great interactive debug and test environment,
but is also a rapid prototyping vehicle.

There are a few Lua programming resources we recommend:

* http://thomaslauer.com/download/luarefv51.pdf
* http://learnxinyminutes.com/docs/lua/
* http://luatut.com/crash_course.html

A few things to keep in mind:

* Variables need not be defined; use of an undefined variable simply returns nil.  Assignment
  defines it.  Lua throws very few errors.  (In the embedded world most things are unattended so no
  one will notice if you scream, so you need to check and handle the exceptions, rather than throw
  them to the “user”
* Variables have global scope unless explicitly declared local! Our advice - declaring everything as
  local unless you really wanted it accessible everywhere.  Especially in concurrent code this WILL
  bite you. And these kind of bugs are hard to find.
  Conditionals treat nil as false, but not 0.
* `function foo() ...` is just syntactic sugar for `foo = function() ...`
* Statements and clauses are explicitly closed.  Line breaks and such don’t matter.
* The central data structure primitive is the table - a sequence of <key,val> like a dictionary.  It
  is used for lists, arrays, modules, objects and just about everything else.
  For example:

  ```lua
  x = {1,2,['apple']='red',['foo']=function() print('foo') end}
  print(x) -- what does this return?
  for k,v in pairs(x) do print(k,v) end -- iterate over all key/value pairs
  x[2] -- treat x like an array
  x.red -- or like a struct (also x["red"])
  x.foo() -- or like an object with a foo() method
  ```
* Ctl-D or Ctl-C should exit you from the terminal. Use `sload tail -i` to reattach. Note that no
  data you type into the shell is persisted on the node , so disconnecting and reattaching will
  delete any variables you had previously declared.


The kernel defines a set of syscalls (documentation is [here](https://github.com/SoftwareDefinedBuildings/storm_elua/wiki))
that we will cover in-depth later. Read the documentation on the `storm.io` module, which we can use
to turn an LED on and off (LED itself is located on the far end of the FireStorm from the USB
cable). To do this, we configure the GP0 pin on the storm as an output, and then write 1 to it.

From the shell:

```lua
stormsh> storm.io.set_mode(storm.io.OUTPUT, storm.io.GP0)
stormsh> storm.io.set(1, storm.io.GP0) -- can also use storm.io.HIGH/LOW instead of 1/0
```

Now, try turning the LED off.
