---
layout: post
author: Hemachander Rubeshkumar
tags: [portfolio, completed, dactyl-manuform]
permalink: /dactyl-manuform/
---

## A project several years in the making

When I first started on this project, i anticipated some learning pains. It was my first time trying an independent project on this scale, with a few major hurdles: soldering, a limited budget, custom firmware, and case file generation. 2-3 years, several broken soldering irons, and a bricked microcontroller later, I am proud to say that I am typing this entirely on *my* dactyl manuform. 

## Initial Goals
I had been admiring custom keyboards for a while and wanted to really go all in on making my own. At first, the appeal of a custom keyboard was it's sound profile and feel. But I reasoned that if I was going to do this, I didnt want any regrets, and researched every minutae to make the perfect keyboard for me. I was really drawn to the practicality of the smaller form factor boards, but I found out that the typical staggering of keys harkened back to the days of the typewriter, and it wasnt very ergonomic (comfort and efficient), which led to columnar stagger, opposed to the row stagger of the usual board. The idea is each finger stays in its column. Then, if you curve each column and account for finger lengths, split the board into 2 halves and tilt it to make it easier on your wrists, you get a dactyl manuform. There are some common sizes for these boards, but I chose to take a 5x6 layout and only have 3 thumbkeys instead. I thought this way there would be less effort placed on the thumbs, and checked that I still had all the keys I could possibly need. 

### The Case
To make the case with my specifications, I found a widely available OpenSCAD script, with parameterized dimensions for finger lengths, amount of keys, how much to tilt the wrists etc. Once I was able to get a decent set of parameters in, I had them printed with the generated .stl model, and set the slicer to 3 walls 35% infill to get the final g-code. Then, it just had to get printed. Each half took approximately 30 hours to fully print, but after the supports were removed, it was good to go. 

### The Acoustics 
I really enjoyed the feel of using clicky switches, and eventually landed on the Kailh White Owl V2s. They feel great to type on, and to let the clickyness shine through, I chose some lighter doubleshot PBT keycaps, letting the higher frequency noises through. I am thinking of using some PE foam when I add some coins to weight it down later. However, I like the way it sounds right now.

## Electronics
Designing the case handled most of the goals I had, but to make a functional keyboard, especially with this unusual form-factor I needed to create a diode matrix. Most standard membrane keyboards line up the wires in a way that causes issues when multiple keys are pressed. To avoid this, I opted for a diode matrix. Keyboard are essentially giant grids, with rows and column wires. When a switch in pressed, it closes the column and row wires. The microcontroller sends a high signal to each column one at a time and checks to which rows read a high signal. Now, cheaper keyboards don't use diodes, which means that when switches (1,3), (3,3), (1,1) are pressed, the microcontroller cant tell if switch (3,1) is pressed or not. This is because that high signal travels up to the 4th corner of the square. To avoid this problem, we use diodes on each switch, to prevent the signal from travelling to other rows and columns. 

Because this is a split keyboard, there needs to be a way for these halves communicate. I went with a TRRS jack, the same ones you might use for headphones. Each ring is its own wire, and for this project I implemented QMKs "bitbanging" protocol. 

### Soldering
Now that we've covered why we are using a diode matrix, lets cover what was required to make it happen. I started this project with no supplies on hand, so there was quite the initial investment required. However, in hindsight, I was far too stingy and it cost me. I ordered 6 spools of 24 AWG wire, cheap wire stripper, 100 Diodes (1N4148 DO-35), lead-free solder, cheap $10 soldering iron and stand at first. 

I started by pushing all the switches into the top of the case, and bending and clipping the diodes row by row. If the pulses were from the columns, then the diodes needed to match that orientation (stripe is the end). Then, with that set up, it was time to solder them together.

> This was the most painful part. I could get through maybe 7 solder joints before the soldering iron tip failed. I was sure I was simply doing something wrong, so I bought another, and another. Eventually I realized that I just needed better equipment, and got the Pinecil. It worked perfectly and sped up my progress a lot. (I'm not affiliated in any way, just wanted to share my experience). 

After I got a decent soldering iron, the solder joints were more tedious than anything. With the diode rows complete, I needed to solder the columns together, with wire segments between switches in the same columns. I chose to color code to keep track of the wires, and completed that. It took a little bit as I had to manuever the iron to avoid accidently melt random parts of the case or the switch housings. Next, these rows and columns needed to be wired to the microcontroller. The left and right halves are different, so keep an eye on the wiring guide. Finally, to get the TRRS cable to work and connect both halves, I soldered the power, ground, and signal pins on both sides. 

## QMK Firmware

After designing, generating case files, printing parts, sourcing components, and meticulously soldering the complete diode array through many broken irons, I had finally gotten the keyboard manufactured. I even went through simulations to optimize its layout. Now it was time to program it. 

To program the keyboard with QMK, the final steps are to flash a compiled .hex file using the QMK Toolbox. To compile that .h file, you need a properly configured rules.mk file, keymap file (.json or .c), and a config.h file.

Let’s start with the keymap file. QMK offers a lot of support and features, and using it to create firmware was fairly straightforward. There were a few key items that made this build a bit different than most boards however. Due to the specific configuration of keys I had chosen, a 5x6 dactyl manuform with 3 thumb keys, there weren’t exact pre-configured files I could directly copy. I would have to actually read through the documentation and write my own version to get this to function. Fortunately, I could look at similar layouts, such as a 5x6 dactyl manuform with 6 thumb keys. QMK offers a web-based configurator that generates a .json file, which stores key layout information. For my purposes however, I needed a .c version of this file. Fortunately, there is a command in QMK MSYS that handles this, and I was able to use the configuration tool, download the .json file, and convert it to a .c file. The reason I needed to do this was to implement the “()” key; ‘(‘ when pressed, ‘)’ when pressed with shift.  That extra logic couldn’t be implemented directly in the web configurator, but in the c code, manual “key overrides” could be used to implement this behavior. Once I had this converted .c file, I had to remove the extra keys from the thumb cluster add that extra key override logic, and it was good to go.
![QMK Configurator](https://www.hemachander65.github.io/images/dactyl1.png)
<div style="text-align: center;">
  QMK's Web Configurator Tool
</div>
 
### Rules.mk
The rules.mk file was almost entirely blank, save for the following line: 
`KEY_OVERRIDE_ENABLE = yes`
This was essential to implementing the ‘()’ key.

> Unfortunately I was unable to get this cool key working as of writing this, but I might return to it at a later date

### Info.json
Info.json handles more of the highlevel information, such as author, microcontroller, bootloader type, etc. It also handles logic concerning the diode matrix. Since not every “coordinate”, or column header and diode row combination, is utilized, there is a segment definiting which switches are used. This is used later in conjunction with the layout file to assign matrix coordinates with the proper “key”. 
 
 ```
 /* LAYOUT [REALITY]

	{Left-Hand}						{Right-Hand}
	WMWMWMWMWMWMWMWMWMWMWMWM		WMWMWMWMWMWMWMWMWMWMWMWM
	MWM[+][+][+][+][+][+]WMW		MWM[+][+][+][+][+][+]WMW
	WMW[+][+][+][+][+][+]MWM		WMW[+][+][+][+][+][+]MWM
	MWM[+][+][+][+][+][+]WMW		MWM[+][+][+][+][+][+]WMW
	WMW[+][+][+][+][+][+]MWM		WMW[+][+][+][+][+][+]MWM
	MWMWMWMWM[+][+][+][+]WMW		MWM[+][+][+][+]WMWMWMWMW
	   WMWMW[+][+][+][+]MWM		WMW[+][+][+][+]MWMWM
	   MWMWMWMWMWMWMW[+]WMW		MWM[+]MWMWMWMWMWMWMW
	            MWMWMWMWM		WMWMWMWMW

*/
/* LAYOUT [QMK]

	{Left-Hand}
		[1][2][3][4][5][6] (Reads Left Column Pins)
	    WMWMWMWMWMWMWMWMWMWMWMWM
[1]	    MWM[+][+][+][+][+][+]WMW
[2]	    WMW[+][+][+][+][+][+]MWM
[3]	    MWM[+][+][+][+][+][+]WMW
[4]	    WMW[+][+][+][+][+][+]MWM
[5]	    MWMWMWMWM[+][+][+][+]WMW
[6]        WMWMWMWMWMWMWM[+]MWM
                    WMWMWMWMW

	{Right-Hand}
		[1][2][3][4][5][6] (Reads Right Column Pins)
	    WMWMWMWMWMWMWMWMWMWMWMWM
[7]		MWM[+][+][+][+][+][+]WMW
[8]		WMW[+][+][+][+][+][+]MWM
[9]		MWM[+][+][+][+][+][+]WMW
[10]	WMW[+][+][+][+][+][+]MWM
[11]	MWM[+][+][+][+]WMWWMWMWM
[12]	MWM[+]WMWWMWMWMWMWMW
	    MWMWMWMWM
*/

//Note: Rows 1-6 are defined as Left Rows, and Rows 7-14 are defined as Right Rows

/* Wiring according to numbers on PCB for ATMega32U4
Columns Left:

 4, 5, 6, 7, 8, 9
D4,C6,D7,E6,B5,B4

Rows Left:

A1,A0,15,14,16,10
F6,F7,B1,B3,B2,B6

Columns Right:

 4, 5, 6, 7, 8, 9
D4,C6,D7,E6,B4,B5

Rows Right:

A1,A0,15,14,16,10
F6,F7,B1,B3,B2,B6
*/

/* Define Matrix */
#define MATRIX_ROWS 12 //QMK treats the RH side as connected to the same matrix as the LH
#define MATRIX_COLS 6  //QMK uses the right side pin definitions for switches in rows 7-12

#define MATRIX_COL_PINS { D4, C6, D7, E6, B4, B5 }
#define MATRIX_ROW_PINS { F6, F7, B1, B3, B2, B6 }
 ```
![Info.json](https://www.hemachander65.github.io/images/dactyl2/)
<div style="text-align: center;">
  Code snippet in info.json handling diode matrix
</div>

### Config.h
The config.h file handles toggling certain features and handles important logic for the split keyboard. For both halves of the keyboard to communicate, they are connected to each other through a TRRS cable, which in my case, is operating through serial. This uses the bitbang driver. Both halves of the board have common power and ground (connected through the TRRS jacks and cables on both sides) as well as an assigned digital pin. This pin is defined in config.h. Additionally, the diode matrix and wires are all defined here as well. 

![Info.json](/images/dactyl3a/)
![Info.json](/images/dactyl3b.png)
<div style="text-align: center;">
  Code snippet in info.json handling diode matrix
</div>
![Info.json](http://www.hemachander65.github.io/images/dactyl4/)
<div style="text-align: center;">
  ASCII Art/Diagram comparing physical keyboard to firmware, and pin definitions in config.h
</div>

After properly setting up these “code snippets of Exodia”, we can finally compile this into a hex file in QMK MSYS.`
![Info.json](http://www.hemachander65.github.io/images/dactyl5a.png/)
![Info.json](/images/dactyl5b)
<div style="text-align: center;">
  QMK MSYS output of the Final, Successful Compilation of Firmware
</div>

There are several setups for dealing with split keyboards, however the setup I went with for this board is to have QMK assume the USB is always connected to the left-half. Conveniently, this is an option where you can flash the same exact file onto both boards. To flash the boards, I simply disconnected the TRRS cable, and plugged in one microcontroller at a time, setting the toolbox to auto-flash, and using a bent wire to trigger a reset by shorting the reset and ground pins. Once one side was done, the other side was also flashed. To test that everything worked properly, I used QMK’s key tester online. (basically it logs the inputs, and you press all the keys on your keyboard to test.)
 
My first test revealed some poor solder joints and other errors, but after cleaning up my earlier solder work, I noticed that I was unable to program on the right half's pro-micro. I fortunately bought a set of 3, so I cut out the old controller and soldered in the new one. 

## Conclusion
There was quite a lot I got to explore through completing this, soldering, writing firmware outside of arduino, sourcing components, etc. But I really did learn a lot, and having a solid completed project I have actually used for weeks, and knowing how it works well enough to fix or improve it however I want is really empowering.

DEBUGGING:
{% assign image_files = site.static_files | where: "image", true %}
  {% for myimage in image_files %}
    {{ myimage.path }}
  {% endfor %}