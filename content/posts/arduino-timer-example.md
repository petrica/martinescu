---
title: "Arduino Timer Example"
date: 2021-02-16T23:48:56+02:00
draft: true
katex: true
---

## Introduction

Another valuable peripheral of a microcontroller that you can use to do more stuff with it is a timer. Although it might sound like a simple counter, it is in fact a lot more complex than that.
With the help of a few registers and some functionality behind it to manage those registries, you can use a timer to count external events, measure time intervals, generate waveforms and more.

## ATmega328P

Arduino Uno with ATmega328P microcontroller has actually more than one timer that you can use for various purposes:
* __Timer 0__ - it is an 8 bit timer, used by Arduino library.
* __Timer 1__ - it is a 16 bit timer, used by a few other Arduino libraries. It has a few additional features, such as the input capture unit that allows measuring frequency, duty-cycle, applying timestamp to events and more.
* __Timer 2__ - it is an 8 bit timer that has the added benefit of allowing an external 32KHz watch crystal to be used independent of the I/O clock. 

## Objective

In this article we will concentrate on using the timer to generate discrete intervals of time in which specific actions can be executed.

## Modes of operation

{{< figure src="/images/arduino-timer/atmega-timer-1-diagram.jpeg" title="ATmega Timer 1 Diagram" class="small" >}}

The timer's main component is a counter registry (TCNT1) that is incremented or decremented sequentially at a specific rate, dictated by the Counter Control Logic circuit. Together with the output compare registers A (OCR1A) and B (OCR1B) you can alter the behaviour of the timer such that the following modes of operation are possible:
* __Normal Mode__ - the counting direction is always up. The counter simply overruns when it passes its maximum 16-bit value. The OCR1A and OCR1B can be used to generate interrupt requests. This mode may be simple but does not have the flexibility to fully control the counting interval.
* __Clear Timer on Compare Match (CTC) Mode__ - The output compare register A (OCR1A) can be used to manipulate the counter resolution. The counter is cleared to zero when the counter value (TCNT1) matches the OCR1A value. There is an additional option to use the input capture register (ICR1) as the reset value, however this will not be discussed in this article.
* __PWM Modes__ - the pulse width modulation modes provide waveform generation output. This will not be covered in this article.

## Clear Timer on Compare Match Mode

This mode of operation is ideal to generate discrete time intervals. The following table details the resolution and the maximum intervals that you can generate on a \\(16MHz\\) CPU clock, choosing one of the prescaler values.

$$ T = {N(1+x) \\over f\_{ck}}$$

| CPU Clock <br/> \\( f\_{ck}\\) | Prescaler <br/>\\(N\\) | Resolution <br/>\\( x = 0 \\)   | Max Interval <br/>\\( x = 65536 \\) |
|-----------|-----------|--------------|--------------|
| \\(16MHz\\)     | \\(1\\)    | \\(62.5ns\\)       | \\(\\sim 4ms\\)         |
| \\(16MHz\\)     | \\(8\\)   | \\(500ns\\)        | \\(\\sim 32ms\\)        |
| \\(16MHz\\)    | \\(64\\)  | \\(4\\mu s\\)          | \\(\\sim 262ms\\)       |   
| \\(16MHz\\)    | \\(256\\) | \\(16\\mu s\\)         | \\(\\sim 1s\\)          |
| \\(16MHz\\)    | \\(1024\\)| \\(64\\mu s\\)         | \\(\\sim 4s\\)          |

## Interval Calculator 
<table class="noborders left" id="calculator">
    <tr>
        <th>CPU Clock:</th>
        <td><input id="cpu" type="text" value="16"/> \(MHz\)</td>
    </tr>
    <tr>
        <th>Interval:</th>
        <td><input id="interval" type="text" value="100" /> \(ms\)</td>
    </tr>
    <tr>
        <th></th>
        <td><button id="calculate">Calculate</button></td>
    </tr>
    <tr class="template">
        <th></th>
        <td>
            \(x=\) <span class="x"></span><br/>
            \(N=\) <span class="n"></span><br/>
            \(T=\) <span class="t"></span>\(ms\)<br/>
            \(f=\) <span class="f"></span>\(Hz\)
        </td>
    </tr>
</table>
<script src="https://code.jquery.com/jquery-3.6.0.slim.min.js" integrity="sha256-u7e5khyithlIdTpu22PHhENmPcRdFiHRjhAuHcs05RI=" crossorigin="anonymous"></script>
<script type="text/javascript">
    const resultHTML = $('.template', '#calculator');
    resultHTML.hide();
    function calculateX(t, cpu, n) {
        return t * cpu * 1000 / n - 1;
    }
    function calculateT(x, cpu, n) {
        return n * (1+x) / (cpu * 1000);
    }
    $('#calculate').click(function () {
        $('.result', '#calculator').remove();
        const cpu = $('#cpu', '#calculator').val();
        const t = $('#interval', '#calculator').val();
        var outputHTML = "";
        $([1, 8, 64, 256, 1024]).each(function (i, n) {
            const x = Math.round(calculateX(t, cpu, n));
            if (x >= 0 && x <= 65536) {
                const newT = calculateT(x, cpu, n);
                $('.n', resultHTML).html(n);
                $('.x', resultHTML).html(x);
                $('.t', resultHTML).html(newT);
                $('.f', resultHTML).html(1/newT * 0.5 * 1000);
                outputHTML += '<tr class="result">' + resultHTML.html()+ "</tr>";
            }
        });
        console.log(outputHTML);
        $('#calculator tr:last').after(outputHTML);
    });
</script>

<br/>

``` arduino
#define PIN 6

// Comment

setup() {
    uint8_t a = 10;
    pinMode(PIN, INPUT);
}

```





