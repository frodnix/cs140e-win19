---
layout: page
title: Sonar, interrupts.
show_on_index: true
---

### Overview

This is a fun lab.  You'll:

 1. Implement a sensor driver on your own --- a distance
 measuring device that uses sonar.    

 2. Hook the output up to an LED that gets brighter as distance gets
 shorter and struggle for a bit trying to make the variation clean.
 This is not as straightforward as it might seem, and not b/c of 
 device issues.

 3. Use the code from the last lecture to set up an interrupt handler
 to fix the problems in (2).

There's some simple skeleton code, but you're going to do all the work
yourself, including getting the data sheets and anything else you
need.  This process will foreshadow what you'll have to do when you
write drivers for devices on your own (with the simplification that
in this case we know its possible to get the needed information!).

##### Motivation.

Most OS classes will have a few minutes where the words "device driver"
or "I/O devices" appear, but that's as close as many get to dealing with actual
devices and actual data sheets.  This class goes the opposite direction.
Two reasons:

    1. 90% of an OS is device driver code, so ignoring it entirely to 
    spend a whole course on the other 10% of the OS doesn't make sense.  You 
    have to work with real devices to have informed views on how to package
    them in a larger system.

    2. After this course, the hope is that you can readily make
    interesting embedded systems (e.g., for art installations, consumer
    gizmos, gee-whiz demos for your parents when they write a tuition
    check).  More than anything else, doing so requires getting through
    device sheets.

You could argue that being able to aggressively extract the information
you need from opaque, confusing, wrong datasheets is the main skill that
separates real system hackers from Javascript programmers.  Once you
get good at it, you'll realize what a superpower it is.

### Implementing the HC-SR04 sonar driver.

First, find the data sheet.   Google search for 'HC-SR04 datasheet'
or some variation of that.  You're looking for the manufacturer's doc for
what to do to use the sensor.
When you find it, you'll notice it's fairly confusing.  This is the norm.

Don't necessarily stop with the first document you find.  
The first one to show up for me is a 3-page sheet that isn't super
helpful (common); there's a longer one if you keep digging.  Also, the
`sparkfun.com` comments for the device have useful factoids (but also
wrong ones, which --- you guessed it --- is common).

The game is to skim around ignoring the vast majority of information
(which will be irrelevant to you) and find the few bits of information you
need: 

  1. What the pins do (input power, output, ground).
  2. What voltage you need for power (`Vcc`).
  3. What reading you get when the the device is "open" (not signaling
	anything).
  4. what reading you get when the device is closed / signaling (`Vout`).
  5. Are there specific time delays you need to have in your code?  one
  thing about real devices: they really love ad hoc time delays, often
  around when you change their state. 

If any of the voltage values are out of range of the pi, you can't connect
directly.

Notice what the range of values are: if "low" is close to zero, but not
guaranteed to be zero, you'll have to set the input pin as a `pulldown
resistor` to clean up the signal.

Some hints, you'll need to:

   0. do the pins need to be set as `pulldown`?

   1. init the device (pay attention to time delays here)

   2. do a send (again, pay attention to any needed time delays)
        
   3. Measure how long it takes and compute round trip by converting
   that time to distance using the datasheet formula

   4. You can start with the code in `gpioextra.h` but then replace it
   with your own (using the broadcom pdf in the docs/ directory).

Troubleshooting:

  0. Check wiring.   Check if the sensor is reversed.  Use specific
  wire colors to keep track of what is going where.  It's easy to be
  off-by-one.

  1. readings can be noisy --- you may need to require multiple
  high (or low) readings before you decide to trust the signal.

  2. there are conflicting accounts of what value voltage you
  need for `Vcc`.

  3. Put the device so it's overhanging from your breadboard, otherwise
  you may get unfortunate reflections.

### Build your own version of `gpio_pullup`/`gpio_pulldown`.

### Build your own version of PWM.

We can make the LED dim by doing `PWM`, which mechanically means turning
the LED on/off quickly to reach a given percentage of brightness.  
Write the code to do this.  From past experience: run it on a few
values without the sonar so that you can see it works.

The interesting thing about computing exact PWM quickly, is that it is a
line drawing algorithm.