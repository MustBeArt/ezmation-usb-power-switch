# ezmation-usb-power-switch
Reverse engineering and usage notes for the USB power switch from [EZmation.com](https://EZmation.com)

## Supplied Documentation

The documentation provided by EZmation for their USB-controlled AC power switch is very brief and not entirely applicable to that specific product. No documentation ships with the product, but some info appears on the product's web page. For reference, here is a copy of what appears there as I write this:

> 
```
Upon application of utility power the device will energize the socket and turn the switch LED RED to indicate power is ON
Power may be toggled by pressing the local power button. The LED will go off when power is OFF
Power may also be turned on or off by sending RS232 commands through the DB9 port at 115200 BPS, N81
  ON => Turns the socket on
  OFF => Turns the socket off
  ? => returns current status, ON or OFF

Specifications

Max Load: 10A / 2200 W
Serial Connectivity: USB Virtual COM Port, RS232 @ 115200N81
Firmware Upgrade: Via serial port
Also available with a DB9 serial port.
Hardware/Firmware customizations available upon request.
```

Here's a list of things in the above text that are just wrong:
* There is no LED on the outside of the  device.
* There is no local power button on the outside of the device.
* There is no RS232 or DB9 port on the device. Instead, there is an FTDI USB Type A to async serial adapter. It provides a physical (not virtual) serial port, which is hardwired to the device.

Here's an incomplete list of questions that are not answered by the above text:
* Does the device respond when AC power is not connected?
* What line ending is sent by the device? CR (Mac), LR (Unix), CRLF (DOS)?
* What line endings are accepted by the device?
* What, if anything, does the device send on powerup?
* What, if anything, does the device send in response to ON or OFF commands?
* What, if anything, does the device send in response to erroneous commands?
* How is firmware upgrade accomplished?

## Device Inspection and Teardown

Here's what the device looks like out of the box. We see a generic metal outlet box from China with duplex 3-prong outlets, just like you'd find at the hardware store. We see a generic 3-prong AC power cord. A narrow black cable terminates in a blue housing with a USB Type A plug at one end.

![Exterior View](images/teardown0372.jpg)

Removing two screws allows us to flip the lid of the outlet box open. Under the lid we see the duplex outlet, wrapped in some electrical tape to keep the bare terminals from shorting to anything inside the box. Remaining in the main box we see a white module. There's nothing else inside.

![Lid Opened](images/teardown0373.jpg)

After pulling out the white module and rearranging things a bit, we can see that two of the wires from the AC power cord (white and black) enter the module on the input side, and the other wire (green) goes to the ground terminal of the metal box. 

Red and black wires leave the module on the output side and supply power to the duplex outlet.

The black cable from the USB connector contains four wires. The red wire doesn't to anywhere; it is terminated in a DuPont connector and is left loose in the box. The other three wires (white, black, and green) go into the module.

![What's Inside](images/teardown0376.jpg)

Here's a good look at the label on the white module. It's a Sonoff Wi-Fi Smart Switch, model BASICR2, rated for 100-240V AC, 50/60 Hz, 10 amps resistive load. It supports WiFi 802.11b/g/n at 2.4 GHz. Operating temperature range is specified as 10T40 (10 to 40 degrees C). FCC ID 2APN5BASICR2, IC 29127-BASICR2.

The FCC filing includes a User Manual for the module, along with photos and other documents. FCC operates a [FCC ID lookup page](https://www.fcc.gov/oet/ea/fccid) where you can search by FCC ID and download any non-proprietary documents related to the filing.

This module is out of production, replaced by a BASIC R4 version. Nonetheless, as of this writing EZmation has this version offered for sale on their web site for just $9.49.

![Sonoff Device](images/teardown0378.jpg)

Inside the module we find a single circuit board. This side of the circuit board contains the AC power handling, along with a small power supply for the digital circuitry, based on an LP3669 switching regulator IC.

We can also see the three wires from the USB connector entering at J2. There's a fourth position on J2, which is unused.

S2 is a pushbutton switch with a tall black button that reaches through the top housing of the module, where it is unlabeled. D5 is an LED. These components are probably what the author of the web site documentation had in mind when mentioning a button and a LED, though there's no provision for either of them to be usable from outside the metal box.

![AC Power Switching Side](images/teardown0379.jpg)

The other side of the circuit board is dominated by the digital circuitry, though it also includes some beefy power traces.

At the lower left we see U5 is an Espressif ESP8285 WiFi System-on-a-Chip, which is essentially an ESP8266 with a megabyte of flash memory built in. The clock crystal is marked 26.000 MHz.

At the edge of the circuit board the pins of J2 are clearly labeled. Pin 1 (the square pad on the right) is the unused position, labeled 3V3 (that is, 3.3 volts DC power). Pin 2 is labeled RX and is hooked up to the green wire from the USB connector. Pin 3 is labeled TX and is hooked up to the white wire. Pin 4 is labeled GND and is hooked up to the black wire. This tells us the architecture of the control port: the blue housing that contains the USB Type A connector must also contain a USB to async serial adapter chip, and the signals traveling within the black wire are async serial data, receive and transmit, very likely at 3.3 volts.

U2 is a MicrOne ME6211A voltage regulator in a SOT89-3 package.

At the lower left corner of the board the WiFi antenna is visible as a wiggly trace in the copper. This USB-controlled power switch product apparently doesn't use the WiFi capabilities of the ESP8285 SOC, but the Sonoff module looks like a cost-effective way to source a microcontroller integrated with a suitable AC power switching relay -- especially if the company got a good deal on a batch of BASICR2 modules after they were replaced by later versions.

![Digital Side](images/teardown0380.jpg)
![Serial Port and Processor](images/teardown0384.jpg)
