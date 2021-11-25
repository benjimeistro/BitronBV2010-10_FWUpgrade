# BitronBV2010-10_FWUpgrade
Bitron BV2010/10 ZigBee USB Stick FW Upgrade Procedure. 

***I DO NOT HOLD ANY RESPONSIBILTY FOR METHODS PROPOSED BELOW THIS WAS JUST MY WORKING METHOD TO UPGRADE THE FW AND NOTES TO HELP ME REMEMBER THE PROCESS***

Background:
The Bitron BV2010 Funkstick I bought from Amazon was loaded with FW version 5.8.0.

Which worked on my OpenHAB system but not very well. :(

I had kept an eye on FW upgrade methods for the Bitron BV2010/10, but nothing concrete had arisen. 

There was this link here:

[https://github.com/zigpy/bellows/issues/348]

Which explains the bootloader process and the chipset flashing methods, but the outcome wasn't specific and looked to be a lot of development 
talk and experimentation for the bellows tool with no concrete FW or method. 

After Googling the chipset and stick I narrowed it down to one user called NilsOF who had a recent post on the OpenHAB forum,
 
[https://community.openhab.org/t/firmware-upgrade-the-bitronvideo-bv-2010-10-zigbee-usb-dongle/128879]

NilsOF had managed to flash a later version FW to his BV2010!! Using a FW file here below:

[https://github.com/grobasoz/zigbee-firmware/raw/af7c35ea8d580152eb9853af1d3fab91bef3b5d4/EM3587/NCP_USW_EM3587-LR_678-115k2.ebl]

This peaked my interest. The process took some playing with to nail down but this is how I upgraded my BV2010/10. :)

Disclaimer

***I do not hold any guarantees that this will work for you its just the documented process I took to upgrade my FW on my Bitron Funkstik BV2010/10.***  
  ***Anything you decide to follow in the document, you do at your own risk. With your own will, with your own consequences. I AM NOT RESPONSIBLE ***

METHOD:
I downloaded Elelabs flashing script for EMBER chipsets from here:

[https://github.com/Elelabs/elelabs-zigbee-ezsp-utility]

Unzipped the directory and cd'd to it, then installed requirements as per instructions on the GitHUB page.

I then ran the script using this command. After granting execute permissions on the script. 

sudo python3 ./Elelabs_EzspFwUtility.py  probe -p /dev/ttyUSB0 -b 57600 -d RAW

A quick breakdown, of the above command. probe [probe device] -p [port] -b [baudrate] -d [debugMode]  

After running the above command it threw an error on the script below. 
I just commented this variable out in the script. I DO NOT recommend you do this, this is just what i did.
 
(It didn't seem to make much of an impact, at the very least it now allowed me to probe and restart the BV2010.)

File "./Elelabs_EzspFwUtility.py", line 549, in probe
    self.logger.info("Firmware: %s" % firmware_version)
UnboundLocalError: local variable 'firmware_version' referenced before assignment

I could now probe and restart the BV2010/10

./Elelabs_EzspFwUtility.py probe -p /dev/ttyUSB0  -b 57600 -d DEBUG
Elelabs_EzspFwUtility:   RESET FRAME
Elelabs_EzspFwUtility:   EZSP adapter in bootloader mode detected:
Elelabs_EzspFwUtility:   EM3587 Serial Btl v5.8.0.0 b134

./Elelabs_EzspFwUtility.py restart -m btl -p /dev/ttyUSB0  -b 57600 -d RAW

Elelabs_EzspFwUtility:   RESET FRAME
Elelabs_EzspFwUtility:   EZSP adapter in bootloader mode detected:
Elelabs_EzspFwUtility:   EM3587 Serial Btl v5.8.0.0 b134
Elelabs_EzspFwUtility:   Allready in bootloader mode. No need to restart
 
Ah! As seen above I can now see the FW version running on the stick and I now know that because of the information
supplied and groundwork on the chipset, the FW supplied by NilsOF in the GitHub link above can be applied to my device! 

We are on 5.8.0..

Right time to get flashing. I immediately tried to flash it using the script above, at first it almost succeeded, or
so I thought.

./Elelabs_EzspFwUtility.py flash -f /home/b/bitron_fw/NCP_USW_EM3587-LR_678-115k2.ebl -p /dev/ttyUSB0 -b 57600 -d DEBUG 
Elelabs_EzspFwUtility:   RESET FRAME
Elelabs_EzspFwUtility:   EZSP adapter in bootloader mode detected:
Elelabs_EzspFwUtility:   EM3587 Serial Btl v5.8.0.0 b134
Elelabs_EzspFwUtility:   Allready in bootloader mode. No need to restart
Elelabs_EzspFwUtility:   Successfully restarted into X-MODEM mode! Starting upload of the new firmware... DO NOT INTERRUPT(!)
Elelabs_EzspFwUtility:   Failed to restart into bootloader mode. Please see users guide.

:(

I scoured the internet again for a solution. There was one ambigious message on the OpenHAB topic above.  

"I used the OpenHAB console zigbee firmware command to flash."

I went straight into my OpenHAB console, I wanted to find this mysterious zigbee firmware flash command. I grepped out the
zigbee command and the only thing found was an otaupgrade command.

zigbee firmware 
Error: Could not find command: firmware

After re-reading the above OpenHAB forum post a few times, it was apparent that the user was using a later version of the ZigBee
binding for OpenHAB than myself. 

I then upgraded the OpenHAB packages to the latest unstable version and refreshed the zigbee bundles in the OpenHAB console. 

After listing the bundles I could see the zigbee bundle had been upgraded to the latest version. 

I jumped to the ZigBee command in OpenHAB console and again grepped the firmware keyword.

Usage: openhab:zigbee - firmware [VERSION | CANCEL | FILE] - Updates the dongle firmware

Excellent, time to move forward. 

I then tried to flash the firmware directly from the console . 

openhab:zigbee firmware /bitron_fw/NCP_USW_EM3587-LR_678-115k2.ebl
Error: Could not find bridge handler for bridge

It's not looking good. Although, I knew it was probably stuck in bootloader mode after the flash. 

I also knew my system could also do with a refresh after installing the new bindings etc. 

rebooted.

After reboot then went straight back into the OpenHAB console:

openhab> zigbee firmware /home/b/bitron_fw/NCP_USW_EM3587-LR_678-115k2.ebl
Dongle firmware status: FIRMWARE_UPDATE_STARTED.
Starting dongle firmware update...

I didn't get any response from the console, it will show this message and then that is all. It just did this and then all was quiet. 

At this point I'm thinking it's either not flashed, it has flashed, it's currently flashing, might even be a brick.. xD

I should have went and watched the LED on the stick, this probably would have given me some sort of instruction as to what was happening. 

I left it alone for around ten minutes, I know better than to panic at this point with FW and I didn't want to interrupt the process of flashing.

I also knew I could ground a pin of the device if it bricked to get it back into bootloader mode, so all would not be lost. 

I opened another console, and started to probe the device using the Elelabs script. 

./Elelabs_EzspFwUtility.py probe -p /dev/ttyUSB0 
Elelabs_EzspFwUtility:   Generic Zigbee EZSP adapter detected:
Elelabs_EzspFwUtility:   EZSP v8

Yes.. new firmware and dongle detected, now I'm going to try it in RAW debug mode..  

./Elelabs_EzspFwUtility.py probe -p /dev/ttyUSB0 -d RAW -b 57600
Elelabs_EzspFwUtility:   RESET FRAME
Elelabs_EzspFwUtility:   Couldn't communicate with the adapter in Zigbee (EZSP) mode, Thread (Spinel) mode or bootloader mode

Notice the second time I set the buad rate to the previous setting. I realised pretty quickly, this failed because the baudrate 
had changed from 57600 to 115200 as when you run the script it defaults to 115200. 

I then retested using the 115200 baud rate. Success.

I then tried to restart the device in normal mode as I now knew the FW upgrade was successful. 

./Elelabs_EzspFwUtility.py restart -m nrml  -p /dev/ttyUSB0  -b 115200 -d RAW
Elelabs_EzspFwUtility:   EZSP v8 detected
Elelabs_EzspFwUtility:   sendVersion: V8
Elelabs_EzspFwUtility:   getValue: EZSP_VALUE_VERSION_INFO
Elelabs_EzspFwUtility:   getMfgToken: EZSP_MFG_STRING
Elelabs_EzspFwUtility:   Generic Zigbee EZSP adapter detected:
Elelabs_EzspFwUtility:   EZSP v8
Elelabs_EzspFwUtility:   Allready in normal mode. No need to restart

Now I am happy the FW flashed successfully and the device is in Normal mode, I unplugged and replugged the dongle.

Now OpenHAB zigbee binding could see the device but couldn't communicate the device, it added it to my Inbox, but couldn't init. 

This is because it is auto added to the Inbox with a default value of 57600 baudrate with Software flow control mentioned by NilsOF also. 

The new firmware operates @ 115200 Hardware flow control. 

I then created a zigbee.thing object to manually apply static settings to the new device and ignored the one generated in the Inbox.

I then set the zigbee logs on OpenHAB to DEBUG and watched the binding init the device. 

After around 5 minutes, eventually it came online and has been functioning as expected and seems to work much better than the previous FW. 

Many Thanks to Elelabs, Chris Jackson and NilsOF.
 
