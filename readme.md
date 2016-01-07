OVERVIEW
---------
This repository contains the basic steps to program the Garmin AIS600 using a macbook. It’s more of a hack than a completed program. First, once programmed you can not change the AIS’s MMSI number. I suspect there is a command from Garmin that allows this to be reprogrammed but I haven’t seen it in the wild. Second, I don’t write C. 

If that hasn’t been enough to deter you from proceeding, by all means please continue. Maybe you can even take what I’ve done and expand on it and make it a real program.

A good amount of the information was obtained from this Captain Unlikely here: http://captainunlikely.com/blog/2015/07/10/under-the-hood-of-the-ais-600-transmitter/

SETUP
---------
Setup is simple. Plug a USB cable into the AIS600 and 

READING THE AIS600
--------
Viewing the information from the AIS600 is easy. Especially if you have a newer (2012+ I think) mac that comes with XXX drivers installed. Execute the following command:

`ls -l /dev/tty.*`

you are looking for a device similar to this: `tty.usbserial-A603OKHG`
each AIS600 unit will have it’s own unique name.

Once you find it, execute:

`screen -S ais -L /dev/tty.usbserial-A603OKHG 38400`

Now you can see the NMEA sentences. However, I was never able to communicate with the garmin unit using the default drivers.

FTDI DRIVERS
--------
To write to the unit I needed to install new FTDI drivers. Use the drivers here:
http://www.ftdichip.com/Drivers/D2XX.htm

and follow the old school install instructions.

NMEA SENTENCES
--------
Next we’ll construct the NMEA sentence to program the unit. I’m not sure what’s required, because programming the unit is a one shot deal. So these are the sentences I used:

```"$PGRM,GET,MMSI*44\r\n"
"$AIAIQ,SSD*39\r\n"
"$DUAIQ,VSD*25\r\n"
"$PGRM,GET,MMSI*44\r\n"
"$AIAIQ,VER*3C\r\n"
"$PGRM,DEBUG,ADC,1*02\r\n"
"$PGRM,DEBUG,LED,1*09\r\n"
"$PGRM,SET,GPSCONF,2,1,3*16\r\n"
"$PGRM,SET,MMSI,XXXXXXXX*49\r\n"
"$PGRM,GET,MMSI*44\r\n"
"$AISSD,CCCC,NNNN,AA,BB,CC,DD,0,AI*2C\r\n"
"$AIAIQ,SSD*39\r\n"
"$AIVSD,36,,,,,,,,*60\r\n"
"$DUAIQ,VSD*25\r\n"
"$PGRM,SET,GPSCONF,2,1,1*14\r\n"
```

The sentence `“$AIVSD,36,,,,,,,,*60\r\n"` programs the AIS600 for a sailing vessel. I’m not sure what the other options are.

The two sentences highlighted are the sentences that you will need to update with your own information.

```
XXXXXXXX=MMSI Number
CCCC=Callsign
NNNN=Vessel Name
AA=AIS Unit distance from bow
BB=AIS Unit distance from stern
CC=AIS Unit distance from port
DD=AIS Unit distance from starboard
```

CHECKSUM
--------
For the two newly created sentences you’ll need to calculate the checksum. The Garmin unit will not program without the correct checksum.

You can use the checksum generator found here: http://www.hhhh.org/wiml/proj/nmeaxor.html

Past all text between the $ and the * into the checksum generator. Add the checksum after the * replacing the existing checksum in the example.

UNLOAD EXISTING FTDI DRIVERS
--------
Next you’ll need to unload the existing FTDI drivers. I did this using: sudo kextunload -v -b com.apple.driver.AppleUSBFTDI

COMPILE
--------
git the files from this repository and issue a make command. It will compile a file called write-test.

EXECUTE
--------
Remember, you can screw up your AIS unit if the MMSI number is not correct. You could also just screw up the unit, remember this is a hack.

./write-test

This should program your unit with the MMSI. When I did this the program DID NOT program my vessels name, and call sign. I suspect it was because the program did not pause after issuing the set mmsi command.

To fix this I copied the program: main.name to main.c, recompiled, re-executed.

./write-test

Remember, I don’t write C?
