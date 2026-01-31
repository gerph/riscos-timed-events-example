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

When run at the command line, CLITimers will display circles in changing sizes
and colours, controlled by the timers. Click the mouse to exit.


## Example for a wimp application

Wimp applications are slightly different, because they have to interact with other
applications. Usually you use a Wimp_PollIdle to get called back when there is
something for the application to do. Adding general timeouts to this process is
not much more than the command line application.

The `Wimp_Poll` is changed to `Wimp_PollIdle` depending on whether there are timers
present. When the null poll is received, the timers are triggered, and handle the
operations. The receipt of the Quit message causes the application to disable the
first timer, and add another timer that will eventually quit the application.

When run, the WimpTimers tool will play two alternating notes. This will continue
until the application is quit from the TaskManager. When quit, a descending tone is
played, on another timer, and finally the application exits.


## Other uses of timed events

Whilst the example here uses very simple timers to change behaviour on timers, this
could just as easily be used to monitor internet or serial connections, or to track
physical hardware. Animations by moving icons or the window could be handled in the
same way. Any number of handlers can be registered with this mechanism with no
additional load on the system except when the timed events are dispatched. In many
cases it would not be necessary to handle more than a couple of timed events, but
depending on the use there may be many many more.

In the Wimp example, we could have had the tick and the tock sounds on separate timers, but the likelihood is that these timers would become synchronised (eg if you pressed
F12), because any delayed timer would end up being triggered as overdue at the same
time.


## Dependencies

Used in this way, there is no other dependence on any other components within the
system to manage the timed events. The application retains all the information it
needs to function and does not need to rely on anything other than the standard
`Wimp_PollIdle` functionality.
