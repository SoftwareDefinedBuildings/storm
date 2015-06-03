---
layout: page
title: "Concurrency and Timers"
category: tut
date: 2015-06-02 02:20:46
---

### Table of Contents

* [Introduction to Timers](#introtimers)
* [Introduction to `cord`](#introcord)

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

### <a name='introcord'></a> Introduction to `cord`

Rich embedded applications, especially embedded network applications, need clean, systematic means of handling
concurrency. In addition to periodic scheduling, waveform generation and event handling, we often need to additional
capabilities:

* async operations: initiate concurrent activities that indicate completion through a callback rather than return
* sequencying over async operations

We have built a library `cord` over `storm.os` that provides a paradigm similar to event-driven JavaScript frameworks (like node)
to provide such an asynchronous execution framework using Lua's non-preemptive thread capabilities -- coroutines. (Similar to Java's
green threads). We refer to our FireStorm "green threads" as "cords".

In order for any cords to run, there needs to be an event loop running that can schedule the executing coroutines. If you
are running the stormsh shell, the event loop is already running. Any application that wants to use cords needs to run
the scheduler, which is entered with the call `cord.enter_loop()`. This is very often the last line in an application
because it does not return.

Here is a simple example that you can run from your Storm shell illustrating two main capabilities:

* create a new cord with `cord.new`, passing it a function that will execute concurrently. That function can utilize other
  `cord` library methods, and assumes that the `cord.enter_loop()` scheduler is running.
* `cord.yield()` allows other cords to execute. This is necessary because of the non-preemptive execution model of cords. This
  is not used often because cords will be naturally yielding to one another by awaiting for other async opreations to complete

```lua
function threads(numThreads, loopIterations)
    local i
    for i = 1,numThreads do
        cord.new(function()
            local j
            for j=1,loopIterations do
                print("Thread",i,"iteration",j)
            end
        end)
    end
end
```

Many of the scheduling inter-relationships can be captured with `cord.await`, borrowed in concept from node.js and C#'s `await`
keywords. Here is an example using `cord.await` in conjunction with `storm.os.invokeLater` to implement sleep.

```lua
function sleepThreads(numThreads, loopIterations)
    local i
    for i = 1,numThreads do
        cord.new(function()
            local j
            for j=1,loopIterations do
                print("Thread",i,"iteration",j)
                cord.await(storm.os.invokeLater, 1*storm.os.SECOND)
            end
        end)
    end
end
```

The general form of `cord.await` is `cord.await(afun, arg1, ... argN-1)`, where `afun` is an asynchronous function
with `n` arguments, the last of which is the completion call-back. `afun` will be called with the `n-1` provided
arguments and a callback that is generated by `cord`.

Now you have all the tools you need to build interesting corded apps. It can be stand-alone -- with the autorun ending
with `cord.enter_loop()` -- or by embedding the stormsh excution.
