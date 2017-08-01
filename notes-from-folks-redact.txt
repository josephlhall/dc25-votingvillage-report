# Work on the ExpressPoll 5000

Sean Roach (@TheEdgeyDev) & TJ Horner (@tjhorner) & Ian Smith

Found a few physical things and a few software things. Main physical thing, sqlite 3 databae on compactflash in the clear. The only thing stopping someone from taking it out is one screw, philips head. This would have allowed us to be able to do a DOS, by removing the card. take a schema of that db, make it without any users. the put it back in with no users. everyon afterwards would not be registereed in that county or precinct, go to another polling staton, 30 minute line would grow precipitously. selectively DOS counties or precicnts that are for your . username and password released on a PDF by state of maryland. plug in a blank drive because we didn't have a database. missing hte swllite file. missing file... create file... missing table.. create table. username as 1 password as 1111. consolidationg ID we utrnined into defdcon. gave us access into  trying use bashbunny, USB thing that emulates a keyboard. sending A's at it to overflow the the text field... Windows CE 5 ... .Net app... trying to get it to fill ram and the ram manager will close that app. they were able to read the fields. realized wireshark to see what ports 80, 443, 5002 (sounds like an error broadcast, doesn't accept any connectinons). 443 closes the connection.

firware upgrades. looks for EBOOT.bin NK.bin... bootloder update, NK is kernal. will find these files download into RAM and the flash the bootloader. No signature.

Smartcard reader. What if we can make it so that the card... what if you register everything is fine, but write an invalid id to the card. this would allow casting a vote, but would not be counted on the backend.

# AVS WinVote

Nick ([redacted])

We were wokring on the winvote, getting into it, 2 USB ports on the back. Just had to attached a keyboard, ctrl-al-del, task manager... Alt-f run... takes arbitrary usb drive contents. WiFi chipset. WEP 104/40 not even 128. Physical access to back of the machine for 15s can do anything. Open ports, MS08067 - Net API exploit vulnerability. Wasn't able to pop a shell, but it seemed to work but hung the machine. WEP keys are hard-coded. Modem RJ11 jack, will communicate externally. Looking at the file format ... there was a file system but no files. But open in a hex editor and you could see that the files are still there and not securely erased. 3-pin key over the guts.

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

# AVS WinVote & ExpressPoll

Alfredo Ortega (@OrtegaAlfredo)

RF survery, winvote printer. Emitting a bunch of RF. Were not able to correlate with specific action. Some very low signal. HackRF -- software defined radio.

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
