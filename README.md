# roboc2-pi-samba-upgrade
Step-by-step instructions for updating Samba configuration on your Robo C2 printer.

The intent is to update the existing Samba software on the Raspberry Pi 3 computer so that it better advertises its NETBIOS name on Windows-based networks and optionally, to open up a Windows share on at least the microSD card in the folder where it stores your GCODE files.  This should make it easier to then push files to your printer without shuttling a USB drive back and forth.

Note that this same method could be used to also share the USB drive itself while it's in the printer.  I might show the method for doing this but wouldn't necessarily recommend doing so.

Note as well that OS X computers may also map these Windows-based share points so it works for those workstations, too.

## Parts
No parts are actually required for this upgrade.  And no 3D printed parts are required either.

## Software
I assume that you already have either `ssh` (OS X) or `putty` (Windows) so that you can connect remotely to the Raspberry Pi 3 that's inside your Robo C2 printer.

## Workstation
I also assume that you have either a Windows- or an OX X—based workstation which you use to run a slicing program like Cura.  In my case, I'm using a MacBook.  Before this update, my existing workflow might look like this:

1. Create a project on my Mac Mini using Autodesk Fusion 360, exporting an STL file to its Desktop
2. On the MacBook, copy the file from the Mac Mini
3. On the MacBook, open Cura and slice the file to a USB drive that's in the MacBook itself
4. Eject the drive on the MacBook, move it to the printer
5. On the MacBook, open a browser session to OctoPrint running on the printer and select the GCODE file from the USB to print it

Clearly, this is a hassle in that I have to shuttle that USB drive and move files around.  Here is the expected new behavior after the Samba upgrade:

1. Create a project on my Mac Mini using Autodesk, exporting the STL file to the microSD drive in the printer
2. On the MacBook, open the STL file from the microSD drive in the printer, slicing it to a GCODE file again directly onto this same drive
3. On the MacBook, open a browser session to OctoPrint running on the printer and select the GCODE file from the microSD drive

## Possible Complications
Although one never ejects the microSD card from the printer while it's still running, it's not unheard of to want to eject the USB drive while it's on.  A net share must be managed properly to make sure that data isn't lost.  Furthermore, OctoPrint's method of uploading files automatically from the USB into the /home/pi/.octoprint/upload/USB folder area makes state management a little complicated.

For these reasons, if you intend to share the USB drive itself, I would suggest that you simply leave it in the printer at all times.  The Samba configuration file will expect the USB drive to be mounted so this will make things simpler if you observe this restriction.

## Definitions
Samba:  This is a royalty-free Linux package that allows computers to behave like Microsoft servers.
SMB:  This is the protocol that Microsoft uses behind-the-scenes for serving up shares on a file server.
SMBD:  This is the SMB daemon that Samba runs, the server-side, if you will.
ssh:  Secure shell allows you to have a remote terminal session to the Raspberry Pi 3 in this case.
putty:  This is the Windows-compatible version of ssh.
share:  A share is a folder and its related set of files which are available to other computers.
map:  A remote computer "maps" a drive to a remote file server's share in order to use it locally.  After doing so, it will appear to be a local drive.
disconnect:  The antonym to mapping a drive.
eject:  The same as disconnecting from a mapped drive.
net use:  From a Windows command line, this is the command used to map a drive to a remote file server.
net use /delete:  The antonym to "net use".

## Instructions
Make sure that your printer is turned on.  Now run the following command in the next code section from a terminal session on your workstation.  If you're on a Windows-based computer, use [putty](http://www.putty.org) to connect to your printer by name or by IP address, using "pi" as the username.

If you've not done this before, make an attempt at connecting to your printer by its name (found on the label as "serial number").  Just make sure to append this with ".local".  Since my printer's name is "charming-pascal", I'll be using this below but you should replace this with your own printer's name throughout.

If this attempt fails, try to find your printer's issued IP address using the LCD interface:  Utilities -> Network Utilities -> Network Status.  See below for the alternate version using the IP address.

```
# Note that the default password for the pi user is "raspberry"
$ ssh pi@charming-pascal.local
# Alternately, using the IP address
$ ssh pi@192.168.0.51
```

If this fails to connect with a "Permission denied" error, make sure that you're including the "pi@" part of this.  If this fails to connect since it can't find the printer, then verify that you're using the correct IP address, perhaps querying your network's DHCP server to find out which address was issued.

All the commands below then are entered while in this remote session to your printer.

```
# It's important to run this next command before any attempt to update using apt-get later
$ sudo apt-get update
# Save a copy of your existing configuration file; you can always revert without ill effect
$ sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.save
# Although technically Samba is already installed, doing it this way will be cleaner
# since you need the CLI tools as well
$ sudo apt-get install samba samba-common-bin
```

Create a new subfolder called share under the location where Octoprint sees GCODE files.  That initial one in the mask means don't allow this folder to be deleted.

```
$ sudo mkdir -m 1777 /home/pi/.octoprint/uploads/share
```

Create a new SMB user "pi" for the `net use` share commands.  The password doesn't have to be "raspberry" but you'd need to remember what it is and to use that later when mapping the drive.

```
# When prompted, enter "raspberry" for the password for this to
# set the password used by Windows share users
$ sudo smbpasswd -a pi
```

Edit the Samba configuration file.  Edit the `global` section as required, then append the `microsd` share section at the end of the file.

```
$ sudo nano /etc/samba/smb.conf

[global]
workgroup=WORKGROUP
encrypt passwords=yes
wins support=no
# Set NETBIOS name below to match its DNS hostname in /etc/hostname
# noting that I've got this commented out at the moment. 
# netbios name=%h
# Set the description to Samba v[Version] on [NetBIOS name]
server string=Samba %v on %L

# Append this part at the end of the file

[microsd]
Comment=Pi share to the microSD share folder
Path=/home/pi/.octoprint/uploads/share
Browseable=yes
Writeable=yes
Read only=no
only guest=no
create mask=0777
directory mask=0777
Public=yes
Guest ok=yes
```

Having saved the file and exited (Ctrl-O, Enter, Ctrl-X), now parse it to verify that SMB likes what you did.

```
$ sudo testparm /etc/samba/smb.conf
```

Assuming that it's happy, now restart the SMBD service to load those changes.

```
$ sudo service smbd restart
# Now review what shares are being served up (in theory, this works)
$ sudo smbstatus --shares
```

## Testing the share
### OS X
In Finder, press Cmd-K to map a drive, entering something similar to:

smb://charming-pascal.local/microsd

Enter "pi" as the user and "raspberry" as the password.

Copy a GCODE file into this mapped drive to verify that you have the rights to do so.

### Windows
Open Explorer (My Computer).  In some versions of Windows, you right-mouse click My Computer once this program is loaded and choose Map Drive from here.  In other versions, you choose this option from the Tools menu at the top.  Enter the path as `\\charming-pascal\microsd`, the drive letter as `P:` or similar, the user as `pi` and the password as `raspberry`.

You may also map a drive in Windows from a command prompt:

```
C:\> net use p: \\charming-pascal\microsd /user:pi raspberry
C:\> dir p:
```

As before, copy a GCODE file into this newly-mapped drive to verify that you have the rights to do so.

## Verifying that it was uploaded
Within the OctoPrint interface, go to the Files area, drill down into the new `share` folder and verify that your file was uploaded successfully.  Depending upon your timing, you may need to press the Refresh button in the File area of the interface.

## Disconnecting from the mapped drive
Feel free to disconnect from the mapped drive if you're no longer interested in copying files to the printer.  In OS X, this involves "Ejecting" the drive.  In Windows, this involves "Disconnecting" from it.

