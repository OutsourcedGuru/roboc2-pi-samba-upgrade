# roboc2-pi-samba-upgrade
Step-by-step instructions for updating Samba configuration on your Robo C2 printer.

The intent is to update the existing Samba software on the Raspberry Pi 3 computer so that it better advertises its NETBIOS name on Windows-based networks and optionally, to open up a Windows share on at least the microSD card in the folder where it stores your GCODE files.  This should make it easier to then push files to your printer without shuttling a USB drive back and forth.

Note that this same method may be used to share the USB drive itself while it's in the printer.  I will be showing the method for doing both.

Note as well that OS X computers may also map these Windows-based share points so it works for those workstations, too.

## Parts
No parts are actually required for this upgrade.  And no 3D printed parts are required either.

## Software
I assume that you already have either `ssh` (OS X) or `putty` (Windows) so that you can connect remotely to the Raspberry Pi 3 that's inside your Robo C2 printer.

## Workstation
I assume that you have either a Windows- or an OX X--based workstation which you use to run a slicing program like Cura.  In my case, I'm using a MacBook.  Before this update, my existing workflow might look like this:

1. Create a project on my Mac Mini using Autodesk Fusion 360, exporting an STL file to its Desktop
2. On the MacBook, copy the file from the Mac Mini
3. On the MacBook, open Cura and slice the file to a USB drive
4. Eject the drive on the MacBook, move it to the printer
5. On the MacBook, open a browser session to OctoPrint running on the printer and select the GCODE file from the USB to print it

Clearly, this is a hassle in that I have to shuttle that USB drive.  Here is the expected new behavior:

1. Create a project on my Mac Mini using Autodesk, exporting the STL file to the USB drive in the printer
2. On the MacBook, open the STL file from the USB drive in the printer, slicing it to a GCODE file again on this same drive
3. On the MacBook, open a browser session to OctoPrint running on the printer and select the GCODE file from the USB drive

Similarly, I might instead create files on the microSD card directly using this same technique.

## Possible Complications
Although one never ejects the microSD card from the printer while it's still running, it's not unheard of to want to eject the USB drive while it's on.  A `net share` must be managed properly to make sure that data isn't lost.

For this reason, if you intend to share the USB drive itself, I would suggest that you simply leave it in the printer at all times.  The Samba configuration file will expect the USB drive to be mounted so this will make things simpler.

```
sudo apt-get install samba samba-common-bin
```

```
sudo mkdir -m 1777 /share
```

```
sudo smbpasswd -a pi
```

```
sudo nano /etc/samba/smb.conf
```

```
[global]
workgroup=WORKGROUP
encrypt passwords=yes
wins support=yes
# Set NETBIOS name below to match its DNS hostname in /etc/hostname
# netbios name=%h
# Set the description to Samba v[Version] on [NetBIOS name]
server string=Samba %v on %L

[microsd]
Comment=Pi share to the microSD share folder
Path=/mnt/usbdisk1/share
Browseable=yes
Writeable=yes
Read only=no
only guest=no
create mask=0777
directory mask=0777
Public=yes
Guest ok=yes
```

```
sudo testpart /etc/samba/smb.conf
sudo service smbd restart
```
