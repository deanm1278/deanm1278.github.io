---
layout: page
title: Blackfin+ GNU Toolchain and Debugger
description: for ADI Blackfin+ fixed point DSP
img: assets/img/IMG_3539.JPG
importance: 5
category: fun
---
#### 2018

ARM processors are everywhere these days, and it's for good reason. They're brilliantly powerful and cost-effective parts. But sometimes it can be fun and interesting to learn about some of the other types of processors that are out there. The Analog Devices Blackfin+ parts are fixed point DSP chips that sought to integrate more microcontroller-like features and peripherals to fill the space for applications that wanted both quick number crunching and modern microcontroller convenience in one package. These were very cool to me when I was looking to learn more about DSP on embedded systems and coming from Cortex M4 microcontrollers that were capable but still left you wanting more in terms of DSP performance.

However, the only supported toolchain for these parts cost thousands of dollars per seat and was far outside my budget as someone who was not using this for commercial products. I took this as an opportunity to REALLY go from the ground up, starting with adding support for the part to the GCC compiler, and then everything in between on the way to creating some audio effects and synthesizers.

In this writeup I'll do a general overview of the pieces of this project. Repositories are posted for those who want to look deeper into any one part.

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

ADI previously supported a GCC toolchain for their Blackfin processors, but it was abandoned long before the Blackfin+ processors came about. Since the Blackfin+ instruction set is a superset of the original Blackfin instruction set, this ADI GCC fork seemed like a great place to start. GCC is massive and even being able to configure and build it from source is difficult. I will leave the details of the compiler modification for another possible writeup in the future and instead will just point to my GCC fork below for those interested in looking through the commits to see what needs to be done for this type of thing:

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
  {% include repository/repo.html repository="deanm1278/blackfin-plus-gnu" %}
</div>

## Debug Support

If we're going to develop for this processor we need to be able to do more than just write and compile code. We also need to be able to use a debug probe to step through code, read and write memory, and set breakpoints. This is a big ask on this processor because the Blackfin+ appears to have an entirely different debug port than previous blackfin processors. The reference manual says it integrates ARM Coresight IP, but everything about how debugging works under the hood is entirely undocumented. This means the debugging procedure had to be determined via reverse engineering.

To reverse engineer the debug procedure I hooked up a logic analyzer to the JTAG port of a development board that could be debugged from the vendor tools (Cross Core Embedded Studio). This allows us to capture the JTAG signals that are sent and received by the probe when you do an action like reading memory or stepping over an instruction.

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

Since the goal is to understand what the debug probe is doing enough to reimplement everything on our own, the data in it's current form is useless. To clean the data up a bit we can re-implement the JTAG state machine in python and loop through the rows, grouping them into operations. This leaves us with 32 operations for our simplest case single word read.

In order to better understand what's going on we can also try to import and parse out register names and address from an XML memory map of the processor (as well as some other special registers that are not in the memory map but were found in documentation for older Blackfin processors). The grouped operations can be matched to the memory map to see what memory locations are being accessed.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/bfin_xml.png" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    memory mapped registers parsed from the map
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

Now we have something we can implement in our open source debugger! We are down from >500 entries of garbage to a few human readable operations. The debugger is executing opcodes directly on the processor to read some data and instruction cache control registers, and then passing the data back to itself via the EMUDAT register. Doing the same steps outlined above with the other necessary debugger operations allows us to make a complete debugger we can use for development.

The full example notebook for reverse engineering the JTAG traffic can be seen <a href="https://github.com/deanm1278/bfin-examples/blob/master/bfin-jtag-example.ipynb">here</a>.

The OpenOCD fork with added Blackfin+ support is located here:
<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
  {% include repository/repo.html repository="deanm1278/openocd" %}
</div>

## Header File Generation

We can now compile, load, and debug code, but we have no drivers of any kind to make use of the on-chip resources. In fact, our build system really has no concept of where anything is on the chip at this point. Since it's impractical to be manually cross-referencing the manual for raw addresses, we want header files that define the memory mapped registers. These can be generated from an XML file detailing the memory map of the chip. I chose to use a format similar to the ARM CMSIS header files, so the goal is to turn somthing that looks like this:

```
<register name="L1DM_DCTL"                       read-address="0x1FC00004" write-address="0x1FC00004" bit-size="32" type="IO" mask="FFFFFFFF" group="L1DM" description="L1DM Data Memory Control Register" />
   <register name="L1DM_DCTL.ENX"                parent="L1DM_DCTL" bit-position="16" bit-size="1" description="Enable Extended Data Access" />
   <register name="L1DM_DCTL.RDCHK"              parent="L1DM_DCTL" bit-position="9" bit-size="1" description="Read Parity Check" />
   <register name="L1DM_DCTL.CBYPASS"            parent="L1DM_DCTL" bit-position="8" bit-size="1" description="Cache Bypass" />
   <register name="L1DM_DCTL.DCBS"               parent="L1DM_DCTL" bit-position="4" bit-size="1" description="Data Cache Bank Select" />
   <register name="L1DM_DCTL.CFG"                parent="L1DM_DCTL" bit-position="2" bit-size="2" description="Configure as Cache or SRAM" />
   <register name="L1DM_DCTL.ENCPLB"             parent="L1DM_DCTL" bit-position="1" bit-size="1" description="Enable CPLB Operations" />
```

into something that can be understood by the GCC compiler:
```
#if !(defined(__ASSEMBLY__))
/* ----- L1DM_DCTL : (32) L1DM Data Memory Control Register ----- */
typedef union {
	struct {
		uint32_t :1;			    /*!< bit	0	Reserved	*/
		uint32_t ENCPLB:1;			/*!< bit	1	Enable CPLB Operations	*/
		uint32_t CFG:2;			    /*!< bit	2..3	Configure as Cache or SRAM	*/
		uint32_t DCBS:1;			/*!< bit	4	Data Cache Bank Select	*/
		uint32_t :3;			    /*!< bit	5..7	Reserved	*/
		uint32_t CBYPASS:1;			/*!< bit	8	Cache Bypass	*/
		uint32_t RDCHK:1;			/*!< bit	9	Read Parity Check	*/
		uint32_t :6;			    /*!< bit	10..15	Reserved	*/
		uint32_t ENX:1;			    /*!< bit	16	Enable Extended Data Access	*/
		uint32_t :15;			    /*!< bit	17..31	Reserved	*/
	} bit;		                    /*!< Structure	used for bit access		*/
	uint32_t reg;		            /*!< Type	used for bit access		*/
} L1DM_DCTL_Type;
#define REG_L1DM_DCTL		(*(RwReg *)0x1FC00004UL)

#else
#define REG_L1DM_DCTL		(0x1FC00004) /**< \brief (L1DM) L1DM Data Memory Control Register */
#endif
```
This is very straightforward and the python code and resulting header file can be found here:
<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
  {% include repository/repo.html repository="deanm1278/bfin-CMSIS" %}
</div>

## Hardware

I designed some hardware in various form factors to play around with for this project. In general the design had the Blackfin+ chip and an Atmel SAMD21 microcontroller to act as a USB bridge and to facilitate the process of booting and flashing the board (Blackfin+ chips do not have onboard flash).

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/IMG_E3540.JPG" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/IMG_E3541.JPG" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/IMG_E3542.JPG" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Left: Arduino-style dev board with stereo audio codec. Middle: small form factor board. Right: guitar pedal board with potentiometers, relays, codec, etc.
</div>

## Board support and other libraries

I made various other tools and libraries to use for development and for some rudimentary DSP work:

<div class="repositories d-flex flex-wrap flex-md-row flex-column justify-content-between align-items-center">
  {% include repository/repo.html repository="deanm1278/audioFX" %}
  {% include repository/repo.html repository="deanm1278/ArduinoCore-blackfin" %}
  {% include repository/repo.html repository="deanm1278/bfin-scheduler" %}
  {% include repository/repo.html repository="deanm1278/bfin-fm" %}
</div>

## Conclusion

I hope you enjoyed this brief overview of these projects. It was a great learning experience to touch on virtually every level of the system from the compiler through to the linker, image generation, gdb and disassembly, debugging and OpenOCD, programming and boot process, peripheral drivers, scheduling and context switching, physical hardware, etc... all the way to the DSP and creating some fun and compelling end musical applications. 