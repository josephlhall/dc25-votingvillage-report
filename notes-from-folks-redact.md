# Diebold ExpressPoll 5000

(This is not a voting machine but an [e-pollbook][6])

We have additional resources from this device in this repository:

* [ExPoll.resources][35] -- A Windows CE `.resources` file for this
  device (BINARY).
* [options.data][36] -- What appears to be a configuration file for
  the ExpressPoll application software.
* [schema.sqlite3][37] -- the DB schema of the data on the device.

## Sean & TJ

Sean Roach ([@TheEdgyDev][1]), TJ Horner ([@tjhorner][2]), Ian Smith,
Morgan Jones, Thomas Fulmer, Thomas Richards

Found a few physical things and a few software things. Main physical
thing: there was an sqlite 3 database on compactflash (CF) which is
written/stored/etc. in the clear (unencrypted). The only thing
stopping someone from taking this CF card out is one screw, a phillips
head screw -- not even a type of security screw. This would have
allowed us to be able to do a DoS attack, by simply removing the card.

While we didn't have access to a resident database (JLH: this may have
been after discovery of a resident DB with 650k voter records on it)
we were able to make a schema of that db, and initialize it without
any users. Then we could put it back in with no users. Everyone
afterwards would not be registered in that county or precinct, they
would be identified as not registered in that polling place or
precinct, and they would have likely had to go to another polling
station (JLH: or cast a provisional ballot). In this case, the typical
30 minute line would grow precipitously. One could selectively DoS
counties or precincts that are for your opponents candidate.

The (default) username and password for this unit was released on a
PDF by State of Maryland (username as 1 password as 1111). We
iteratively created a correct database as follows: we first plugged in
a blank drive because we didn't have a database. A very helpful
succession of error messages would appear, for example, "You are
missing the following sqlite file." We would then create the missing
file... then it would say "there is the following missing table", and
we would create the named table. We did this to create each file,
table, and field in each table (all blank of course as we created them
from scratch). The schema looks like so (JLH: rinon has the
schema). There was a consolidation ID which we turned into
`defcon`. This gave us access to the system with no more errors.

We then attempted to see if bad data in the database could cause
problems and began trying use a bashbunny -- a USB thing that emulates
a keyboard. We were sending `A` characters at it to overflow the the
text field. The thinking is that this is a Windows CE 5, .Net app; and
we were trying to get it to fill RAM and the RAM manager would then
close that app, crashing the application. (JLH: as outlined in TJ's
post, this didn't do more than slow the unit down and "time stopped".)

We scanned and then used wireshark to investigate its networking. Open
ports included port 80, 443, and 5002.  5002 was a bit weird and it
seems like an error broadcast, doesn't accept any connections. 443
closes the connection immediately. 80 is the default Windows CE server
("Welcome to Windows CE!...") or something; itallows connecting shuts
down quickly if we try to send too much data.

We then explored the possibility of doing unauthorized firmware
upgrades. The device looks for `EBOOT.bin` (bootloader) and `NK.bin`
(kernel). If `Eboot.bin` is present, the device will perform a
bootloder update with that file; if `NK.bin` is present, it will load
that into RAM an replace the kernel/OS on the device with it. The
device will find these files if they are present on the CF card,
download them into RAM and the flash the bootloader or update the
kernel. No signature checking is performed.

We explored some attack vectors with the smartcard reader on the
device. We considered what if we could make it so that the card
silently discards your vote? We think we can (JLH: how far did this
team get?) modify a smartcard with this unit so that when you
check-in/register everything is fine, but we can write an invalid
consolidation id to the card.  This would allow the voter to cast a
vote, but it would not be counted on the backend because only the
votes associated with a specific consolidation id would be tallied.

## TJ additional write-up

TJ has a fantastic additional write-up [here][3]. Notably:

* It runs Windows CE 5.0
* ExpressPoll software is a giant .NET app with the assembly name `ExPoll`
* ExpressPoll software uses WinForms for UI
* Bootloader is proprietary
* Processor’s architecture is ARM
* Database file is stored on memory card (more details later)

Several exploits were tested against this, such as the usual
suspects: buffer overflow, Windows key shortcuts, network exploits,
etc.

When the memory card is inserted into the machine and it boots up, the
bootloader first checks for a file called `NK.BIN`. If this file is
found, it will attempt to load it into RAM and boot into it with
absolutely no signature verification. The firmware is then flashed and
is permanent upon next boot.

The `NK.BIN` file is generally a Windows CE 5.0 update for the machine
bundled with the ExpressPoll software, but an attacker is able to
upload and run any custom NT kernel, and possibly a Linux image
(untested thus far).

We tested this on our ExpressPoll 5000, and sure enough, it loaded our
Windows CE without complaining: [image of loading NK.bin from CF card]

Very similar to the firmware injection exploit, the bootloader also
checks for an update to itself while booting, with the file name
`EBOOT.BIN`. It is also loaded without signature verification, but we
have not been able to get a bootloader to successfully work on a
machine. However, the bootloader injection itself is confirmed to work
since the bootloader we put onto the `EBOOT.BIN` file is indeed
flashed and is present upon reboot without the memory card. This also
means we basically bricked the thing (we had to get another, heh).

### .NET `.resources` File Override

When the ExpressPoll software is "launched" from the "Launch Express
Poll" button, it reads the memory card for a file named
`ExPoll.resources`. This is a resources file defining any resources
that the app will use, such as translation strings, images, buttons,
layouts, etc. It would generally be used for overriding and
customizing the registration process (for example, adding your own
logo instead of having the Diebold logo.)

The weak point in this system is that buttons (and potentially other
UI elements) can be overridden to take different actions, such as run
an executable stored on the memory card (which is mounted in something
like `/Storage Card`) or run a command.

If an `ExPoll.resources` file is found, it will first load it into
memory, use it for that session, and also copy it to the device’s
internal memory so it can be used again.

This vulnerability was confirmed by creating a random .NET
`.resources` file, naming it `ExPoll.resources` and copying to the
memory card. We popped it into the machine, pressed the "Launch
Express Poll" button, and (as expected), it errored. Woohoo! It loaded
the file. Sadly, since it also copied to memory, it is essentially
bricked if you give it a bad file (you can’t get to the "Launch
Express Poll" button). Sadly, this was the case with our device so I
don’t have any images of it in this state.

### Unencrypted SQLite3 Database

When starting the ExpressPoll software, it looks for a file called
`PollData.db3` on the memory card. This is an standard unencrypted
SQLite3 database, including *literally all the information*. Polls,
parties, voters, all of it. (Link coming soon with sample database.)
Using this information, an attacker could launch several types of
attacks:

* Data exfiltration
   * For example, the attacker could easily come with a card that has
     an empty database, and silently swap out the card when they
     register to vote, thus obtaining the entire voter database (which
     includes: name, address, last 4 of SSN, signature, and many
     others) for the county.
* Falsifying voter information
   * For example, the attacker could forge their own database with
     fake voter information (e.g. change their parties/signatures,
     etc.) and place it into the machine when they register to vote.
* Other attacks. Your imagination is the limit when you have access to
  the entire database.

(JLH: Get schema from TJ.)

### 2 Open USB Ports

These pose less of a threat, but it is still threatening. An attacker
could plug any USB device they wanted (for example, a USB Rubber Ducky
or LAN Turtle) to launch a creative attack. I have tested this with my
Bash Bunny by spamming the letter “a” infinitely into one of the text
boxes in an attempt to trigger a buffer overflow and potentially crash
the .NET app (it didn’t work).

## Stephen (@rinon), Heather (github: heathervm)

We were working with an ExpressPoll 5000 software version 2.0.27. This device
has a CF card slot and a PCMCIA storage card slot on the back, covered by a
cover with a hole that appears to be for attaching a tamper evident seal to this
cover. The device also has two unprotected USB ports and an RJ45 (ethernet) port
on the back.

We also started by recreating a database schema from SQL error messages that the
device helpfully provided for the `PollData.db3` on removable storage. After
finding that the default password (1111) from
the
[Maryland State Board of Elections ExpressPoll Pollbook Acceptance Test Guide][26] still
worked we were able to reconstruct the following (incomplete) database schema:

    CREATE TABLE Options (int id);
    CREATE TABLE Consolidations (consolidationNum int, consolidationId, consolidationName, countyId, vCenterId, pollName, pollAddress, pollCityStateZip, pollTelephone);
    CREATE TABLE users (username, password);
    CREATE TABLE Precincts (consolidationId, precinctId, precinctNumber, portion, portions, ballotId, reportingUnitId, baseUnitId);
    CREATE TABLE Countys (countyId, gemsElectionFile, gemsElectionId, gemsDlVersion, gemsElectionName, countyName, countyNumber);
    CREATE TABLE Partys (sortOrder, partyId, name, abbreviatedName, vGroup1Id, vGroup2Id);
    CREATE TABLE Ballots (ballotId, precinctId, style, name);
    CREATE TABLE ConsolidationMaps (consolidationId);
    CREATE TABLE Voters (voterId, precinctId, name, status, idRequired, absentee, nameLast, nameFirst, middleInitial, partyId);
    CREATE TABLE Streets (streetName, houseFrom, houseTo, streetId, city, state, zip, side, apartmentFrom, apartmentTo, precinctId);

As others have highlighted, information in this database is unencrypted and
available for reading and writing by anyone who can access the back panel of the
device. In the field this panel should be protected by a tamper evident seal,
but this is a very weak mitigation.

We also tested providing a modified `ExPoll.resources` file to the device. It
does appear that this file is cached, at least in part, since modifications to
this information persist even after the file is removed from flash/PCMCIA
storage. This file is a .NET 1.1 resources file that programs or customizes the
UI of the pollbook. We were unable to determine exactly how this UI is
customized, but adding a valid resources file dramatically altered the UI of the
machine and we were unable to revert these changes. We tried overflowing string
values in this resource file to no effect.

The device keeps a log database (sqlite3) containing an event log with login,
logout, power, load, and open events. However, this log would not be sufficient
to prevent tampering. It is only written by the device and does not reflect any
file changes that occur on the disks.

### Hardware information

Main Processor: Intel X Scale (ARMv5) PXA270. Looks to be packaged by Marvell.

Flash Memory: Numonyx StrataFlash (P30), mounted on a small daughterboard that
plugs into the corner of the main PCB.

Other components:
- ATMel Two-wire Serial EEPROM (AT24C08A iirc?)
- XILINX XC2C64A CoolRunner-II CPLD
- Samsung K4M56323LG SDRAM
- TI SN74LVCH16T245 16-bit Dual-supply Bus Transceiver

### Miscellaneous Resources

ExpressPoll 5000 is certified for use in [AZ][27], [IN][28], [VA][29], [PA][30], [FL][31], [MD][32] ([interesting addendum report][33]), [GA][34] (and more, I got tired of searching).

Apparently ES&S sold refurbished units as recently as 2016:

    Remarks: The Board of Public Works approved the State Board of Elections awarding a
    contract for electronic poll books: 150 refurbished ExpressPoll 5000 units and 150 ExpressPoll
    printers with battery back-up and power brick. The contract amount was $178,050.

(from http://bpw.maryland.gov/MeetingDocs/2016-July-6-Agenda.pdf)

# AVS WinVote

## Carsten Schürmann

From Carsten:

I was the one who hacked the WinVote machine in 90 minutes. I did
neither touch nor get closer to the machine then 10 meters.

The reason that it took me 90 minutes was not that I had to buy a USB
keyboard, but that I went to Barbara Simons talk, which took 60
minutes.  And then there was 20 minutes of Harri Hursti’s introduction
to the voting machine hacking village, give or take a few.  (JLH
erroneously attributed on a [public radio program][38] that the first
hack of Friday of the AVS WinVote to Nick below, who went out to get a
keyboard.)  The hack was not a USB hack, it was wireless hack.  Recall
that the the Winvote always has wireless turned on.

It took me a few minutes to figure out the IP address of the machine
(which was `100.100.7.151`). It too me a few seconds to break into the
machine and get access to the vote database using he
[MS03-26-DCOM][39] exploit from 2003.

* I could read and update the vote database;
* I could mirror whatever was shown on the machine on my laptop; and,
* I could turn the machine off (which always surprised the people
  standing around it).

The bottom-line is that the WinVote can be hacked from a car outside
the polling station (but still within range of the wireless signal)
and as far as 150 —300 ft away, without much preparation and
incredible quickly and effectively.

For example, this can be leveraged across an election jurisdiction;
You can cover a lot of ground driving from polling station to polling
station in the afternoon of Election Day.

The machines have been used in 2004, 2008, 2012, and the exploit I
have used was applicable already then.

BTW, when you use Remote Desktop (password: `admin`), you can even start
the Task Manager wirelessly — with all administrator privileges.

## Nick

Nick ([email redacted])

Nick and a few others were working on the AVS WinVote.  The first
thing they did is to attempt to get into the device.  There was a
locked panel on the front of the device; however 1) the lock is an
easily-pickable 3-pin lock, typical of hotel mini bars (meaning the
keys are in wide circulation and [available for very cheap ($10)
online][7]); and, 2) the hinge on the cover of the locked compartment
was easily compromised without damage.  This compartment contained a
printer (presumably for close-of-polls results printing) and a USB
port.  This physical security protecting the USB port was ineffective,
due to the findings in the next paragraph.

While examining the case further, they noticed that there were 2
unprotected/uncovered USB ports on the back of the machine.  These
were within easy reach of a voter, and unfortunately the privacy
screen for the AVS WinVote is so private that it would be difficult to
tell someone was messing with USB ports on the back of the machine
while voting.  They next thought of attaching a USB keyboard.
Unfortunately, there was no USB keyboard around in the Voting Village
and we had to run an errand to get one at a store. When we returned,
all we had to do was to simply attach the keyboard, type ctrl-al-del
(the "three-fingered salute" for Windows), and the Windows task
manager pops up. At that point, we can type `alt-f run` and run any
software we want to, including software on a USB stick that is
inserted into the other USB slot on the back of the machine. Seems to
accept arbitrary USB drive contents. (JLH: This was the first hack of
the Village, coming around 1:40; the major delay involved procuring a
USB keyboard.)  Nick said, "With physical access to back of the
machine for 15s, an attacker can do anything."

The unit surprisingly contained a WiFi chipset. It was using WEP
104/40-bit encryption -- not even 128-bit WEP (which is still easily
broken).  The WEP keys are hard-coded.  There are definitely open
ports on the device; Nick recommended checking out [MS08067][8] -
which is a .Net API/RPC exploit vulnerability. Nick was not able to
pop a shell using this, but it seemed to work but then hung the
machine.  There is a modem with RJ11 (phone) jack, and it will easily
communicate externally. Looking at the file format: there was a *file
system*... but no files. But when one opens unallocated space in a hex
editor, he could see that the files are still there, just not securely
erased.

## Alfredo

Alfredo Ortega ([@OrtegaAlfredo][5])

Alfredo and a few others performed an RF survey of the WinVote
printer, in order to determine if a TEMPEST attack would be possible
from a reasonable distance. The unit was emitting a bunch of RF. We
were not able to correlate with specific actions of the device. Some
very low signal in our measurements, but would need much, much more
data to determine anything. We used [HackRF One][4], a software
defined radio kit.

# Sequoia AVC Edge

## Nick

Nick ([email redacted]) also messed around with the AVC Edge.

The AVC Edge has an internal CompactFlash (CF) card. In terms of OS,
it's running [pSOS][9], a real-time OS developed in 1989 (unclear when
this version of the operating system was from). This OS was used
heavily in retail equipment. Records a lot of data as a hex files;
hard to easily figure out what the results mean without a bit more
information or reverse-engineering of the file format.  Results are
stored, but then also sent to flash storage on a PCMCIA card in a slot
in the back of the machine.  This particular AVC Edge was from the
24th precinct of Washington DC, which Nick got by just running
`strings` on the data; there may be more with more careful
examination.  Nick definitely could see the candidates and such, no
voter identities or similar.  He tried to boot the OS image in a VM,
but didn't get too far; He could see a menu in the PSOS boot file,
that is, could see the strings, but couldn't get it to boot.  There
was a RAM file that seemed to give them a "file not found" in the boot
sequence.  One of the PCMCIA slots can update the ballot and the other
is for getting results off the machine.  Saw some stuff with `binwalk`
that indicated that there may be use of an *8-bit cipher* (yes eight
(8) bit).  Last update to display firmware (video output devices) was
in 1989; that is, at least one file there had not been updated since
1989. The data was from the 2008 election -- Obama, McCain, etc. Nick
also saw a serial port on back that looked like it could be fun.

## University of Houston Cybersecurity Club

Tsukinaki ([email redacted], [@Whiskeys373n][10]) and Joe ([email
redacted])

They found a 16MB CompacFlash card, embedded on the main circuit
board. It held what appeared to be a proprietary operating system for
the machine, all the uploads of ballot information were in plain
text. We looked at the drivers for the devices. And then examined the
two PCMCIA interfaces on the outside, on the back of the machine. One
slot is used to load the ballot information, and the other records the
count and then offloaded after the election. No hardware or software
encryption.

# ES&S iVotronic

## Scott

Scott Brion

ES&S iVotronic

1. Components Analyzed
   1. PEBs (Personal Election Ballot)
   2. PEB Reader
   3. iVotronic voting machine
2. Components not analyzed
   1. The election management software (UNITY)
   2. Compact Flash card

### Technical Findings

#### PEBs

The PEB contains an 8 bit processor, eprom, flash memory, infrared
port, magnet, serial pins and a battery. The PEBs is identified by a
unique serial number. The PEB controls authentication and access to
the voting machine. The pin-outs for the serial connection are as
follows;

* pin 1, 
* pin 2, 
* pin 3, `IN` 
* pin 4, `OUT`
* pin 5, `Ground`
* pin 6, `5v`

#### PEB Reader

The PEB reader contains an EPROM, 8-bit processor, USB port, serial
pins, and an IRdA receiver port.  Communication to the firmware was
established through the serial PINs, however nothing of value was
obtained. The pin-outs for the serial connection are as follows;

* pin 1,
* pin 2,
* pin 3, `IN`
* pin 4, `OUT`
* pin 5, `Ground`
* pin 6, `5v`

#### Voting Machine

The iVotronic machine is a touch screen interface with a PEB slot,
compact flash slot, audio port and standard DB9 serial port.  A
functioning PEB is required to activate the voting machine.  The
compact flash and serial port are externally accessible and can be
used to extract data through system functions. There are unique
passwords for each configuration menu item. These passwords were
discovered with a google search (“ivotroni password reset PEB”).

## Kris

Kris Hardy ([email redacted])

Kris and his team focused on attacking access to the PEB and PEB
readers (the ES&S iVotronic uses a small device called a personal
electronic ballot (PEB) to both activate the machine for each voter,
as well as transmit the ballot style to the unit so that it displays
only the races that the voter is entitled to vote on).

They first hooked up a hardware debugger ([PICkit 3][11],
debugger/programmer) to various chips on the PEB or PEB reader
circuit boards.

The microcontroller in the PEB is a `PIC16F873`.  They were able to
pull firmware from one of the chips.  The other two were garbage; the
security fuses on the chips may have been blown to prevent reading out
the software.  They got the firmware off of one of the chips and have
decompiled it.  It looks reasonable, but they did not have time to do
much more before the village closed shop on Sunday.

There is a flash memory chip on the board, an Atmel AT58DB161B SPI
Flash memory chip (JLH: Not sure the AT58 exists; maybe this should be
[AT45DB161B][12]). Chip for the IRdA port (IR input/output) is the [TI
TIR1000][13].  They had not yet pulled the data off of the flash
memory off of the Atmel chip.

Here is the pin out (using the PICkit 3):

For the PEB (chip pin, chip function):

* pin 1, `pgd`
* pin 2, `mclr/vpp`
* pin 3, not sure
* pin 4, `pgc`
* pin 5, `pgm`
* pin 6, `vss`
* pin 7, `bdd`

For the PEB reader:

* pin 1 & 2, unsure (didn't use)
* pin 3, `pgc`
* pin 4, `pgd`
* pin 5, `vss`
* pin 6, `vdd`
* pin 7, `mclr/vpp`
* pin 1, `pgm` (unlabeled; used `j3` screen on the PEB reader PCB. Pin
  is not connected to the header, instead connected with a jumper).

Toolchain for reading (that is, IDE toolchain) was [Microchip
MPLAB-X][14] and for decompilation they used [gputils][15].

PEB reader: PIC model: [PIC18F2455][16].  They were Able to pull the
firmware off of one of those as well.  Have not decompiled, security
fuses may be set here, in which case it will be garbage.

# Premier AccuVote-TSx

Joe FitzPatrick ([@securelyfitz][17]), Schulyer St. Leger
([@docprofsky][18]) ([email redacted]), Ryan (github: [rqu1][19],
[@rqu45][20]), Wasabi ([email redacted]), Ayushman ([email redacted])

Wasabi mapped out that one EPROM goes to the battery controller, and it's clear that
when this PIC is removed, nothing works anymore.  That is an easy DoS
vector as the PIC is in a socket (no desoldering required), although
you would have to take off the entire back cover to access it. Wasabi
was able to extract the `NK.bin` contents and found a LAN and dialup
modem console in the firmware.

Ayushman was exploring how careful this device is with memory care.
There is an `.ini` file that has the passwords, users, modem
configuration for the device.  20k chars in the key/value pairs, it
would probably be easy to pop into some sort of terminal here and read
and/or change elements in that file.  You would need a serial (`db9`)
port in order to interact with the terminal.

rqu1: has put [dumps up of two the EPROMs][25]: one from the battery
controller, one from the modem.  The modem is interesting as that's a
form of networking: ran strings on the firmware of TSx, found the
company/brand, WaveLAN, that makes PC WiFi cards, with 2.4GHz Wifi.

Joe and Schuyler focused on getting the firmware off the TSx via the
JTAG interface, which allows access to the chip and system. To do
this, Joe and crew first found out that this is an ARMv5 chip design,
and common chip programmer/debuggers only work with ARMv6 and later.
After getting the right debugger that could do ARMv5, they used
`openocd` v10 ([Open On-Chip Debugger][21]). The command line uses
default files and we ran the following commands:

    openocd -f interface/um232h.cfg -f target/pxa255.cfg

(one for the tool, one for the chip).  Using the UM, it told me the ID
code of the device (`0x69264013`) which is a [PXA255 Intel chip][22],
that's the `.cfg` file. In terms of hardware configuration, they used
an [Adafruit FT232H breakout board][23].  This was a standard ARMv5,
20-pin.  Here's the pinout (debug header pin, chip function):

* pin 3, connects to ft232h `c0`, `treset`
* pin 4, connects to debugger pin `ground`, and is ground (could be
  any ground)
* pin 5, `tdi`, connects to d1 on debug adapter
* pin 7, `tms`, connects to d3 on debug adapter
* pin 9, `tclk`, goes to d0 on debug adapter
* pin 13, `tdo`, connects to d2 on debug adapter
* pin 15, `sreset`, connects to c1 on debug adapter

It's now hooked up and running a server; they opened a new console,
opened a terminal, and did `telnet localhost 444` which opens up
console on the machine.  Then they typed directly `reset halt` to
reboot.  Now the system is rebooted in the debugging rig; they could
then step through instructions one by one. They could also start up
`gdb` in another window.  To do this run `gdb-multiarch`. After gdb
is running you can type: `set arch armv5te` to set the target
architecture. Then point it to the gdb port with `target remote
localhost:333`.  There were some small quirks; but the result was
mostly like running gdb on a regular program.  To dump the firmware,
they did `dump_image reset.bin 0x0 0x1000000` which resulted in a
nearly ~16MB image. Then they used [binwalk][24] v2.1.1 (firmware
analysis tool). By typing `binwalk -E reset.bin` we can measure the
entropy, and that indicated the firmware is about 11MB, with a
compressed 3MB file towards the beginning, which could be
interesting. The command `binwalk -e reset.bin` identifies files and
extracts them. The most important file was clearly `NK.bin` (.Net
kernel). By running strings we saw mostly copyright information
embedded in the software, and a lot of certificate information.

# Back-Office Simulated Network

(JLH: Need to get in touch with Bash Kazi, who did this work.)


# Models of equipment in the Village

Courtesy of Ryan Macias, who kept good notes on make, model, and
software versions. (please let us know if you have any additional or
different information!)

* ES&S iVotronic -- Hardware Revision 1.1 (ES&S Code IV 1.24.15.a);
  * PEB (personalized electronic ballot module) version 1.7c-PEB-S
    **[Note: the first set of information is from the voting unit, the
    second is from the PEB]**
* Premier AccuVote -- TS Unit Model Number AV-TSx; firmware 4.7.8
* Sequoia AVC Edge; version 5.0.24 **[Note: unsure if Edge I or Edge
  II]**
* AVS WinVote version 1.5.4 **[Note: did not capture HW version]**
* Diebold Express Poll 5000; versions 2.1.1, 2.0.27

[1]: https://twitter.com/TheEdgyDev/
[2]: https://twitter.com/tjhorner/
[3]: https://blog.horner.tj/post/hacking-voting-machines-def-con-25
[4]: https://greatscottgadgets.com/hackrf/
[5]: https://twitter.com/OrtegaAlfredo
[6]: https://en.wikipedia.org/wiki/Electronic_pollbook
[7]: http://timeclocksusa.com/134-Time-Clock
[8]: https://technet.microsoft.com/en-us/library/security/ms08-067.aspx
[9]: https://en.wikipedia.org/wiki/PSOS_(real-time_operating_system)
[10]: https://mobile.twitter.com/Whiskeys373n
[11]: http://www.microchip.com/Developmenttools/ProductDetails.aspx?PartNO=PG164130
[12]: http://www.atmel.com/Images/doc2224.pdf
[13]: http://www.alldatasheet.com/view.jsp?Searchword=IR1000
[14]: http://www.microchip.com/mplab/mplab-x-ide
[15]: https://gputils.sourceforge.io/
[16]: http://www.microchip.com/wwwproducts/en/PIC18F2455
[17]: https://twitter.com/securelyfitz
[18]: https://twitter.com/docprofsky
[19]: https://github.com/rqu1/
[20]: https://twitter.com/rqu45
[21]: http://openocd.org/
[22]: http://www.datasheet4u.com/pdf/PXA255-pdf/359785
[23]: https://learn.adafruit.com/adafruit-ft232h-breakout/overview
[24]: https://github.com/devttys0/binwalk
[25]: https://github.com/rqu1/HackTheElection
[26]: https://www.eac.gov/assets/1/28/PollbookAcceptanceTest_v1.3_03092016.pdf
[27]: https://www.azsos.gov/sites/azsos.gov/files/2016_0220_-_official_list.pdf
[28]: https://www.in.gov/sos/elections/files/Vendor_Certified_Equipment_Configuration.pdf
[29]: http://www.elections.virginia.gov/registration/voting-systems/
[30]: http://www.dos.pa.gov/VotingElections/Documents/Voting%20Systems/ExpressPoll%205000%20w%20EZRoster%202.7.12.4,%20Cardwriter/Sec%20Rep.pdf
[31]: http://dos.myflorida.com/media/695120/dominion-gems-release-1216-version-1-revision-2-certification.pdf
[32]: http://elections.maryland.gov/pdf/Minutes_07-24-2006.pdf
[33]: https://www.ola.state.md.us/Reports/Election%20status%20reports/ElectionStatusReport9-28-06.pdf
[34]: http://www.co.camden.ga.us/DocumentCenter/Home/View/2404
[35]: https://github.com/josephlhall/dc25-votingvillage-report/blob/master/resources/ExPoll.resources
[36]: https://github.com/josephlhall/dc25-votingvillage-report/blob/master/resources/options.data
[37]: https://github.com/josephlhall/dc25-votingvillage-report/blob/master/resources/schema.sqlite3
[38]: https://www.sciencefriday.com/segments/hacking-the-vote-how-can-we-secure-our-voting-systems/
[39]: https://technet.microsoft.com/en-us/library/security/ms03-026.aspx
