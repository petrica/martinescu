---
title: "Mac Butterfly Keyboard Double Key Press"
date: 2020-04-15T20:49:27+03:00
draft: true
---

## Introduction
If you have a mac with a butterfly keyboard, you most probably already know and have experience the most common issue that comes with it: doubling of the pressed keys.

Searching the web for this issues will yield all sort of results that will most likely point to a "dust" problem that accompanies these keyboards. While this might be a legitimate reason, this was not necessary the case for me. 

Having a look at the way the keyboard is built, it does come close to the keyboards found in the old mobile phones that suffered from the same constraints: slim profile devices with not enough space to fit in a proper keyboard. In order to create two contact leads that can be pressed against each other to form a switch, you could use the PCB board as one lead and a thin plastic or metal foil on top as the second lead. The same metal or plastic foil can act as a string too in order to push the button back after is has been pressed. This is perfectly fine for buttons that are meant to be pressed from time to time, but when we are talking about a keyboard that needs to be used daily, the mechanics need to be a little bit different, hence normal profile keyboards that last a lifetime.

## How to fix it

Apple butterfly keyboards suffer physical damage after months of use and this will show as doubling of the keys being pressed. This is nothing but an imperfect electrical contact due to a hardware damage.

* How to properly fix this? By replacing the damaged lead (i.e. the plastic foil). The downside is that this is expensive and you will have to keep doing this for every key that fails to work properly.
* How to temporary fix this? Using a software that prevents doubling of the keys.

## Using Unshaky

A software that addressed just that, preventing a key to register multiple sequential clicks by throttling the keys. In other words, disabling the keys for a fraction of a second after the initial press so that it does not register multiple times. This fraction of a second is configurable for each key individually and it is expressed in milliseconds.

My overall experience with this tool has been promising and has definitely improved the previously frustrating
experience of having to keep correcting myself each and every time a key would double. However, I still feel relieved
whenever I switch to an old or in fact, to any other keyboard. This keyboard is absolute garbage when it comes to enjoying the act of typing.

Coming back to Unshaky, I was able to make this work for me by following these steps:
1. Set a 70ms delay for all keys and start using your mac for a few days.
2. Open the statistics window where you can see a top of all the shaky keys that have been identified. Reset the delay to 0ms to all keys and set a 70ms delay only for the most notorious shaky keys. Some may just be false positives where you actually intended to have them double pressed (eg: left arrow).
3. For any new shaky key that did not make the top initially, set a 70ms delay. 
4. Rinse and repeat step 3.

![Shaky Stats](/images/unshaky/unshaky-stats.png)
![Shaky Stats](/images/unshaky/unshaky-config.png)

That's it. Enjoy your keyboard for a few more... months probably.








