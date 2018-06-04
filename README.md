# GJ2-Pixhawk

This documentation is for the **flight controller setup** on the Ginger Judge 2, but other pieces of this project will be referenced too. The Ginger Judge is a remotely-piloted research vessel with on board instrumentation. (The on-board instrumentation isn't exactly "on-board" yet, that can be a problem for v2.1)

## Overview

The Ginger Judge 2 uses a [Pixhawk](https://pixhawk.org/start) flight controller currently running on [Ardupilot](http://ardupilot.org/ardupilot/index.html) firmware. This can all be controlled from a ground station of your choosing, I've listed a few below.

 - [Mission Planner](http://ardupilot.org/planner/docs/mission-planner-overview.html) - Windows, Mac OS X ([Using Mono](http://nathan.vertile.com/blog/2016/12/22/running-ardupilot-mission-planner-with-mono-on-mac-osx/))
 - [APM Planner 2.0](http://ardupilot.org/planner2/) - Windows, Mac OS X, Linux
 - [QGroundControl](http://qgroundcontrol.com/) - Windows, Mac OS X, Linux, Android and iOS

A more complete list of ground stations can be found [here](http://ardupilot.org/copter/docs/common-choosing-a-ground-station.html).

The Ginger Judge 2.0 was controlled with Mission Planner using a Microsoft Surface Pro 3, but feel free to use another supported ground station based on your requirements

## Getting Started

Let's start with the physical components. The Pixhawk, seen below, is the physical device that tells the throttle and steering components what to do.
![Pixhawk](https://lh3.googleusercontent.com/ixZTlZA7BFhjgoCuHDQP_iNWVLQw587ahcejcuuwv-xcpYd_98tKHbe31dhFLEmttXi39GzCHSc =200x225)
For our purposes, we don't need to worry about all of the connections on the device, but there are a few we should be familiar with.

### Connections

Our Pixhawk receives power (5v) through the TELEM 2 connector. This is also how it receives serial communication from a Raspberry Pi.
![Pixhawk & Pi serial wiring](https://lh3.googleusercontent.com/O3lXHDz-5hCAq-frIfHvYL6HvzFEEyp18WHTUVHQ139x6B-GcNOsDnL8cl8C8POwh8wiy8X0r6oH "Pixhawk-Pi")
For more information on how this communication process works and for setting up a Raspberry Pi, click [here](http://ardupilot.org/dev/docs/raspberry-pi-via-mavlink.html).

The Pixhawk's SWITCH connector would normally have a safety switch connected to it that needs to be enabled/disabled to launch. Since our system is enclosed within a Pelican case this switch is inaccessible, so to deal with this the switch parameter is set to default off (ready for launch). Parameters are an important part of setting up our flight controller and are covered in their own section below.
![Safety switch](https://lh3.googleusercontent.com/lNZfQnV6G2LhMQHW_KQttDf0oZ6vV0NnR4I21gZhNnKFvQiu9RrJygO4dkqSXl7oRVN3EOc3KQz- "Safety Switch" =200x225)
Since the switch is set to default off we don't even have it connected. It is possible to remote toggle if you choose to enable it, but the way we run the system this proved to be quite the hassle.

Our Pixhawk has a single GPS/Compass connected to the GPS and I2C connections respectively. The image below shows that wiring along with a secondary GPS if one is ever added.
![Pixhawk dual GPS wiring](https://lh3.googleusercontent.com/qY4zZLHdLIbfX-5BE_9ssfNtnGVgDCpevBIyBpxup2r4s_0QcjccAD-KBKGeuAM7VPEpcLG5kXLa "Pixhawk-GPS")

### Main Output

You'll see on the side of the Pixhawk there are 3 rows of 16 pins. We only need to deal with a few of these. 1-8 (without the white background) are the main output pins and the only pins we deal with for this project.
![Pixhawk servo rail](https://lh3.googleusercontent.com/F_eJmiOKs-NUVQkVwdt3ZqP2lin2tG6MJ5mVYcdIHK5OOThaHmTbHLJY6x_rZWHlvg5NeYbsFzHF "Pixhawk Pins")

 1. Radio control receiver input
 2. S.Bus output
 3. Main outputs (we use these)
 4. Auxiliary outputs

The outputs we use are pin 1 (steering) and pin 3 (throttle) of main output. The signal (bottom rail) will output a PWM signal to the respective control devices. The ground pin (top rail) we have used is pin 1 paired with the steering arduino connection. Remember, it's very important you have the correct signals going to the correct devices. Pin 1 (steering) goes to the arduino and pin 3 (throttle) goes to the teensy.

More information for these devices and their code can be found below...

[Steering/linear actuator](https://github.com/mazhigbee/GJ2)
[Throttle](https://github.com/nickdossantos/ginger_judge2)

In short, the only 3 connections we care about are signal to steering, signal to throttle, and ground.

### Ground Control

This is going to be a brief rundown on what you need to know to use Mission Planner (the ground control station I chose to use for this project) to control the GJ2.

![Mission Planner](https://lh3.googleusercontent.com/HaqpISNlPlEXRK7bH_BgUEFlz8bBQGs9WghVZxg6aoRJqW0_ozcohhsUEWyaB20HWTJxB47rOKlb "Mission Planner")
This is what you will see when you launch Mission Planner. You can find everything a more detailed overview on the software at [their website](http://ardupilot.org/planner/) but I'll tell you a few things you need to know.

You will be disconnected from the system when you first launch the software and knowing how to connect is important. You will select TCP in the dropdown to near the connect button in the top right. When you hit connect after that you will be prompted to enter an address. This address must be whatever address the raspberry pi is set up with. I'm not going to put that here as it shouldn't be public but you should have access to it if you're working on the project. Assuming the router on the GJ2 is up and running then you should connect successfully.

This connection works because the raspberry pi is communicating with the Pixhawk via MAVLink protocol which was described earlier in **Connections**.

### Parameters

The ardupilot parameters get their own section because it's very important to understand what they are, what they do, and how to change them for this project.

The Pixhawk is expecting to be running directly on a drone, plane, or rover. All of the testing and set up for the Pixhawk for our project had to be done off the boat which ended up being difficult. There are tons of safety parameters that won't allow the system to arm or launch if requirements aren't met. The parameters that were left on the device at the end of the original Ginger Judge 2 development allow for the system to arm and launch under almost any circumstance, connected or disconnected from the system.

If for some reason these parameters are lost or the Pixhawk needs to be reflashed with new firmware and started from scratch then you'll need to work through all the arming checks that don't allow the system to activate and disable any restricting parameters. These parameters can change depending on firmware or version so keep in mind you may need to do some googling to find out which parameter does what. Disabling RC input requirement for example. This one is important since we have no RC controller or input.

A full parameter list can be found [here](http://ardupilot.org/copter/docs/parameters.html).