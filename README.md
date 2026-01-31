# Timed events inside a RISC OS application

## Introduction

Within most applications - not just ones under RISC OS - it is necessary to have timed
events, or events which happen when the system is not involved in other operations.
These events are often called callbacks, timers, triggers, or some other similar names.
They're often necessary because the controlling process wants to do something at a later
time, or when they're not involved in something more complex.

When used within a polled environment, it allows the system to check their state, or
trigger timed operations when it is safe to do so. In a threaded environment you might
just have a thread sleep and wake up when there was something to process, using a mutex,
semaphore or other similar notification. But in RISC OS, we don't have threads, so we
cannot do this - the polled behaviour is necessary.

It doesn't matter what the language is, the usual behaviour for the application is to
set up a callback of some form which happens at a later event, and the caller can handle
the callback in whatever manner they wish to.

## Timeouts library

To demonstrate the use of timed events, and callbacks, I'm going to use a very simple
library that I wrote some years ago called 'timeouts'. This has been used in a number of
applications, and modules, to ensure that events are triggered when the system is safe
to handle them.

The library has a simple interface, which is very similar to what you would find in most
libraries providing callbacks. The original code for this library was written for Nettle,
and subsequently modified to be used within the Mozilla browser.

* `TO_SetTimeout` - sets a timeout to happen in a number of centiseconds. The original version of this library used milliseconds, but that's not very friendly.

* `TO_SetEvery` - sets a timeout to happen in a number of centiseconds, and in that number of centiseconds repeatedly.

* `TO_ClearTimeout` - removes a previously set timeout.

* `TO_TriggerTimeouts` - triggers all the timeout calls which are outstanding at this point.

* `TO_NextTimeout` - returns the time of the next event, or 0xFFFFFFFF if there are no timeouts remaining to be triggered.

* `TO_ShowTimeouts` - show the list of timeouts that are registered.


## Example for a simple application

A command line application can easily handle timeouts whilst it is handling other
processing, if it has a central loop, or time at which it should perform operations.
In this example we are going to track the mouse and plot a circle on every call
through the loop.

The timeouts will be used to change the size of the circle on one timed event,
and to change the colour on another timed event.

