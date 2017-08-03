# Work on the ExpressPoll 5000

Sean Roach ([@TheEdgyDev][1]), TJ Horner ([@tjhorner][2]), Ian Smith,
Morgan Jones, Thomas Fulmer

Found a few physical things and a few software things. Main physical
thing: there was an sqlite 3 database on compactflash (CF) which is
written/stored/etc. in the clear (unencrypted). The only thing
stopping someone from taking this CF card out is one screw, a philips
head screw -- not even a type of security screw. This would have
allowed us to be able to do a DoS attack, by simply removing the
card.

While we didn't have access to a resident database (JLH: this may have
been after discovery of a resident DB with 650k voter records on it)
we were able to make a schema of that db, and initialize it without
any users. Then we could put it back in with no users. Everyone
afterwards would not be registereed in that county or precinct, they
would be identified as not registered in that polling place or
precinct, and they would have likely had to go to another polling
staton (JLH: or cast a provisional ballot). In this case, the typical
30 minute line would grow precipitously. One could selectively DoS
counties or precicnts that are for your opponents candidate.

The (default) username and password for this unit was released on a
PDF by State of Maryland (username as 1 password as 1111). We
iteratively created a correct database as follows: we first plugged in
a blank drive because we didn't have a database. A very helpful
succession of error messages would appear, for example, "You are
missing the following sqlite file." We would then create the missing
file... then it would say "there is the following missing table", and
we would create the named table. We did this to create each file,
table, and field in each table (all blank of course as we created them
from scratch). The schema looke like so (JLH: rinon has the
schema). There was a consolidation ID which we turned into
`defcon`. This gave us access to the system with no more errors.

We then attempted to see if bad data in the database could cause problems
and began trying use a bashbunny -- a USB thing that
emulates a keyboard. We were sending `A` characters at it to overflow the the
text field. The thinking is that this is a Windows CE 5, .Net app; and
we were trying to get it to fill RAM and the RAM manager would then close
that app, crashing the application. (JLH: as outlined in TJ's post, this didn't do more than slow the unit down and "time stopped".)

We scanned and then used wireshark to investigate its networking. Open
ports included port 80, 443, and 5002.
5002 was a bit weird and it seems like an error broadcast, doesn't
accept any connectinons. 443 closes the connection immediately. 80
allows connecting to something but shuts down quickly if we try to
send too much data.

We then explored the possibility of doing unauthorized firmware upgrades. The
device looks for `EBOOT.bin` (bootloader) and `NK.bin` (kernel). If
`Eboot.bin` is present, the device will perform a bootloder update
with that file; if `NK.bin` is present, it will load that into RAM an
replace the kernel/OS on the device with it. The device will find
these files if they are present on the CF card, download them into RAM
and the flash the bootloader or update the kernel. No signature
checking is performed.

We explored some attack vectors with the smartcard reader on the
device. We considered what if we could make it so that the card
silently discards your vote? We think we can (JLH: how far did this
team get?) modify a smartcard with this unit so that when you
check-in/register everything is fine, but we can write an invalid
consolidation id to the card.  This would allow the voter to cast a
vote, but it would not be counted on the backend because only the
votes associated with a specific consolidation id would be tallied.

### TJ additonal write-up

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

#### .NET `.resources` File Override

When the ExpressPoll software is "launched" from the "Launch Express
Poll" button, it reads the memory card for a file named
`ExPoll.resources`. This is a resources file defining any resources
that the app will use, such as translation strings, images, buttons,
layouts, etc. It would generally be used for overriding and
customizing the registration process (for example, adding your own
logo instead of having the Diebold logo.)

The weak point in this system is that buttons (and potentially other
UI elements) can be overriden to take different actions, such as run
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

#### Unencrypted SQLite3 Database

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

#### 2 Open USB Ports

These pose less of a threat, but it is still threatening. An attacker
could plug any USB device they wanted (for example, a USB Rubber Ducky
or LAN Turtle) to launch a creative attack. I have tested this with my
Bash Bunny by spamming the letter “a” infinitely into one of the text
boxes in an attempt to trigger a buffer overflow and potentially crash
the .NET app (it didn’t work).

# AVS WinVote

### Nick

Nick ([redacted])

We were wokring on the winvote, getting into it, 2 USB ports on the back. Just had to attached a keyboard, ctrl-al-del, task manager... Alt-f run... takes arbitrary usb drive contents. WiFi chipset. WEP 104/40 not even 128. Physical access to back of the machine for 15s can do anything. Open ports, MS08067 - Net API exploit vulnerability. Wasn't able to pop a shell, but it seemed to work but hung the machine. WEP keys are hard-coded. Modem RJ11 jack, will communicate externally. Looking at the file format ... there was a file system but no files. But open in a hex editor and you could see that the files are still there and not securely erased. 3-pin key over the guts.

### Alfredo

Alfredo Ortega ([@OrtegaAlfredo][5])

Alfredo and a few others performed an RF survery of the WinVote
printer, in order to determine if a TEMPEST attack would be possible
from a reasonable distance. The unit was emitting a bunch of RF. We
were not able to correlate with specific actions of the device. Some
very low signal in our measurments, but would need much, much more
data to determine anything. We used [HackRF One][4], a software
defined radio kit.

# Sequoia AVC Edge

Nick ([redacted])

Edge internal CF, runninc PSOS, real-time OS (developed in 1989) used in retail equipment. Records a lot of data as a hex files... hard to figure out what the results mean. Results are stored, but then sent to PCMCIA. 24th precinct of DC. Candidates and such. Tried to boot in a VM... menu in the PSOS boot file... can see the strings. Also a RAM file that gives us file not found. One of the PCMCIA slots can update the ballot and the other is for getting results off the machine. Saw some stuff with binwalk that there may be use of an 8-bit cipher. Last update to display (video output devices) was in 1989... at least one file not updated since 1989. 2008 election, Obama, McCain, etc. Serial port on back looks like it could be fun.

# iVotronic

Brian Sean

At first we were stimied by a password prompt.

...

# TSx

Joe FitzPatrick (@securelyfitz), Schulyer St. Leger (@docprofsky) ([redacted]), Ryan (github: rqu1, @rqu45), Wasabi ([redacted]), Ayushman ([redacted])

rqu1: will put dumps of two the ERPOM, one from the battery controller, one from the modem. will post on github. string in the firmware of TSx, company/brand makes PC cards, with 2.4GHz Wifi (WaveLAN).

wasabi: EPROM goes to battery controller, PIC that when it's removed, nothing works anymore.

ayushman: memory care, .ini file has the passwords, users modem configuration. 20,0000k chars in the key/value pairs, pop into some sort of terminal. Serial (db9) port needed in order to interact with the terminal.

getting the firmware off right now. openocd v10. command line uses default files. "openocd -f interface/um232h.cfg -f target/pxa255.cfg". one for the tool, one for the chip. Using a um, it told me the ID code of the device (0x69264013). Intel chip. cfg file. Hardware config, ft232h (chip) adafruit breakout board. Standard ARM 5, 20-pin. Debug header pin 3, connects to ft232h c0, treset. Debug header pin 4, connects to debugger pin ground, and is ground (could be any ground). debug header pin 5, tdi connects to pin d1 of the debug header. debug connector pin 5, tdi, connects to d1 of the debug adapter. debug header pin 7, tms, connects to d3 on debug adapter. debug header 9, tclk, goes to pin d0. debug header pin 13, tdo, connects to d2. debug header pin 15, sreset, connects to c1.

that is a server, in a new console, open a terminal, `telnet localhost 444` opens up console type directly `reset halt`. Now the system is rebooted in debugging rig... can use step to step through instructions one by one. Can also start up gdb in another window. To get `gdb -multiarch`. After gdb is running: `set arch armv5te`. `target remote localhost:333` (gdb port). Some quirks... but mostly like running gdb. `dump_image reset.bin 0x0 0x1000000` (~16MB). From that used binwalk v2.1.1 (firmware analysis tool). `binwalk -E reset.bin` entropy indicated the firmware is about 11MB, compressed 3MB file towards the beginning. `binwalk -e reset.bin` identifies files and extracts. Most important file: NK.bin (.Net kernel). Copyright information, and a lot of certificates.
. Copyright information, and a lot of certificates.

# Sequoia AVC Edge

University of Houston Cybersecurity Club

Tsukinaki ([redacted])
@Whiskeys373n Joe ([redacted])

16MB CF card, embedded on the main circuit board. holds a proprietary operating system for the machine, uploads ballot information plain text. drivers for the devices. two pcmcia interfaces onthe outside. one used to load the balot information, also to record the count and then offloaded. No hardware and software encryption.

# ES&S iVotronic

Kris Hardey ([redacted])

PEB access, PEB readers. microcontroller in the PEB: PIC 16f873. Able to pull firmware from one of them... the other two were garbage (Security fuses may have been blown). Got the firmware off of one and have decompiled it, looks reasonable. Atmil AT58db161b SPI Flash memory chip. Chip for IR TI IR1000. Not yet pulled the flash off of the Atmil chip.

Pin out: PICKit 3, debugger/programmer.

PEB: programmer header, pin 1, pgd; pin 2, mclr/vpp; pin 3, not sure; pin 4, pgc; pin 5, pgm; pin 6, vss; pin 7 bdd.

PEB reader: pin 1 & 2, unsure (didn't use); pin 3, pgc; pin 4, pgd, pin 5 vss, pin 6 vdd pin 7 mclr/vpp. pin 1 used j3 screen on the PEB reader PCB. pgm pin is not connected to the header, connected with a jumper.

Toolchain for reading (IDE toolchain): Microchip MPLabX
Decompilation: gputils

PEB reader: PIC model 18f2455. Able to pull the firmware off of one of those as well. Have not decompiled, security fuses may be set here.

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
* Diebold Express Poll 5000; version 2.1.1

[1]: https://twitter.com/TheEdgyDev/
[2]: https://twitter.com/tjhorner/
[3]: https://blog.horner.tj/post/hacking-voting-machines-def-con-25
[4]: https://greatscottgadgets.com/hackrf/
[5]: https://twitter.com/OrtegaAlfredo
