---
layout: page
title: Blackfin+ GNU Toolchain and Debugger
description: for ADI Blackfin+ fixed point DSP
img: assets/img/IMG_3539.JPG
importance: 5
category: fun
---

ARM processors are everywhere these days, and it's for good reason. They're brilliantly powerful and cost-effective parts. But sometimes it can be fun and interesting to learn about some of the other types of processors that are out there. The Analog Devices Blackfin+ parts are fixed point DSP chips that sought to integrate more microcontroller-like features and peripherals to fill the space for applications that wanted both quick number crunching and modern microcontroller convenience in one package. These were very cool to me when I was looking to learn more about DSP on embedded systems and coming from Cortex M4 microcontrollers that were capable but still left you wanting more in terms of DSP performance.

However, the only supported toolchain for these parts cost thousands of dollars per seat and was far outside my budget as someone who was not using this for commercial products. I took this as an opportunity to REALLY go from the ground up, starting with adding support for the part to the GCC compiler, and then everything in between on the way to creating some audio effects and synthesizers.

Here are some videos of projects that I used my toolchain to make:

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        <iframe width="100%" height="480" src="https://www.youtube.com/embed/4m1YorzgxkY" title="FM Synth Bass Demo" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
    </div>
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        <iframe width="100%" height="480" src="https://www.youtube.com/embed/Mct14THKZX8" title="blackfin fm synthesizer demo" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
    </div>
</div>

## GCC Support

ADI previously supported a GCC toolchain for their Blackfin processors, but it was abandoned long before the Blackfin+ processors came about. Since the Blackfin+ instruction set is a superset of the original Blackfin instruction set, this seemed like a great place to start. GCC is massive and even being able to configure and build it from source is difficult. I invite those interested in what is involved in extending and modifying the different tools in GCC to have a look through the commits in the repo.

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
  {% include repository/repo.html repository="deanm1278/blackfin-plus-gnu" %}
</div>

## Debug Support

If we're going to develop for this processor we need to be able to do more than just write and compile code. We also need to be able to use a debug probe to step through code, read and write memory, and set breakpoints. This is a big ask on this processor because the Blackfin+ appears to have an entirely different debug port than previous blackfin processors. The reference manual says it integrates ARM Coresight IP, but everything about how debugging works under the hood is entirely undocumented. This means the debugging procedure had to be determined via reverse engineering.

To reverse engineer the debug procedure I hooked up a logic analyzer to the JTAG port of a development board that could be debugged from the vendor tools (Cross Core Embedded Studio). This allowed the JTAG signals that are sent and received by the probe when you do an action like reading memory or stepping over an instruction.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/bfin_read.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    read memory with the vendor debugger
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/bfin_decode.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    export the captured debugger trace
</div>

The exported data can then be imported into Python for analysis. At this point even the most simple action of reading one word of memory gives us a mess of over 500 JTAG events. Stepping over a line of code gives nearly 3000 events, and some operations give many more.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/bfin_export.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    export the captured debugger trace
</div>

Since the goal is to understand what the debug probe is doing enough to reimplement everything on our own, the data in it's current form is useless. To further clean the data we can re-implement the JTAG state machine in python and loop through the rows, grouping them into operations. This leaves us with 32 operations for our simplest case single word read.

In order to better understand what's going on we can also try to import and parse out register names and address from an XML memory map of the processor (as well as some other special registers that are not in the memory map but were found in documentation for older Blackfin processors). The grouped operations can be matched to the memory map to see what memory locations are being accessed.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/bfin_xml.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    memory locations parsed from the map
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/bfin_join.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    grouped operations joined against the memory map
</div>

We can then further simplify and group these operations into 'transactions'. Also, since I had just been implementing the opcodes in the compiler portion of this project, I noticed that a lot of the data being sent to the processor looked suspiciously similar. Disassembling these opcodes and parsing the transactions shows pretty clearly what's going on in our memory read.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/bfin_txn.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

Now we have something we can implement in our open source debugger! We are down from >500 entries of garbage to a few human readable operations. The debugger is executing opcodes directly on the processor to read some data and instruction cache status registers, and then passing the data back to itself via the EMUDAT register. Doing the same steps outlined above with the other necessary debugger operations allows us to make a complete debugger we can use for development.

