---
layout: page
title: "FireStorm Border Router"
category: ref
date: 2015-06-02 21:38:16
---

### Table of Contents

* [Hardware Setup](#hardware)
* [Software Setup](#software)
* [Network Configuration](#network)
* [Troubleshooting](#troubleshooting)


### <a name="hardware"> Hardware Setup

The FireStorm Border Router (FSBR) uses the [W5200 Ethernet Shield](http://www.seeedstudio.com/wiki/images/e/e7/W5200_Datasheet.pdf).
There are 4 wires that must be attached to the shield for it to function with the FireStorm (this is
because the shield does not actually connect SPI to the GPIO pins, and the interrupt is not
connected.

Wiring diagram coming soon.

The ethernet shield should be connected to the FireStorm like a normal shield.

**Discontinuation note**: The [W5200 has been discontinued](http://www.seeedstudio.com/depot/W5200-Ethernet-Shield-p-1577.html), and Seeed recommends
that all users move to the [W5500](http://www.seeedstudio.com/depot/W5500-Ethernet-Shield-p-2433.html), but the code hasn't caught up yet.

### <a name="software"> Software Setup

To configure the kernel to support the ethernet shield, we need to edit several files in the Kernel
application as well as the `build.lua` file. All paths mentioned are relative to the base
`firestorm_environment` or `workspace` directory mentioned in other tutorials.

#### `build.lua`

In `build.lua`, change the `kernel_opts` table to enable the ethernet shield. Disabling quiet mode
is optional, but can be helpful for debugging the setup of the ethernet shield.

```lua
kernel_opts = {
    quiet = false, -- set to false for DEBUG information
    eth_shield = true -- sets a flag for the Kernel Makefile
}
```

Specifically, this is setting the `WITH_WIZ` flag and changing the linker file to give more memory
to the ethernet shield.

The ethernet shield does not contain the Lua runtime, so whatever libs and autorun you specify in
the `build.lua` file will be ignored.

#### `Makefile`

In `toolchains/stormport/apps/Kernel/Makefile`, replace the IPv6 address associated with
`IN6_PREFIX` to be your /48 prefix (see the [network configuration section](#netweork)). For example, if your IPv6 prefix was `2001:470:1234::/48`, then
you would choose a subnet (e.g. `2`) and place that in the Makefile

```
PFLAGS += -DIN6_PREFIX=\"2001:470:1234:2::\"
```

Further down the file, make sure that the `RPL_SINGLE_HOP_ROOT` flag is enabled, and the
`RPL_SINGLE_HOP` flag is disabled. The latter flag is for non-border router nodes.

```
#PFLAGS += -DRPL_SINGLE_HOP=\"fe80::212:6d02:0:3028\"
PFLAGS += -DRPL_SINGLE_HOP_ROOT
```

Nodes running a Kernel version >=4.0.3.0 will discover the IPv6 prefix automatically, but will NOT
discover the border router root node automatically. This must be configured on each node in the
Makefile, making sure to disable the `RPL_SINGLE_HOP_ROOT` flag.

```
PFLAGS += -DRPL_SINGLE_HOP=\"fe80::212:6d02:0:XXXX\"
#PFLAGS += -DRPL_SINGLE_HOP_ROOT
```

Where `XXXX` is the 4-byte id of the border router node.

#### `EthernetP`

In `toolchains/stormport/apps/Kernel/EthernetP.nc`, we need to statically assign an IPv4 address to
the border router for the IPv6 tunnel. Look for the following code inside `event void
IPControl.startDone` and edit the addresses to something that makes sense for your network. The
border router does not currently do DHCP to receive an IPv4 address.

* `destip`: the IPv4 address of the computer on your network that is acting as the IPv6 tunnel.
* `srcip`: the desired IPv4 address of the border router. This will be used in the IPv6 tunnel configuration.
* `netmask`, `gateway`: whatever makes sense for your network

```c
destip = makeIPV4(10, 4, 10, 142);
{
    uint32_t srcip   = makeIPV4(10,4,10,141);
    uint32_t netmask = makeIPV4(255,255,255,0);
    uint32_t gateway = makeIPV4(10,4,10,1);
    call EthernetShieldConfig.initialize(srcip, netmask, gateway, mac);
}
```

### <a name="network"> Network Configuration

This assumes you have a public IPv6 address and an allocated /48 subnet. [Hurricane
Electric](https://tunnelbroker.net/) is a great service for this.

The tunnelling server should be a computer that has access to the static IPv4 address of the
border router. The configuration uses the following information:

* `FSBR_IP4`: the IPv4 address of the border router (e.g. `10.4.10.2`)
* `EXTERN_IP6`: the external (public-facing) IPv6 address of the server (e.g. `2001:470:4b:375::/64`)
* `FSBR_SUBNET_TUN`: The /64 of the border router subnet (based off the /48) (e.g. `2001:470:4112:2::/64`
* `SERVER_SUBNET_TUN`: The /64 of the server (based off the /48) (e.g. `2001:470:4112:1::/64`)
* `SERVER_IP6`: the internal IPv6 address of the server (on `SERVER_SUBNET_TUN` subnet) (e.g. `2001:470:4112:1::5/64`)

Then run

```
ip tun del tunnel
ip tunnel add tunnel mode sit remote $FSBR_IP4
ip link set tunnel up
ip addr add $SERVER_IP6 dev tunnel
ip route add $EXTERN_IP6 dev tunnel
ip route add $SERVER_SUBNET_TUN dev tunnel
ip route add $FSBR_SUBNET_TUN dev tunnel
```

### <a name="troubleshooting"> Troubleshooting
