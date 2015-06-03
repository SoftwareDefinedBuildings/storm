---
layout: page
title: "Getting Started"
category: tut
date: 2015-06-02 02:20:46
---

### Table of Contents

* [Setting up Your Environment](#environment)



**[Note] This tutorial assumes you have basic Unix knowledge**



### <a name="environment"></a> Setting up Your Environment

There are two ways of establishing a programming environment for the FireStorm:

* installing the environment directly on an Ubuntu system
* using a virtual machine

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

[{% image 400xAUTO virtualbox_reinitialize.png %}]( {{ site.url }}/assets/virtualbox_reinitialize.png )

The username for the virtual machine is "oski" and the password is "bear".

Once the VM has booted, open up a terminal and verify that you are in the `/home/oski/workspace` directory. 

Unlike working with a local native environment, plugging the FireStorm into the USB port on your computer will not automatically
make it availble to the virtual machine. Whenever a FireStorm is plugged in, you will need to pass through the USB device to the
virtual machine by selecting it from the drop down list of USB devices. The FireStorm will likely be the only FTDI device
in your USB list, so it should be easy to identify.

[{% image 400xAUTO virtualbox_usbdevice.jpg %}]( {{ site.url }}/assets/virtualbox_usbdevice.jpg )

This needs to be done every time a FireStorm is plugged in, even if you had previously enabled the passthrough and then disconnected.


