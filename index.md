QMK Firmware:
After designing, generating case files, printing parts, sourcing components, and meticulously soldering the complete diode array through many broken irons, I had finally gotten the keyboard manufactured. I even went through simulations to optimize its layout. Now it was time to program it. 
To program the keyboard with QMK, the final steps are to flash a compiled .hex file using the QMK Toolbox. To compile that .h file, you need a properly configured rules.mk file, keymap file (.json or .c), and a config.h file.
Let’s start with the keymap file. QMK offers a lot of support and features, and using it to create firmware was fairly straightforward. There were a few key items that made this build a bit different than most boards however. Due to the specific configuration of keys I had chosen, a 5x6 dactyl manuform with 3 thumb keys, there weren’t exact pre-configured files I could directly copy. I would have to actually read through the documentation and write my own version to get this to function. Fortunately, I could look at similar layouts, such as a 5x6 dactyl manuform with 6 thumb keys. QMK offers a web-based configurator that generates a .json file, which stores key layout information. For my purposes however, I needed a .c version of this file. Fortunately, there is a command in QMK MSYS that handles this, and I was able to use the configuration tool, download the .json file, and convert it to a .c file. The reason I needed to do this was to implement the “()” key; ‘(‘ when pressed, ‘)’ when pressed with shift.  That extra logic couldn’t be implemented directly in the web configurator, but in the c code, manual “key overrides” could be used to implement this behavior. Once I had this converted .c file, I had to remove the extra keys from the thumb cluster add that extra key override logic, and it was good to go.
 
QMK Web Configurator Tool
The rules.mk file was almost entirely blank, save for the following line: KEY_OVERRIDE_ENABLE = yes
This was essential to implementing the ‘()’ key.
Info.json handles more of the highlevel information, such as author, microcontroller, bootloader type, etc. It also handles logic concerning the diode matrix. Since not every “coordinate”, or column header and diode row combination, is utilized, there is a segment definiting which switches are used. This is used later in conjunction with the layout file to assign matrix coordinates with the proper “key”. 
 
Code snippet in info.json handling diode matrix
The config.h file handles toggling certain features and handles important logic for the split keyboard. For both halves of the keyboard to communicate, they are connected to each other through a TRRS cable, which in my case, is operating through serial. This uses the bitbang driver. Both halves of the board have common power and ground (connected through the TRRS jacks and cables on both sides) as well as an assigned digital pin. This pin is defined in config.h. Additionally, the diode matrix and wires are all defined here as well. I had accidentally flipped two data wires, but fortunately I could define them here to correct that error. 
  
ASCII Art/Diagram comparing physical keyboard to firmware, and pin definitions in config.h
After properly setting up these “code snippets of Exodia”, we can finally compile this into a hex file in QMK MSYS. 
  
QMK MSYS output of the Final, Successful Compilation of Firmware
There are several setups for dealing with split keyboards, however the setup I went with for this board is to have QMK assume the USB is always connected to the left-half. Conveniently, this is an option where you can flash the same exact file onto both boards. To flash the boards, I simply disconnected the TRRS cable, and plugged in one microcontroller at a time, setting the toolbox to auto-flash, and using a bent wire to trigger a reset by shorting the reset and ground pins. Once one side was done, the other side was also flashed. To test that everything worked properly, I used QMK’s key tester online. (basically it logs the inputs, and you press all the keys on your keyboard to test.)
 
My first test revealed some poor solder joints and other errors

