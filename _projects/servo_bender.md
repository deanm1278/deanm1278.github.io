---
layout: page
title: Electronic String Bender Guitar
description: wireless string bender powered by 4 brushless servos
img: assets/img/bender_cover.jpg
importance: 3
category: fun
---

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/bender_cover.jpg" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

Electric guitar players have been using 'pull-string' or 'B-bender' devices to mimick the sound of a pedal steel guitar for decades. These devices are usually manually actuated using the palm of your hand or by pulling on the guitar strap. This can be difficult to do while playing, and due to the design of these devices they are mostly only used on one of the six guitar strings. The pedal steel guitars the players are trying to imitate have multiple pedals to change the tuning of multiple strings individually, and the pedal steel player can use their feet to work the pedals. I wanted to create a string bender guitar that could use pedals on the floor to electronically change the tuning on the fly.

The criteria for this design were:
 - be compact enough to fit onto the body of an electric guitar without feeling bulky
 - be wirelessly operated using foot pedals
 - be able to individually change tuning of 4 separate strings
 - be able to change the tuning quickly and quietly so as not to interrupt the music
 - be battery powered

Here are some video demos of the design in action. For the non-musician: pay attention to the pitch changes of the notes at the times when the pedals are pressed or released.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        <iframe width="770" height="480" src="https://www.youtube.com/embed/OuNCbN86QOk" title="Electronic string bender guitar demo" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
    </div>
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        <iframe width="770" height="480" src="https://www.youtube.com/embed/Mp2r41Q7cyQ" title="Lulu swing (servo motor bender guitar demo)" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
    </div>
</div>

This is a very impractical thing, but I like the way it sounds and there are a number of qualities about it that attracted me to it as an interesting and challenging engineering project. I'll explain the design and where the challenges stem from.

## Choice of Motors

The first and perhaps most important part of the design is to determine the proper type of motor to use. Accurate position control is a requirement, so the most obvious answer might seem to be the type of hobby servo motors that are cheap and widely used in RC vehicles. However, these servos are typically heavily geared down DC motors that only travel in a <360 degree arc. The gearing creates lots of audible acoustic noise which is a major issue in a musical instrument. The constrained arc of travel also made it difficult to come up with an elegant mechanical design.

Another option would be small stepper motors. These don't necessarily require positional feedback, but tend to be quite large in order to get the needed torque for this application without having to gear down. Some initial tests were performed using the tuning machines from an old Gibson robot tuner assembly which use stepper motors but they were far too loud and slow to work for this application.

I chose instead to opt for 3 phase brushless DC motors that are commonly used in quadcopter drones. These motors have the advantage that they have a very high torque-to-weight ratio in a very small package. When the right control scheme (Field Oriented Control) is used, they can be very quiet and move much faster than the hobby servos mentioned above. However, these advantages come with greatly increased complexity in terms of control scheme and needed electronics to drive the motor. This will be discussed in more detail later.

These drone motors are not designed for position control in any way. Due to the strong permanent magnets used they can feel very 'coggy' when they are turned. What this means is the rotor wants to 'snap' to certain spots when no current is running through the coils. This creates headaches for accurate position control, but can be worked with if there is enough static friction to keep the rotor in place.

To get the required positional feedback from the motors an encoder assembly needed to be created. The motors were modified with a diametric magnet on the back that would sit above a magnetic encoder IC.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets\img\chief-2207-2050kv-motors-v2.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets\img\servo_exploded.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Left: example of a BLDC drone motor. Right: exploded servo assembly.
</div>

## Mechanical Design

I opted to use the threads for the prop on the drone motors as a sort of lead screw. This helped to make the design less complex and avoid having to use gears. Each string bender servo actuates a lever which rotates around a hardened steel axle. The end of the guitar string is anchored around the lever, and this serves to increase or decrease the tension of the string. The top of the lever attaches to an extension spring which is adjusted to counterbalance the tension of the guitar string. This helps to reduce the load on the motors as well as reduce binding in the mechanical parts.

The Lever assembly was designed around off-the-shelf hardware where possible and was 3D printed out of steel.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets\img\lever_exploded.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    exploded render of lever assembly
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets\img\bender3.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    render of lever assembly and counterbalance spring
</div>

The chassis to house the mechanical components was 3D printed out of a nylon material that is infused with glass beads for added stiffness.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets\img\Screenshot (155).png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets\img\Screenshot (156).png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    renders of mechanical components housed in chassis.
</div>

This entire mechanical assembly was installed into a routed out guitar body as is shown in the image at the top of the page. An off-the-shelf guitar bridge with rolling string saddles to reduce friction was modified to allow the strings that would be bent to exit through the rear of the bridge, and teflon tubing was added to prevent string breakage due to rubbing.

The pedal assembly was designed around 4 modified piano sustain pedals. A magnet and a linear Hall effect sensor were added to each pedal to produce a continuous voltage signal corresponding to how far down the player is pressing the pedal. The enclosure for the pedals was fabricated out of bent sheet metal.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets\img\IMG_9131.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets\img\IMG_9181.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Left: render of pedal assembly. Right: completed pedal assembly.
</div>

## Electrical Design

Each servo requires 3 Half-H-Bridges. A three-phase BLDC gate driver IC was used to drive the H-Bridges. The H-Bridges, gate driver, and current sense resistors for each motor were placed on a separate PCB that can be easily replaced since these parts are the most sensitive to getting smoked when anything goes wrong.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets\img\powerStage.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    power stage PCB
</div>

As mentioned earlier, driving the motors quietly and efficiently while being able to get maximum torque out of them requires a somewhat complicated control scheme called Field-Oriented Control (FOC) as opposed to the typical open-loop drive ESC drive that is used for drones. FOC can be difficult to get right and requires precise timing and alignment of the PWM signals into the gate drivers and the ADC readings for the motor phase currents. Thankfully, Trinamic offers an IC that does FOC very well. This IC (the TMC4671) is a bit expensive, but since this is a personal project and we are not being super cost-conscious it fits the bill. Tuning all the different FOC parameters in order to make everything work still requires quite a bit of know how, but all together it's a lot easier than trying to roll your own.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets\img\TMC4671.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    simplified block diagram of the TMC4671
</div>

Since the TMC4671 is doing most of the heavy lifting for the motor control only a basic STM32 ARM Cortex M0+ microcontroller is needed to run the rest of the firmware tasks. For wireless communication with the pedal assembly, NRF24L01+ radio modules are used. These radio modules are cheap and simple with plenty of existing documentation. All of the rest of this circuitry goes on another PCB. The guitar circuitry is powered by a 3S Lipo battery. It's important to have a sufficiently low impedance path directly from the battery to each motor power stage as theses motors can draw many amps of current. Without this low impedance path, voltage drops that occur when a motor suddenly turns on will wreak havoc on noise-sensitive components in the circuit like the radio module.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets\img\quadServo.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets\img\IMG_1733.jpg" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    testing the electricals wired up to the mechanical assembly (missing back cover on the chassis)
</div>

## Noise Considerations

One of the challenging aspects of this project is the elimination of electromagnetic interference in the guitar signal. Guitar pickups are very sensitive to noise and interference of exactly the type that electric motors create. For an idea of what this noise sounds like, take a look on youtube for the videos of Eddie Van Halen using an electric drill above his guitar pickups on "Poundcake". It's a neat effect, but awful if you're still trying to play the guitar. To keep as much of this noise as possible out of the pickups, the best things to do are:
 - place the motors as far from the pickups as possible
 - reduce the load on the motors as much as possible with mechanical counterbalancing. More current through the motors creates a stronger magnetic field
 - use a MuMETAL shield around the motors
