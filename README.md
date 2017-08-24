# roboc2-pi-samba-upgrade
Step-by-step instructions for updating Samba configuration on your Robo C2 printer.

The intent is to update the existing Samba software on the Raspberry Pi 3 computer so that it better advertises its NETBIOS name on Windows-based networks and optionally, to open up a Windows share on at least the microSD card in the folder where it stores your GCODE files.  This should make it easier to then push files to your printer without shuttling a USB drive back and forth.

Note that this same method may be used to share the USB drive itself while it's in the printer.  I will be showing the method for doing both.

## Parts
No parts are actually required for this upgrade.  And no 3D printed parts are required either.

## Software
I assume that you already have either `SSH` (OS X) or `putty` (Windows) so that you can connect remotely to the Raspberry Pi 3 that's inside your Robo C2 printer.
