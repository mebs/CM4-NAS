# CM4 NAS Solution

This Compute Module 4 carrier board design exposes a subset of the CM4's interfaces, including its single PCIe 2.0 lane to accept an external SATA controller card.
This design is based off of the official Raspberry Pi Foundation's CM4 IO board (the KiCad project is available on the [IO board official page](https://www.raspberrypi.org/products/compute-module-4-io-board/?resellerType=home)). Removed the IO that was less relevant for a NAS and rearranged the remaining interfaces for a smaller footprint that would fit within the width of a standard 3.5" hard drive.

The board was intentionally kept simple to limit mistakes as this is my first ever attempt at designing a PCB and I have no background in electronics, so all the power management is left to external power supplies and buck converters. It's important to use good quality regulated power supplies with this board as it offers no protection other than what is built into the CM4.

Also made 3D printed enclosure for the final product. Design available here: https://www.thingiverse.com/thing:4820662

Feel free to explore the discussion section of this repo to share ideas.

[Latest Reddit post here.](https://www.reddit.com/r/raspberry_pi/comments/mkbxss/cm4_custom_nas_complete/)

![](https://user-images.githubusercontent.com/2614134/108195493-d519ed80-70e5-11eb-8293-3591e432dd8e.jpg)

##### Table of Contents
- [Software](#software)
- [Board IO](#board-io)
  * [CM4 IO Breakout](#cm4-io-breakout)
  * [Other Connections on the Board](#other-connections-on-the-board)
- [PCB Assembly](#pcb-assembly)
  * [Manufacturing](#manufacturing)
  * [Required Parts](#required-parts)
  * [Building](#building)
- [Pinout](#pinout)
- [Revision History](#revision-history)

# Software
For the software part of this project, I highly recommend [Jeff Geerling's repo](https://github.com/geerlingguy/raspberry-pi-pcie-devices/issues), where he documented similar endeavors, but you will likely have to recompile the kernel after enabling some modules to be able to use a PCIe SATA controller.

Don't hesitate to ask questions in the Discussion tab of this repo.

# Board IO
## CM4 IO Breakout
The interfaces the board forwards from the CM4 are the following:
- **PCIe x1 Gen2**: Good old PCIe 2.0, nothing special about it... if not that it's on a Raspberry Pi.
- **Ethernet**: To reduce the number of components on the board and make assembly easier, a MagJack is used instead of a simple RJ45 connector. I had to look this up when I saw it in the IO board design, but a MagJack seems to be simply an RJ45 connector with integrated filters to help with signal integrity. Footprint is specific to the part listed bellow.
- **Single master/slave USB, selected with jumper**: A USB type A port to plug a keyboard, a flash drive, a hub, or whatever you want to use as a USB slave to the CM4. Footprint is not standard so make sure it fits yours if you manufacture the PCB. Can also be used in slave mode (CM4 as slave). You would use this to program the eMMC with you computer. This connection taps into the same single USB bus the CM4 has to offer, but when this is used, the VCC input from the master signals that the CM4 should behave as a slave. Currently the footprint for this is a simple 5 pin header.
- **Full size HDMI**:  Mostly for debugging purposes, hooked to the HDMI0 output of the CM4.
- **I²C**: This is marked "OLED" on the board because this was the intended use, but you can use this to slave any I²C device to the CM4. Five-pin JST connector footprint (fits simple headers too).
- **Single wire temperature sensor**: Marked "Temp_sensor" on the board, hooked to GPIO_4 on the CM4. I would recommend using it for a temp sensor near the SATA controller because they can get really hot and control the fan (see bellow) based on that.
- **9 GPIOs**: General purpose IO header, including Gnd, 3V3 and 5V pins.
- **Configuration headers**: nRPIBOOT, EEPROM_nWP, AIN1, SYNC_IN, SYNC_OUT, TV_OUT, GLOBAL_EN, RUN_PG, WL_nDis, BT_nDis. The pinout is detailed on the board.

## Other Connections on the Board
Other than relaying the compute module's IOs, the board adds other connections, mainly for power.
- **Fan control**: The board expects a dedicated PWM fan controller IC (see part list). Its output is routed to this JST connector (or your typical PWM fan connector) marked "Fan_PWM" on the board.
- **USB selection headers**: To avoid needing a USB MUX on board, this header should be used with a jumper to select if you want to use the USB bus as master or as slave.
- **Power input**: The board expects a 5V input for the CM4, and a 12V input for the PCIe card, both through the same JST connector labeled "PWR_IN".
- **3V3 buck**: The PCIe card also requires a dedicated 3V3 supply. This can be provided through the "3V3_BUCK" header near the power input. The pinout was made to accommodate [this daughter board](https://www.aliexpress.com/item/32817933017.html?spm=a2g0s.9042311.0.0.27424c4dr779wi) from AliExpress, but you can use whatever you want for as long as you follow the pinout bellow.
- **HDDs power**: Labeled "SATA_PWR". This is the power rail for the hard drives.

# PCB Assembly
## Manufacturing
There is a *Fab* folder at the repo's root. This folder contains all the files that you need to order PCBs from a manufacturer. You will find in the folder a Fab_rev1.4.zip archive that is all you need to order from JLC PCB. This is the latest version and the one I have been running for months now.

## Required Parts
Other than the standard caps and resistors, here are the components you will need if you want to assemble the board:
- **High density connectors** ([digikey](https://www.digikey.ca/en/products/detail/hirose-electric-co-ltd/DF40C-100DS-0.4V%2851%29/1969495), [LCSC](https://lcsc.com/product-detail/Mezzanine-Connectors-Board-to-Board_HRS-Hirose-DF40C-100DS-0-4V-51_C597931.html)): You will need these to connect the CM4 to the board. They can be very daunting, but I found [this video](https://www.youtube.com/watch?v=eukcrFc18P4) of someone neatly soldering them with a hot air gun. When I finaly got to try it, I was surprise at how doable it was.
- **MagJack** ([digikey](https://www.digikey.ca/en/products/detail/bel-fuse-inc/0826-1G1T-43-F/2107992)): I made the footprint for this specific one. It's expensive and I should have looked on LCSC before ordering.
- **Fan controller** ([digikey](https://www.digikey.ca/en/products/detail/microchip-technology/EMC2301-1-ACZL-TR/4696431), [LCSC](https://lcsc.com/product-detail/_MICROCHIP_EMC2301-1-ACZL-TR_EMC2301-1-ACZL-TR_C148036.html)): This chip is the exact same one that is on the official IO board and it's controlled by the CM4 through I²C.
- **ESD sinks** ([digikey](https://www.digikey.ca/en/products/detail/texas-instruments/TPD4EUSB30DQAR/2503671), [LCSC](https://lcsc.com/product-detail/Diodes-ESD_Texas-Instruments-TPD4EUSB30DQAR_C90627.html)): They are optional in my (very uninformed) opinion. I think they were included on the IO board for POE applications mostly.
- **Signal buffer** ([digikey](https://www.digikey.ca/en/products/detail/diodes-incorporated/74LVC1G07SE-7/2356550), [LCSC](https://lcsc.com/product-detail/Logic-Buffers-Drivers-Receivers-Transceivers_Diodes-Incorporated-74LVC1G07SE-7_C67531.html)): Used for the power LED and to buffer RUN_PG signal to be able to wake the CM4 from a sleep state.
- **HDMI connector** ([LCSC](https://lcsc.com/product-detail/Audio-Video-Connectors_SOFNG-HDMI-019S_C111617.html)): Basic full size HDMI connector.

## Building
To build the board, you will need a soldering iron and a hot air gun (or a reflow oven). I would recommend starting with the Hirose high density connectors as they are without a doubt the hardest part. [This video](https://www.youtube.com/watch?v=eukcrFc18P4) I referred to earlier is a good starting point, it makes for a fine tutorial.
It only took a couple tries before I could get a decent result (image from microscope at around 50-60x):

![](https://user-images.githubusercontent.com/2614134/103139543-5ab38b00-46ab-11eb-81e4-08bcccfd27b5.jpg)


Once the HD connectors are in place, you can move on to the ESD sinks. They are very small and can also be daunting, but soldering them turned out to be easier than expected.
The trick is to put as little solder paste as possible and the chips will naturally be pulled in place by the solder's surface tension.

![](https://user-images.githubusercontent.com/2614134/103139571-94849180-46ab-11eb-9140-62274b9e4ab9.jpg)


The rest of the parts should be fairly easy to solder with a standard iron.

# Pinout
![](https://user-images.githubusercontent.com/2614134/115098729-9cd53680-9eff-11eb-8c69-481f20cef7db.png)


# Revision History
- **1**: Initial version
- **1.1**: Fixed wrong footprint for fan controller
- **1.2**: Changed the USB select headers for a simpler solution with a single jumper
- **1.3**: Replaced the HDMI header by a full size HDMI female connector, exposed 9 GPIOs, inverted polarity back for PCIe Tx/Rx signals and added a power LED.
- **1.4**: Add pin to OLED connector for button, use SMD caps and LEDs, more info on silkscreen and fixed some traces.
