# Bitron BV AV2010/10 Firmware Upgrade Method.
Bitron BV AV2010/10 ZigBee USB Stick FW Upgrade Procedure. 

Disclaimer: ***I DO NOT HOLD ANY RESPONSIBILTY FOR ANY OF THE METHODS PROPOSED BELOW, THIS WAS JUST MY WORKING METHOD TO UPGRADE THE FW AND NOTES TO HELP ME REMEMBER THE PROCESS***

Firstly: Many Thanks to Elelabs, Chris Jackson and NilsOF.

Background:
The Bitron BV AV2010/10 Funkstick I bought from Amazon was loaded with FW version 5.8.0, which worked on my OpenHAB system but not very well. :( I had kept an eye on FW upgrade methods for the Bitron BV AV2010/10, but nothing concrete had arisen. 

There was this link here: https://github.com/zigpy/bellows/issues/348

The link above is interesting and explains the bootloader process and the chipset flashing methods, but the outcome wasn't too specific and looked to be a lot of development talk and experimentation for the bellows tool with no concrete FW or method, that I could see. 

After Googling the chipset and stick I narrowed it down to one user called NilsOF who had a recent post on the OpenHAB forum:
 
https://community.openhab.org/t/firmware-upgrade-the-bitronvideo-bv-2010-10-zigbee-usb-dongle/128879

In the topic above NilsOF mentioned that they managed to flash a later version of firmware to the BV AV2010. 

https://github.com/grobasoz/zigbee-firmware/raw/af7c35ea8d580152eb9853af1d3fab91bef3b5d4/EM3587/NCP_USW_EM3587-LR_678-115k2.ebl

This piqued my interest. The process took some playing with to nail down but this is how I upgraded my BV AV2010/10. :)

NilsOF provided the method of flashing via the openHAB console in the OpenHAB forum link above also. 

Disclaimer:

***I do not hold any guarantees that this will work for you, its just the documented process I took to upgrade my firmware on my Bitron Funkstik BV AV2010/10.***  

***Anything you decide to follow in this document, you do at your own risk. At your own consequence. THIS IS MY PUBLIC PERSONAL GUIDE FOR PERSONAL USE, I AM NOT RESPONSIBLE FOR ANY DAMAGE INCURRED IF YOU FOLLOW ANY OF THE BELOW INSTRUCTIONS.***

METHOD:

I downloaded Elelabs flashing script for EMBER chipsets to check the firmware versions from here:

https://github.com/Elelabs/elelabs-zigbee-ezsp-utility

I unzipped the directory and cd'd to it, then installed requirements as per instructions on the Elelabs Github page.

I then ran the script using this command after granting execute permissions on the script: 
```
sudo python3 ./Elelabs_EzspFwUtility.py  probe -p /dev/ttyUSB0 -b 57600 -d RAW
```
A quick breakdown of the above command is. probe [probe device] -p [port] -b [baudrate] -d [debugMode]  

After running the above command it threw an error, I commented this variable that errored in the script. 

I DO NOT recommend you do this, this is just what i did.

I could now probe and restart the BV AV2010/10 in bootloader mode and back into normal mode.
```
./Elelabs_EzspFwUtility.py probe -p /dev/ttyUSB0  -b 57600 -d DEBUG
Elelabs_EzspFwUtility:   RESET FRAME
Elelabs_EzspFwUtility:   EZSP adapter in bootloader mode detected:
Elelabs_EzspFwUtility:   EM3587 Serial Btl v5.8.0.0 b134

./Elelabs_EzspFwUtility.py restart -m btl -p /dev/ttyUSB0  -b 57600 -d RAW

Elelabs_EzspFwUtility:   RESET FRAME
Elelabs_EzspFwUtility:   EZSP adapter in bootloader mode detected:
Elelabs_EzspFwUtility:   EM3587 Serial Btl v5.8.0.0 b134
Elelabs_EzspFwUtility:   Allready in bootloader mode. No need to restart
``` 
As seen above I can now see the firmware version running on the stick I have and I now know that because of the information
supplied by NilsOF and research on the chipset, that the firmware linked by NilsOF in the GitHub link can be applied to my device! :) 

My BV AV2010/10 is on firmware 5.8.0 from the output above.

Time to get flashing. I immediately tried to flash it using the elelabs script above, at first it almost succeeded, or
so I thought, the x-modem baudrate seems to be hard coded in the python script..
```
./Elelabs_EzspFwUtility.py flash -f /bitron_fw/NCP_USW_EM3587-LR_678-115k2.ebl -p /dev/ttyUSB0 -b 57600 -d DEBUG 
Elelabs_EzspFwUtility:   RESET FRAME
Elelabs_EzspFwUtility:   EZSP adapter in bootloader mode detected:
Elelabs_EzspFwUtility:   EM3587 Serial Btl v5.8.0.0 b134
Elelabs_EzspFwUtility:   Allready in bootloader mode. No need to restart
Elelabs_EzspFwUtility:   Successfully restarted into X-MODEM mode! Starting upload of the new firmware... DO NOT INTERRUPT(!)
Elelabs_EzspFwUtility:   Failed to restart into bootloader mode. Please see users guide.
```
:(

I scoured the internet again for a solution there wasn't much. I went back over the OpenHAB topic posts. There was one ambigious message on the same OpenHAB topic above from NilsOF.  

"I used the OpenHAB console zigbee firmware command to flash." .Thank You.

I went straight into my OpenHAB console, I wanted to find the mysterious zigbee firmware flash command. I had looked previously and was sure it didnt exist, I grepped out the zigbee command and the only thing found was an otaupgrade command.
```
zigbee firmware 
Error: Could not find command: firmware
```
After re-reading the above OpenHAB forum post a few times again, it was apparent to me that NilsOF was using a later version of the ZigBee
binding for OpenHAB than myself. 

I then upgraded the OpenHAB packages to the latest unstable version and refreshed the zigbee bundles in the OpenHAB console. 

After listing the bundles I could see the zigbee bundle had been upgraded to the latest version. 

I jumped to the ZigBee command in OpenHAB console and again grepped the firmware keyword.
```
Usage: openhab:zigbee - firmware [VERSION | CANCEL | FILE] - Updates the dongle firmware
````
Excellent, time to move forward. 

I then tried to flash the firmware directly from the console . 
```
openhab:zigbee firmware /bitron_fw/NCP_USW_EM3587-LR_678-115k2.ebl
Error: Could not find bridge handler for bridge
```
It's not looking good. Although, I knew it was probably stuck in bootloader mode after the flash attempt with the Elelabs script. 

I also knew my system could also do with a refresh after installing the new bindings etc. 

rebooted.

After reboot then went straight back into the OpenHAB console and ran the zigbee firmware command again:
```
openhab> zigbee firmware /bitron_fw/NCP_USW_EM3587-LR_678-115k2.ebl
Dongle firmware status: FIRMWARE_UPDATE_STARTED.
Starting dongle firmware update...
```
I didn't get any response from the console, it will show this message and then that is all. It just did this and then all was quiet. At this point I'm thinking it's either not flashed, it has flashed, it's currently flashing, might even be a brick.. xD In hindsight, I should have went and watched the LED on the stick, this probably would have given me some sort of instruction as to what was happening. :)

I left it alone for around ten minutes, I know better than to panic at this point with FW and pull the usb or reboot. I didn't want to interrupt the process of flashing. I also knew I could ground a pin of the device if it bricked to get it back into bootloader mode, so all would not be lost. 

I opened another console, and started to probe the device using the Elelabs script. 
```
./Elelabs_EzspFwUtility.py probe -p /dev/ttyUSB0 
Elelabs_EzspFwUtility:   Generic Zigbee EZSP adapter detected:
Elelabs_EzspFwUtility:   EZSP v8
```
Yes.. new firmware and dongle detected, now I'm going to try it in RAW debug mode..  
```
./Elelabs_EzspFwUtility.py probe -p /dev/ttyUSB0 -d RAW -b 57600
Elelabs_EzspFwUtility:   RESET FRAME
Elelabs_EzspFwUtility:   Couldn't communicate with the adapter in Zigbee (EZSP) mode, Thread (Spinel) mode or bootloader mode
```
Notice that the second time I set the baud rate to the previous setting. I realised pretty quickly (the firmware has 1152 on the end). This failed because the baudrate has changed from 57600 to 115200 after the upgrade, as when you run the script without the -b setting it defaults to 115200. 

I then retested using the 115200 baud rate as per the FW file. Success.

I then tried to restart the device in normal mode, as I now knew the FW upgrade was successful. 
```
./Elelabs_EzspFwUtility.py restart -m nrml  -p /dev/ttyUSB0  -b 115200 -d RAW
Elelabs_EzspFwUtility:   EZSP v8 detected
Elelabs_EzspFwUtility:   sendVersion: V8
Elelabs_EzspFwUtility:   getValue: EZSP_VALUE_VERSION_INFO
Elelabs_EzspFwUtility:   getMfgToken: EZSP_MFG_STRING
Elelabs_EzspFwUtility:   Generic Zigbee EZSP adapter detected:
Elelabs_EzspFwUtility:   EZSP v8
Elelabs_EzspFwUtility:   Allready in normal mode. No need to restart
```
Now I am happy the FW flashed successfully and the device is in Normal mode, I unplugged and replugged the dongle.

Now OpenHAB zigbee binding could see the device but couldn't communicate the device, it added it to my Inbox, but couldn't initialize. 

This is because it is auto added to the Inbox with a default value of 57600 baudrate with Software flow control mentioned by NilsOF also. 

The new firmware operates @ 115200 Hardware flow control. 

I then created a zigbee.thing object manually on OpenHAB to apply static settings to the new device and then ignored the one generated in the Inbox.

I then set the zigbee binding logs on OpenHAB console to DEBUG and watched the binding start to initialize the device successfully. 

After around 5 minutes, it came online and has been functioning as expected and seems to work much better than the previous FW. 

Many Thanks again to Elelabs, Chris Jackson and NilsOF.
