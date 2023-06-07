---
layout: page
title: Music Reactive FPGA LED strip controller
description: for controlling lots of RGB LEDs
img: assets/img/led-wall.gif
importance: 6
category: fun
---
#### 2022

The most cost effective addressable LEDs only require one data line from the controller to the chain of LED, but utilize a sensitive timing-based protocol. The protocol requires an 800khz PWM-like signal with a different duty cycle for writing either a 1 or a 0 to the LED. This places a hard limit on the frame rate relative to the number of LEDs you are using in the chain. This limit for 24 data bits per LED is around:

$$ frames\ per\ second = {800000 \over (24 * num\ leds)} $$

This means that if you wanted to control a chain of 3000 LEDs you would only be able to achieve a frame rate of around 11 frames per second. This is far too low for creating music reactive animations, so the solution is to use multiple parallel channels. However, commonly available microcontrollers do not have the peripherals to accomplish this in a clean way, especially if you would like to use DMA to leave your processor open for number crunching tasks while LED data is shifted out.

An FPGA is a nice solution to this problem because you can create peripherals to handle this simple timing-based protocol. An FPGA integrated with a hard processor is an even nicer solution because you can utilize the processor to do your DSP and then push the framebuffers out to your FPGA peripherals via AXI DMA. I had this exact thing laying around in the form of a Zybo dev board with a Xilinx Zynq SoC.

It's way overpowered, but that makes it more fun.

Here are some videos of the LED controller in action:

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        <iframe width="100%" height="480" src="https://www.youtube.com/embed/6V8bBpglwkE" title="Zynq LED controller demo" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
    </div>
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        <iframe width="100%" height="480" src="https://www.youtube.com/embed/ijQ-leadnDU" title="Strymon Cloudburst Ambient Reverb" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
    </div>
</div>
<div class="caption">
    it is used at various places as the backdrop of this Strymon Cloudburst marketing video starting at around the 40 second mark
</div>

These applications are only running 3000 LEDs at around probably 90 frames per second across 10 channels, but the design would support many times this number of LEDs and channels.

## Design

The block diagram in vivado is (open the image in a new tab to see it larger):

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/led-bd.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

Essentially we have a custom AXI Stream verilog module called `ledstream_0` in the design that takes a stream of data provided by `axi_dma_1`. The design also includes another axi DMA IP to get data from the I2S IP that communicates with the codec on the dev board.

The ARM processor sets up the DMAs to read circularly linked descriptors from BRAM in the FPGA fabric. When new audio data is available an interrupt is fired, and the data is processed. The filled framebuffers are setup to DMA out to the LED controller peripheral that writes all channels in parallel.

The DSP for the example in the video is relatively simple: A Mel scale filterbank is applied to an FFT of the incoming audio and this data is used to make a visualizer.

Check out the related repositories for further info:

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
  {% include repository/repo.html repository="deanm1278/ledstream" %}
  {% include repository/repo.html repository="deanm1278/zybo_dma_hw" %}
  {% include repository/repo.html repository="deanm1278/zynq_led_controller_hw" %}
  {% include repository/repo.html repository="deanm1278/zynq_led_controller_app" %}
</div>