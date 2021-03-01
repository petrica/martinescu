---
title: "Arduino Timer Example"
date: 2021-02-16T23:48:56+02:00
draft: true
---

## Introduction

Another part of a microcontroller that you can use to do more stuff with it is a timer. Although it might sound like a simple counter, it is in fact a lot more complex than that.
With the help of a few registers and some functionality behind it to fill in those registries, you can use a timer to count external events, measure time intervals, generate waveforms and more.

## ATmega328P

Arduino Uno with ATmega328P microcontroller has actually more than one timer that you can use for various purposes:
* __Timer 0__ - it is an 8 bit timer, used by Arduino library.
* __Timer 1__ - it is a 16 bit timer, used by a few other Arduino libraries. It has a few additional features, such as the input capture unit that allows measuring frequency, duty-cycle, applying timestamp to events and more.
* __Timer 2__ - it is an 8 bit timer that has the added benefit of allowing an external 32KHz watch crystal to be used independent of the I/O clock. 

## Objective

In this article we will concentrate on using the counter to generate discrete intervals of time in which specific actions can be executed.

## Modes of operation

![ATmega Timer 1 Diagram](/images/arduino-timer/atmega-timer-1-diagram.jpeg#right)

The timer main component is a counter (TCNT1) that is incremented or decremented sequentially at a specific rate, dictated by the Counter Control Logic circuit. Together with the output compare registers A (OCR1A) and B (OCR1B) you can alter the behaviour of the timer such that the following mode of operation are possible:
* __Normal Mode__ - the counting direction is always up. The counter simply overruns when it passes its maximum 16-bit value. The OCR1A and OCR1B can be used to generate interrupt requests. This mode may be simple but does not have the flexibility to fully control the counting interval.
* __Clear Timer on Compare Match (CTC) Mode__ - The output compare register A (OCR1A) can be used to manipulate the counter resolution. The counter is cleared to zero when the counter value (TCNT1) matches the OCR1A value. There is an additional option to use the input capture register (ICR1) as the reset value, however this will not be discussed in this article.
* __PWM Modes__ - the pulse width modulation modes provide waveform generation output. This will not be covered in this article.

## Clear Timer on Compare Match Mode

This mode of operation is ideal to generate discrete time intervals. The following table details the minimum and the maximum intervals that you can generate on a 16Mhz CPU clock, choosing one of the prescaler values.

| CPU Clock | Prescaler | Min Interval | Max Interval |
|-----------|-----------|--------------|--------------|
| 16Mhz     | clk / 1   | 62.5ns       | ~4ms         |
| 16Mhz     | clk / 8   | 500ns        | ~32ms        |
| 16Mhz     | clk / 64  | 4us          | ~262ms       |   
| 16Mhz     | clk / 256 | 16us         | ~1s          |
| 16Mhz     | clk / 1024| 64us         | ~4s          |







