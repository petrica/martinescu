---
title: "Homio RFC Draft"
date: 2021-06-07T00:05:48+03:00
draft: true
---

![Homio Home Automation](/images/homio/homio.jpg)

## Brief Project Overview

The purpose of this post is to try and document a proposal for a home automation implementation for different sensors and components
using Arduino framework and hardware components that are easily available in the Arduino ecosystem. 

The main driver was to steer away from having to install a WiFi chip on each and every sensor or component. The reasons for this are as follows:
* reduce or limit polluting the list of connected devices on your WiFi router.
* drastically reduce power consumption on sensors using alternative wireless chips, protocols and MCUs.
* reduce the overall cost of the sensor or device by using alternative wireless chips and MCUs.

Software and hardware intended to be used:
* [Arduino](https://www.arduino.cc/) - probably the most popular electronics platform
* [Atmel AVR MCUs](https://en.wikipedia.org/wiki/AVR_microcontrollers) - cheap and widely used microcontrollers.
* [nRF24](https://lastminuteengineers.com/nrf24l01-arduino-wireless-communication/) - cheap and reliable wireless communication chip.
* [ESP8266/ESP32](https://www.espressif.com/en/products/socs/esp8266) - WiFi chip that is cheap and popular amongst DIYs engineers. 
* [ESPHome](https://esphome.io/) - framework intended for ESP8266/ESP32 WiFi chips, that uses simple configuration files to generate the firmware for your device.
* [NRFLite](https://github.com/dparson55/NRFLite) - probably the lites library for controlling nRF24 chips with only two pins.

## High Level Diagram

<div class="mxgraph" style="max-width:100%;border:1px solid transparent;" data-mxgraph="{&quot;highlight&quot;:&quot;#0000ff&quot;,&quot;nav&quot;:true,&quot;resize&quot;:true,&quot;toolbar&quot;:&quot;zoom layers lightbox&quot;,&quot;edit&quot;:&quot;https://app.diagrams.net/?client=1#Hpetrica%2Fmartinescu%2Fmaster%2Fstatic%2Fdiagrams%2Fhomio-highlevel-architecture.drawio&quot;,&quot;url&quot;:&quot;https://raw.githubusercontent.com/petrica/martinescu/master/static/diagrams/homio-highlevel-architecture.drawio&quot;}"></div>
<script type="text/javascript" src="https://viewer.diagrams.net/embed2.js?&fetch=https%3A%2F%2Fraw.githubusercontent.com%2Fpetrica%2Fmartinescu%2Fmaster%2Fstatic%2Fdiagrams%2Fhomio-highlevel-architecture.drawio"></script>

The diagram should be self explanatory. ESPHome Hub is the central piece of the architecture. It is the bridge between the home automation console and the myriad of IoT devices that can be actuators, sensors or generic components.
The architecture makes use of the already existing integration between ESPHome and Home Assistant, thus just building on top of this to connect custom IoT devices.

|Component|Description|
|---------|-----------|
| Home Assistant | Open-source software for home automation that is designed to be the central control system for smart home devices with focus on local control and privacy. |
| ESPHome Hub | ESP8266/ESP32 based hardware device that has a custom implementation of the Homio hub in order to connect and communicate with Homio sensors and components |
| Sensors | Custom built Homio sensors using Arduino and a protocol to communicate with ESPHome Hub |
| Components | Custom built Homio components using Arduino and a protocol to communicate with ESPHome Hub |

### ESPHome Hub Diagram

<div class="mxgraph" style="max-width:100%;border:1px solid transparent;" data-mxgraph="{&quot;highlight&quot;:&quot;#0000ff&quot;,&quot;nav&quot;:true,&quot;resize&quot;:true,&quot;toolbar&quot;:&quot;zoom layers lightbox&quot;,&quot;edit&quot;:&quot;https://app.diagrams.net/#Hpetrica%2Fmartinescu%2Fmaster%2Fstatic%2Fdiagrams%2Fhomio-hub.drawio&quot;,&quot;url&quot;:&quot;https://raw.githubusercontent.com/petrica/martinescu/master/static/diagrams/homio-hub.drawio&quot;}"></div>
<script type="text/javascript" src="https://viewer.diagrams.net/embed2.js?&fetch=https%3A%2F%2Fraw.githubusercontent.com%2Fpetrica%2Fmartinescu%2Fmaster%2Fstatic%2Fdiagrams%2Fhomio-hub.drawio"></script>

The hub as a central piece of the architecture will have two hardware modules that are used for wireless communication: nRF24 for low power wireless communication and the actual ESP WiFi module to connect the hub to Home Assistant.
Sensors connected to the hub need to also have a representation at the hub level in order for them to be logically represented at the console level.

### Homio Component Diagram

<div class="mxgraph" style="max-width:100%;border:1px solid transparent;" data-mxgraph="{&quot;highlight&quot;:&quot;#0000ff&quot;,&quot;nav&quot;:true,&quot;resize&quot;:true,&quot;toolbar&quot;:&quot;zoom layers lightbox&quot;,&quot;edit&quot;:&quot;https://app.diagrams.net/#Hpetrica%2Fmartinescu%2Fmaster%2Fstatic%2Fdiagrams%2Fhomio-component.drawio&quot;,&quot;url&quot;:&quot;https://raw.githubusercontent.com/petrica/martinescu/master/static/diagrams/homio-component.drawio&quot;}"></div>
<script type="text/javascript" src="https://viewer.diagrams.net/embed2.js?&fetch=https%3A%2F%2Fraw.githubusercontent.com%2Fpetrica%2Fmartinescu%2Fmaster%2Fstatic%2Fdiagrams%2Fhomio-component.drawio"></script>

The component or sensor is really simple, compound out of a communication module nRF24, the actual MCU for processing the sensor data and the actual sensor or component that needs to be controlled.

## Network Topology

A mesh of interconnected devices is usually the way to go for a professional network solution. The nRF24 chip already has a few libraries that offer the possibility to create a mesh of interconnected IoT devices, however this paper will present
an alternative network topology that will focus to reduce the overall network complexity. 

The objectives are:
* make use of a central hub or device to which all the other IoT devices will connect to and communicate with.
* have the possibility to support a large number of devices intermittently connected to the hub.
* support [NRFLite](https://github.com/dparson55/NRFLite) library that has the possibility to connect nRF24 chip to a MCU with only **two wires**.

The nRF24 chip comes with quite a few interesting features that we can leverage to connect a large number of independent devices:
* Automatically packet handling - assemble, validate and disassemble the messages between sender and receiver.
* Auto acknowledgement - whenever a message is sent from one chip to another, for a short period of time the chips automatically switch roles such that the initial sender will receive an ACK packet back confirming that the receiver got the message.
* Acknowledgement payload - the ACK packet can contain an optional payload that can be sent pack to the sender as a response. We will make use of these feature to create a custom communication protocol.
* Auto retransmission - builds on top of the auto acknowledgement packet. Whenever the sender does not receive the ACK packet back, the chip will retransmit the message. With the growing number of IoT devices communicating with the same hub, the probability of message clashing will increase.
To mitigate this the auto retransmission will be our ally.
* MultiCeiver - enables the chip to receive data on six different logical pipes, using the same frequency channel. In other words it opens the possibility to independently receive packets from six different devices at one time, preserving the above features for each pipe: auto packet handling, auto ACK, ACK payload and retransmission. 

### How Do the nRF24 Chips Communicate

For two chips to communicate, each chip must listen to an unique address that the other chip can use to send packets to. The ACK packet is handled a bit differently as described below.

<div class="mxgraph" style="max-width:100%;border:1px solid transparent;" data-mxgraph="{&quot;highlight&quot;:&quot;#0000ff&quot;,&quot;nav&quot;:true,&quot;resize&quot;:true,&quot;toolbar&quot;:&quot;zoom layers lightbox&quot;,&quot;edit&quot;:&quot;https://app.diagrams.net/#Hpetrica%2Fmartinescu%2Fmaster%2Fstatic%2Fdiagrams%2Fhomio-two-nrf-diagram.drawio&quot;,&quot;url&quot;:&quot;https://raw.githubusercontent.com/petrica/martinescu/master/static/diagrams/homio-two-nrf-diagram.drawio&quot;}"></div>
<script type="text/javascript" src="https://viewer.diagrams.net/embed2.js?&fetch=https%3A%2F%2Fraw.githubusercontent.com%2Fpetrica%2Fmartinescu%2Fmaster%2Fstatic%2Fdiagrams%2Fhomio-two-nrf-diagram.drawio"></script>

Considering this simple scenario, **nRF Sender** will be configured having the transmission address set to 1 (TX: 1) and receiving address set to 1 as well (RX: 1), as the ACK packet will be sent back from **nRF Receiver** using it's listening address.

For the MultiCeiver situation, the **nRF Hub** will listen to multiple addresses (RX: 1, RX: 2 ... Rx: n) and will be able to send back an ACK packet on each individual address or pipe.

<div class="mxgraph" style="max-width:100%;border:1px solid transparent;" data-mxgraph="{&quot;highlight&quot;:&quot;#0000ff&quot;,&quot;nav&quot;:true,&quot;resize&quot;:true,&quot;toolbar&quot;:&quot;zoom layers lightbox&quot;,&quot;edit&quot;:&quot;https://app.diagrams.net/#Hpetrica%2Fmartinescu%2Fmaster%2Fstatic%2Fdiagrams%2Fhomio-multi-nrf-diagram.drawio&quot;,&quot;url&quot;:&quot;https://raw.githubusercontent.com/petrica/martinescu/master/static/diagrams/homio-multi-nrf-diagram.drawio&quot;}"></div>
<script type="text/javascript" src="https://viewer.diagrams.net/embed2.js?&fetch=https%3A%2F%2Fraw.githubusercontent.com%2Fpetrica%2Fmartinescu%2Fmaster%2Fstatic%2Fdiagrams%2Fhomio-multi-nrf-diagram.drawio"></script>

When **nRF Hub** needs to initiate the communication, the situation described previously between two chips applies. The **nRF Hub** simply sets the TX address to one of the child nRF devices and
one of the pipes to listen to the same TX address for the ACK packet. Usually the pipe with the 0 index is used for receiving ACK packets from children, leaving out 5 more pipes for active listening from 5 other devices.

## Homio Communication Protocol

Considering our initial goal of having a large number of IoT devices communicating with the Hub and the fact that the nRF24 chip can only listen to 6 individual addresses at one time, we need to find an alternative communication protocol that
can somehow satisfy our hypothesis. Knowing that the IoT devices will most of the time stay silent and rarely go back to the Hub to dump their data or receive updates from the Hub, then we are in a position of sharing the Hub communication pipes between multiple
devices.

### Requesting a Communication Pipe

In order for a device to send data to the Homio Hub, it will need to ask the Hub for an available communication pipe address. To do this, we could use one main address of the Hub to listen to these specific requests.
This means that multiple devices will use the same address to send their request packet to the Hub and get back the ACK packet containing the available pipe address that they can use to privately talk to the Hub.
With this approach there are a few issues. Let's see what those issues are and how we can mitigate them:
* there is a good chance that the ACK packet will be received by all devices that sent the request for a communication pipe. This is because all devices are listening to the same address (main hub address) for the ACK packet.
When this happens the Hub actually processed only one of the requests in the background, such that the ACK will contain the ID of the device that got the permission to talk to the Hub. The Hub acts as a referee.
* there is a good change that packets might clash

<div class="mxgraph" style="max-width:100%;border:1px solid transparent;" data-mxgraph="{&quot;highlight&quot;:&quot;#0000ff&quot;,&quot;nav&quot;:true,&quot;resize&quot;:true,&quot;toolbar&quot;:&quot;zoom layers lightbox&quot;,&quot;edit&quot;:&quot;https://app.diagrams.net/#Hpetrica%2Fmartinescu%2Fmaster%2Fstatic%2Fdiagrams%2Fhomio-pipe-request-diagram&quot;,&quot;url&quot;:&quot;https://raw.githubusercontent.com/petrica/martinescu/master/static/diagrams/homio-pipe-request-diagram&quot;}"></div>
<script type="text/javascript" src="https://viewer.diagrams.net/embed2.js?&fetch=https%3A%2F%2Fraw.githubusercontent.com%2Fpetrica%2Fmartinescu%2Fmaster%2Fstatic%2Fdiagrams%2Fhomio-pipe-request-diagram"></script>


