__I take no responsibility for you blowing your car up, bricking your ECU, or just being bad at educating yourself.__

# Hardware
* Tactrix OpenPort
* Windows Latpop with at least two USB ports.
* Wideband with serial out.
* Serial to USB adapter. Pay attention to pin out on adapter, sometimes  they are what are known as "null modem" and one of the pins are switched.
* OBD2 Extension cable - you will break your openport without one.

# Install

[Romraider](http://www.romraider.com/RomRaider/Download)

[ECUFlash](https://www.tactrix.com/index.php?option=com_content&view=category&layout=blog&id=36)

[32bit Java Runtime Environment](https://java.com/en/download/manual.jsp)

[Tactrix OpenPort drivers](https://www.tactrix.com/index.php?option=com_content&view=category&layout=blog&id=38&Itemid=61)

[Putty](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)

# Configuration

Download the files below.


[`logger.xml`](http://www.romraider.com/forum/topic1642.html) - The correct file is at last post on page. It might be worth googling around and finding the highest version number you can. At the time of writing this it is v346.


[`cars_def.dtd` and `cars_def.xml`](http://www.romraider.com/forum/viewtopic.php?t=5792) - Save outside of Romraider install folder.

[`ecu_defs.xml`](https://github.com/TD-D/SubaruDefs/blob/Alpha/RomRaider/ecu/) - This seems to be where these files are maintained now. This file needs to be saved in RomRaider install directory.

# Setup

Open RomRaider logger, it will give a lot of errors. Click "Settings > Logger Definition File" and provide the path to `logger.xml`.

Restart RomRaider Logger. It should not error anymore if configured correctly. Close it.

Now open up RomRaider ECU Editor and get through the errors. Click "ECU Definitions > ECU Definition Manager". Click "Add" on the next prompt, and select the `ecu_defs.xml` file stored in the directory that is opened The file MUST be saved here. Click apply and save. Restart RomRaider ECU Editor, it should not error. Close it.

Open ECU Flash. Click "ECU > Select Vehicle Type". You can add more types by finding the appropriate files in the SubaruDefs repo linked above. The standard `ecu_defs.xml` applies to most Subaru's.

Go out to your car. Connect green test mode connectors. Connect tactrix cable. Open ECUFlash and click the "Blue arrow going up from the microchip button" to read the ECU  - Follow the insructions. Your car will make weird noises as the test mode circut is connected.

When done reading, save the file as a `.bin` file. 

You should be able to view the map in RomRaider ECU Editor now.


# Logging Wideband via Serial
Connect and find the appropriate wires and adapter to convert your wideband out to a serial input via USB. Install drivers if necessary.

Hook up USB and with laptop hooked up and key in ignition open putty and open a serial connection to `COMX`. This will verify that you can communicate with your computer, RR logger is kind of buggy.



Enjoy.
