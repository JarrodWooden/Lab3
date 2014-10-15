Lab3
====

Talking from the MSP430 to the Nokia LCD Screen

(This marks the end of the Mega Prelab.)
---------------------------------------------------------------
## Logic Analyzer
The answers to the logic analyzer section will be posted to GitHub along with the functionality code.
###Physical communication
Connect the Nokia 1202 Booster Pack to your TI Launch Pad.  Make sure that the buttons on the Booster Pack are pointed away from the USB connector (and on the same side of the board as the MSP430 buttons), just like in the following picture.
(Picture can be found on the ECE382 website under "Lab 3")

Download <a href="lab3.asm">lab3.asm</a> and build a project around the file.
Run the program and observe the output on the LCD every time you press the SW3 button.  It should look something like the following image after a few button presses.<br>
(Picture can be found on the ECE382 website under "Lab 3")

When SW3 is detected as being pressed and released (lines 56-62), the MSP430 generates 4 packets of data that are sent to the Nokia 1202 display, causing a vertical bar to be drawn. Complete the following table by finding the 4 calls to writeNokiaByte that generate these packets. In addition, scan the nearby code to determine the parameters being passed into this subroutine. Finally, write a brief description of what is trying to be accomplished by each call to writeNokiaByte.

|Line|R12|R13|Purpose|
|:-:|:-:|:-:|:-:|
| 66 | move #NOKIA_DATA | #0xE7 the value to be drawn to the LCD screen | To Draw 1110 0111 to the first column an starting at the first row down |
| 276 | #NOKIA_CMD | xB1 - Was what the column was but switched to what row we want to start drawing to | Sets the cursor on the correct row.  |
| 288 | #NOKIA_CMD | x10 - Would set where the column is for the first set of bits (need two sets of bits for all the columns | Sets the cursor on the first three bits if the cursor needs to be in the first three bits  |
| 294 | #NOKIA_CMD | mask upper bits x01 - set second set of bits for the column | Sets the cursor on the second set of bits if it needs to be there. |


Configure the logic analyzer to capture the waveform generated when the SW3 button is pressed and released. Decode the data bits of each 9-bit waveform by separating out the MSB, which indicates command or data. Explain how the packet contents correspond to what was drawn on the display.  Be specific with the relationship between the data values and what and where the pixels are drawn

In the below Picture all the packages sent to the LCD:

The first line in the Waveform is the RESET pin
The second line is the Clock pin
The last line (third line) is the MOSI pin

![alt text](https://raw.githubusercontent.com/JarrodWooden/Lab3/master/AllPackages.jpg "All Packages Sent")

Below shows a close up of the first package sent (which is a Data package, not a command):

![alt text](https://raw.githubusercontent.com/JarrodWooden/Lab3/master/FirstPackage.jpg "First Package/ Data Package")

Here is the Second Package:

![alt text](https://raw.githubusercontent.com/JarrodWooden/Lab3/master/SecondPackage.jpg "Second Package/ Command")

Third Package:

![alt text](https://raw.githubusercontent.com/JarrodWooden/Lab3/master/ThirdPackage.jpg "Third Package/ Command")

Fourth Package:

![alt text](https://raw.githubusercontent.com/JarrodWooden/Lab3/master/FourthPackage.jpg "Fourth Package/ Command")

The following Table corresponds to the above pictures. E7 is the data that is sent from picture "First Package" and tells the
LCD what it is going to draw 1110 0111.

B1 is sent as command for the first row since it masks out the B, and it'll leave the 1 for the row.

Then the Third Package and fourth package would be the first column.

|Line|Command/Data|8-bit packet|
|:-:|:-:|:-:|
| 66 | Data | E7 |
| 276 | Commamd | B1 |
| 288 | Command | 10 |
| 294 | Command | 01 ||

Hint: in order to probe the signals while the LCD is connected to the LaunchPad, you will need to use the LaunchPad header pins with the probe hook grippers. Be careful when attaching and detaching the grippers to the pins, as they may easily bend and then no longer serve you well. Also, don't forget the ground pin!<br>

![Logic analyzer connection](left.jpg)<br>

You will get a waveform similar to that shown below. Note that the command/data bit is significantly far away from the 8 data bits. <br>

![alt text](https://raw.githubusercontent.com/JarrodWooden/Lab3/master/LA_datastream.jpg "LA Datastream")

Next, setup the Logic Analyzer to capture the RESET signal on a falling edge. Measure the duration that the RESET line is held low in the initNokia subroutine. Hint, the code to hold the reset line low can be found on lines 93-100. 

When running this with the logic analyzer, I got that the reset loop took 18.849ms (reading from when the RESET Pin went low and then back high)

Below is the waveform from the logic analyzer:

![alt text](https://raw.githubusercontent.com/JarrodWooden/Lab3/master/ResetLowPulse.jpg "Reset Loop Waveform")

How many counts does the firmware loop count down from?  0xFFFF or 65,535.

Using the delay you just measured and the number of counts, calculate the amount of time each iteration of the delay loop consumes: So it would be 18.849ms divided by 65,535 counts which equals 28.76 micro seconds.

###Writing modes
The native write operation to the Nokia 1202 will overwrite any information that is was on the display with new information.  However, that may not be the best course of action in your application.  The new bits being added to the image may be merged using the AND, OR, XOR operators.  To do this treat a black pixel as a logic 1 and a white pixel as a logic 0.  The pixel values from the same locations are combined using a logical operator and placed at the corresponding location in the destination imaged.
Import the following image into a paint program and show the result of the operation between the two bits maps combined using the logic operator specified.

Here is the result of the AND, OR, and XOR operators:

![alt text](https://raw.githubusercontent.com/JarrodWooden/Lab3/master/bitblock.bmp "XOR BitMaps")

## Functionality
Required functionality: Create a block on the LCD that is 8x8 pixels.  The location of the block must be passed into the subroutine via r12 and r13.
A functionality: Move the 8-pixel block one block in the direction of the pressed button (up, down, left, right).
