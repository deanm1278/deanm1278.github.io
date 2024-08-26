---
layout: page
title: Synth Guitar
description: a guitar-shaped synthesizer
img: assets/img/IMG_2805.jpg
importance: 4
category: fun
---
#### Ongoing

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/IMG_2805.jpg" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    the current prototype
</div>

This is a currently ongoing project with the goal of making the guitar-shaped synthesizer I've always wanted. There have been many guitar synths and MIDI controllers over the years but I think there's always room for one more.

In my opinion there is a lot about the concept of "a guitar that sounds like a piano" or "a guitar that triggers whatever synthesizer you want" that doesn't quite work. The goal for this one is to make an instrument that's entirely it's own thing and matches the guitar's ability to respond to very fine changes in the players technique.

The general system design is shown below. A microcontroller receives control input from fretboard sensors, load cell tension sensors, and optical string trigger/displacement sensors. The microcontroller board is connected via a USB cable to an embedded Linux host (this also supplies power).

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/IMG_4727.jpg" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

## Tension Sensor

The naturalness and responsiveness will be achieved by adding a tension sensor to each string in the form of a load cell. These type of strain gauge load cells are widely available and are very common in commercial weigh scales. The load cells used are rated at 750g so only a very small component of the string tension can be transferred to the load cell.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/4630-03.jpg" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    A small strain gauge load cell (photo courtesy of Adafruit)
</div>

The tension head and other hardware in the tension sensor assembly are 3D printed out of stainless steel. Plastic parts flex too much under the stress of the strings and create cross coupling between different sensors.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/tension_asm3.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/IMG_2800.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Left: rendering of tension assembly. Right: current prototype.
</div>

The amplifier for the load cell uses an AD8426 instrumentation amplifier into a MAX7427 5th order lowpass filter, and then an amplifier with adjustable offset to place the signal in the desired range for the microcontroller ADC.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/loadamp.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

The microcontroller samples at 5.2kHz and then downsamples by 2 before sending to the host. The high sample rate for the tension sensor will allow us to detect fast changes in tension and infer pull-off technique.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/mainboard.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The mainboard for the guitar.
</div>

## Fretboard Assembly

We also need to know which fret the player is touching on the guitar. The neck has 21 frets, with each fret divided into 6 slices for a total of 126 sensors. Because a fretboard like this would be prohibitively expensive to prototype for a project like this I have adapted one from an old MIDI guitar.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/IMG_4270.jpg" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The MIDI guitar the neck was adapted from.
</div>

It would have been nice to be able to use the fretboard as-is, so I reverse engineered the protocol it used. However it ended up being too slow and 'careful'. It seems to have been designed to give a stable, debounced fret number reading (as you would need for a MIDI guitar). For this instrument we really just want all the data as fast as possible so I instead opted to design replacement circuitry to fit the same mechanical footprint but that would behave as desired for this application.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/IMG_4380.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/IMG_4410.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Left: original neck circuitry. Right: replacement neck circuitry.
</div>

I decided to be cost consicious on this so 4 extremely inexpensive microcontrollers are used that all run identical firmware and pipe their readings down to the next one in line, and then finally to the more powerful main microcontroller on the guitar main board.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/neck1.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/neck2.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/neck3.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    The replacement neck circuit boards.
</div>

Note that in designs like this it's typical to multiplex all the different switches because it allows you to use far less GPIO pins. However that would require each string to be electrically isolated, creating a major headache for the mechanicals. Instead each fret division is connected to it's own microcontroller pin.

## Optical Trigger

We also need to sense when the player hits or touches the strings with their picking hand. For this we will use photointerruptors mounted near the bridge. The string sits in the slot between the LED and the phototransistor. This allows us to sense not only vibration (like a magnetic pickup can) but also absolute displacement from muting the string.

The signal from the phototransistors undergoes some lowpass filtering and scaling to place it in the range of the microcontroller ADC. It is sampled at a rate of 5.2kHz and downsampled by 2 before sending to the host.

## Linux Host

The guitar enumerates as a USB CDC device to an embedded Linux host (currently I'm using a Nvidia Jetson nano) and sends raw sensor data. The use of an embedded Linux device makes it easy to collect telemetry data from the controller and develop the needed signal processing routines for converting the sensor data into sound, and also provides more than enough horsepower for whatever audio generation we choose. Here's a simple real time dashboard for printing the sensor data:

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/gtt_monitor.gif" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

## Next: Processing the sensor data