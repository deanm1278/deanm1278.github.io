---
layout: page
title: Tiny Analog/Digital Synthesizer
description: with an FPGA?
img: assets/img/synth.jpg
importance: 7
category: fun
---
#### 2017

A number of years ago I wanted to make a tiny synthesizer about the size of a credit card. I wanted it to have an analog filter and voltage controller amplifier (VCF), but use digital wavetable oscillators. At the time the Icestorm open source FPGA toolchain was getting a lot of attention for bringing FPGAs closer to the "maker" market. I thought FPGAs were really cool, so I decided to try to use one in my synthesizer. The features of the synth were:

- 3x 16 bit digital Wavetable Oscillators
- Fine and coarse tuning controls for each oscillator
- user loadable waveforms
- 1 square wave sub oscillator for each oscillator
- Full analog 4 pole resonant low pass filter
- ADSR envelope generator mappable to filter cutoff and VCA
- Monophonic and 3 voice Paraphonic modes
- Glide rate control
- 2x LFO (also any user loadable waveform)
- Mappable to many parameters including rate or depth of the other LFO
- Headphone output
- External input to filter
- All parameters controllable via Standard MIDI
- Header connections for modification
- Loading and saving presets
- Low cost and small form factor

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        <iframe width="100%" height="480" src="https://www.youtube.com/embed/Ub5NRgZzTfE" title="bottomfeeder synthesizer" frameborder="0" allow="accelerometer; autoplay;         clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
    </div>
</div>

## Design

Looking back on it all now the design was .... bad, but it still made sound and I guess you learn a lot from making bad designs. The synthesizer itself was very small and was attached to a larger faceplate with the controls.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/synth.jpg" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/synth-pcb.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    the PCB layout
</div>

The general architecture was an ARM Cortex-M0+ microcontroller would do the housekeeping tasks such as reading controls, receiving MIDI, LFO generation, CV generation, etc. and then would communicate with a Lattice ICE40 FPGA via SPI. This FPGA was handling the wavetable synthesis (likely with lots of aliasing) and outputting the 3 oscillators via some crude 16bit R2R DACs. The now analog audio would go through a voltage controlled filter (VCF) and then a voltage controlled amplifier (VCA).

The VCF was my own design and operated on a similar principle to VCFs that used LM13700 operational transconductance amplifiers. However, at the time I believed that I either didn't have enough PCB space or maybe that the needed DC-DC conversion for the LM13700 supply voltages would be too noisy. I designed this VCF and VCA that could be run from a +3.3V single supply.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/vcf.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

The FPGA is generally a very poor fit for this application as a simple microcontroller would have done a better job with far less effort, but part of the point was to learn some FPGA. A HDL called Chisel which was very new at the time was used to create the FPGA configuration.

All-in-all this was an invaluable learning experience for me at the time. For a closer look at any of this stuff see the repo:

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
  {% include repository/repo.html repository="deanm1278/bottomfeeder" %}
  {% include repository/repo.html repository="deanm1278/Chisel-wavetable" %}
</div>