---
layout: page
title: "Concurrency and Timers"
category: tut
date: 2015-06-02 02:20:46
---

### Table of Contents

* [Introduction to Timers](#introtimers)
* [Introduction to `Cord`](#introcord)

### <a name="introtimers"></a> Introduction to Timers

Many aspects of embedded systems have to do with time -- sensors are sampled at some rate (or o nsome period), actions are
taken, activities are scheduled. The [`storm.os`](https://github.com/SoftwareDefinedBuildings/storm_elua/wiki#stormos) 
module provides some basic timer mechanisms for your use.

You will also find that in embedded systems it is very important to enclose and manage state, which helps with
tasks such as building state machines and constructing concurrent activities. Closures are a great way
to do that in a language like Lua. Here's an example of using closures to encapsulate state:

```lua
-- storm.os.now grabs the current time at full or reduced precision (storm.os.SHIFT_0, etc)
function deltaMaker()
    local start = storm.os.now(storm.os.SHIFT_0) -- capture start time
    return function()
        return storm.os.now(storm.os.SHIFT_0)
    end
end

d = deltaMaker()
print(d())
```

#### Scheduling Activities

Embedded systems make observations and take actions over time, either according to various schedules or in response
to various events. The most common is to take some action periodically. Here's a simple example:

```lua
t = storm.os.invokePeriodically(4*storm.os.SECOND, function()
    print("it is now", storm.os.now(storm.os.SHIFT_0))
end)
```

Notice that instead of a loop that busy-waits or sleeps for a period of time, we receive a handler for a time event.
When it is not active, it frees up the MCU for other activities, or drops to a low power state if there are no pending
tasks. The "fiber" that we have just created is effectively running in the background, and we can write other functions
and run other timers and code while it runs. We can cancel the timer by cancelling the handler.

```lua
storm.os.cancel(t)
```

Now, try writing a function that will cause the LED to blink on and off every second.

```lua
function blink(pin)
    storm.io.set_mode(storm.io.OUTPUT, pin)
    return storm.os.invokePeriodically(storm.os.SECOND, function()
        storm.io.set(storm.io.TOGGLE, pin)
    end)
end
```

### <a name='introcord'></a> Introduction to `Cord`
