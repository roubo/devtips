# OpenWrt NOTICES

## What's the "input device grab"

    By default, X input events are delivered to all clients that registered for a specific event on a window.
    The basic premise of a grab is that it affects event delivery to deliver events exclusively to one client only.
    There are three types of grabs, two classes of grabs, and two modes for a grab. The three types are:
    * active grabs
    * passive grabs
    * implicit passive grab
    The two classes are core grabs and device grabs and the two modes are synchronous and asynchronous.
    A grab comes in a combination of type + class + mode, so a grab may be an "active synchronous device grab".

## Something about the "Blcontrol problem"

    Since the input device (here is a touch screen) can not be grab by two clients with the block mode.
    And it is hard to catch the events by realtime in user space, so I processes the events in kernel space.
    Some ioctl are usefull.

## How to make code simple

    When I feel that it is hard to manage a project, and the project may be not good in the beginning.
    Simple things are not necessarily the best, but the best one must be simple.
    Every function must be used by many others.
    Every file must be used by many others.
    If not, it is failure.

