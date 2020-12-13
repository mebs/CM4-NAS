
# CM4 NAS Solution

This Compute Module 4 daughter board design exposes a subset of the CM4's interfaces, including its single PCIe lane to accept an external SATA controller card.
This design is based off of the official Raspberry Pi Foundation's CM4 IO board, the KiCad project is available on the [IO board official page](https://www.raspberrypi.org/products/compute-module-4-io-board/?resellerType=home)). Removed the unnecessary IO and rearranged the remaining interfaces for a smaller footprint.

The board could be smaller, but it was designed to be as easy to assemble as possible, so almost all the components are through-hole with the exception of the high density CM4 connections, the fan controller (although the pitch is high so it's easy to solder by hand), a buffer, and a few ESD sinks.

Also working on a 3D printed tower style case to complement the board. It will accommodate a 92mm fan, four HDDs, a small OLED screen and of course the board itself with a SATA controller board. The full design will be made available here and on Thingiverse as soon as I have it completed.

[Reddit post here.](https://www.reddit.com/r/raspberry_pi/comments/jt89dm/compute_module_4_nas_pcb_with_pcie/)

![](https://preview.redd.it/12567x1d10z51.jpg?width=3264&format=pjpg&auto=webp&s=f8476466ab5820a5ea29eb13be4bf1354736fcdf)

##### Table of Contents
[Current State](#Current-State)
[Board IO](#board-io)
[Assembly](#assembly)
[Diagrams and Pinouts](#diagrams-and-pinouts)
[Next Steps](#next-steps)

# Current State
I will update this section as I go.
I sadly can't get my hands on a CM4, so nothing was tested yet. I have ordered and received a first batch of 5 PCBs, as well as all the parts I need to assemble a prototype (see bellow for parts list).
I am however still waiting on some tooling to do the very fine pitch soldering. More details on assembly bellow.

The current version of the board was not meant to be used with the lite version of the CM4 as it doesn't forward the microSD leads of the module, but it retains the necessary IO to program the CM4's eMMC.

This project gained some unexpected visibility thanks to Jeff Geerling's YouTube videos, so I'll make sure to update this repo as I find and solve issues, much like he does on [his repo](https://github.com/geerlingguy/raspberry-pi-pcie-devices/issues), which I highly recommend to help with the software part of the problem.
Don't hesitate to ask questions in the Discussion tab of this repo.

## Board IO
### CM4 IO Breakout
The interfaces the board forwards from the CM4 are the following:
- **PCIe x1 Gen2**: Only worthy note is that because I took the footprint for the slot as it was in the IO board project, I inherited a P/N swap in the TX and RX differential pairs. They did that to improve routing on the official board and I could probably have inverted them back but I wanted to keep as much as I could from the official design, as my understanding of the impacts are fairly limited.
- **Ethernet**: To reduce the number of components on the board and make assembly easier, a MagJack is used instead of a simple RJ45 connector. I had to look this up when I saw it in the IO board design, but a MagJack seems to be simply an RJ45 connector with integrated filters to help with signal integrity. Footprint is specific to the part listed bellow.
- **Single slave USB**: A USB type A port to plug a keyboard, a flash drive, a hub, or whatever you want to use as a USB slave to the CM4. Footprint is not standard so make sure it fits yours if you manufacture the PCB.
- **Master USB**: USB connection to use the CM4 as slave. You would use this to program the eMMC with you computer. This connection taps into the same single USB bus the CM4 has to offer, but when this is used, the VCC input from the master signals that the CM4 should behave as a slave. Currently the footprint for this is a simple 5 pin header.
- **HDMI**:  For debugging purposes, I really wanted to have an HDMI output, but I didn't want to solder a fine pitch HDMI female connector, so this is is again just a header (20 pin). The pinout was specifically thought for [this breakout board](https://www.aliexpress.com/item/32904598840.html) I found on AliExpress.
- **I²C**: This is marked "OLED" on the board because this was the intended use, but you can use this to slave any I²C device to the CM4. Four pin JST connector footprint (fits simple headers too).
- **One GPIO**: Marked "Temp_sensor" on the board, but can be used for anything. Hooked to GPIO_4 on the CM4. I would recommend using it for a temp sensor near the SATA controller because they can get really hot and control the fan (see bellow) based on that.
- **Configuration headers**: nRPIBOOT, EEPROM_nWP, AIN1, SYNC_IN, SYNC_OUT, TV_OUT, GLOBAL_EN, RUN_PG, WL_nDis, BT_nDis. The pinout is detailed bellow.

### Other Connections on the Board
Other than relaying the compute module's IOs, the board adds other connections, mainly for power.
- **Fan control**: The board expects a dedicated PWM fan controller IC (see part list). Its output is routed to this JST connector (or your typical PWM fan connector) marked "Fan_PWM" on the board.
- **USB selection headers**: To avoid needing a USB MUX on board, these headers should be used with jumpers to select if you want to use the master or the slave USB.
- **Power input**: The board expects a 5V input for the CM4, and a 12V input for the PCIe card, both through the same JST connector labeled "PWR_IN".
- **3V3 buck**: The PCIe card also requires a dedicated 3V3 supply. This can be provided through the "3V3_BUCK" header near the power input. The pinout was made to accommodate [this daughter board](https://www.aliexpress.com/item/32817933017.html?spm=a2g0s.9042311.0.0.27424c4dr779wi) from AliExpress.
- **HDDs power**: Labeled "SATA_PWR". This is the power rail for the hard drives.

## Assembly
### Required parts
I'll do a more detailed breakdown of all the parts as I build this document (resistor values etc.), but it's worth mentioning the main parts. Links to DigiKey Canada are provided.
- **[High density connectors](https://www.digikey.ca/en/products/detail/hirose-electric-co-ltd/DF40C-100DS-0.4V%2851%29/1969495)**: You will need these to connect the CM4 to the board. They can be very daunting, but I found [this video](https://www.youtube.com/watch?v=eukcrFc18P4) of someone neatly soldering them with a hot air gun. Never did it myself, can't wait to try when I get the hot air gun.
- **[MagJack](https://www.digikey.ca/en/products/detail/bel-fuse-inc/0826-1G1T-43-F/2107992)**: I made the footprint for this specific one.
- **[Fan controller](https://www.digikey.ca/en/products/detail/microchip-technology/EMC2301-1-ACZL-TR/4696431)**: This chip is the exact same one that is on the official IO board and it's controlled by the CM4 through I²C.
- **[ESD sinks](https://www.digikey.ca/en/products/detail/texas-instruments/TPD4EUSB30DQAR/2503671)**: They are optional in my (very uninformed) opinion. I think they were included on the IO board for POE applications mostly.
- **[Signal buffer](https://www.digikey.ca/en/products/detail/diodes-incorporated/74LVC1G07SE-7/2356550)**: Used to buffer RUN_PG signal to be able to wake the CM4 from a sleep state. Not needed if you're not planning on doing that.

### Manufacturing
*Coming soon*


## Diagrams and Pinouts
*Coming soon*


# Next Steps
If the current design works, it would already make for a usable product. The board is fairly simple but until I test it with a CM4, a PCIe card and hard drives, with some real power draw I won't know for sure.
I will open issues and solve them as I go, but even if there are none, there would be a few enhancements the project could benefit from.
I opened a couple discussions in the [ideas section](https://github.com/mebs/CM4-NAS/discussions?discussions_q=category:Ideas) to discuss enhancements and alternative designs.
