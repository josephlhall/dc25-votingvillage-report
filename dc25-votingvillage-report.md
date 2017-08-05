# Report from the DC25 Voting Machine Hacking Village

Author(s): [Matt Blaze][14], [Joseph Lorenzo Hall][1], [Harri Hursti][4]

***--DRAFT--THIS IS A DRAFT AND UNOFFICIAL UNTIL WE AGREE ON A RELEASE
   CANDIDATE--DRAFT--***

## Introduction

As a new "village" for DEF CON 25, Dark Tangent [announced][3] the
creation of a Voting Machine Hacking Village. In his words:

> CONCEPT: Get a bunch of voting machines and start hacking on them to
> raise awareness and find out for ourselves what the deal is.  I'm
> tired of reading misinformation about voting system security so it
> is time for a DEF CON Village.
>
> Until now getting access to real voting machines has been almost
> impossible.  The public has been assured by the vendors that the
> systems are safe, but who can verify that?  The DEF CON Voting
> Machine Hacking Village provides you access to real voting machines,
> used in past elections and to be used in future elections.  Now we,
> as community, can take a look ourselves and asses the security of
> these systems and help general public to get educated and the policy
> makers to get old-fashioned facts.
>
> As a first year Village we will get everyone started on
> understanding the technology and systems these machines live in.  By
> year three we hope to have a complete functioning stand alone voting
> network that we can test.  Believe it or not no such network has
> ever been security tested or audited - only separate pieces.

The Village was organized by [Jake Braun][7] (Cambridge Global), [Matt
Blaze][8] (U Penn), [Harri Hursti][9] (Nordic Innovation Labs), and
[Maggie MacAlpine][10] (Nordic Innovation Labs).

It was already a well established fact that voting machines were (and
are) hackable, but only a very limited number of people, mostly in
academic or industrial contexts, have been allowed to inspect the
machines before.  Past official studies -- such as the California
[Top-To-Bottom Review][5] and the Ohio [EVEREST Review][6], ten years
ago this summer -- had significant restrictions on what the
participating researchers were allowed to try -- restrictions which
obviously would not exist for criminals or nation-state attackers.  Of
course, those studies were also done in a "white box" environment --
researchers had access to source code, documentation, and equipment
under strict NDA -- while the DEF CON Voting Village is largely a
"black box" environment where hackers would need to create, copy, or
cobble together their own tools.  A common rebuttal to the discoveries
about the vulnerability of voting machines in the past has been that
the studies were allowed weeks or months, a length of time unrealistic
for an attacker to have with full access to the technology.  In
contrast, the DEF CON Voting Village had 25 hours over three days
(10:00-20:00 ET Fri-Sat (27-28 August 2017) and 10:00-15:00 ET Sun (29
August 2017)); this was a significant but not lengthy amount of time,
much more realistic in terms of access an attacker may have in some
jurisdictions where voting equipment may spend days in the home or
garage of a poll worker before Election Day.

The DEF CON 25 Voting Machine Hacking Village sought to change this:
we provided an environment (mostly) free from rules where anyone could
lay their technical hands and minds on examples of the machinery of
our democracy.  This technical report (and accompanying [technical
notes][11]) describe the goals of the Voting Village, the equipment
that was available in the room, and then findings from the actual work
hackers and security researchers performed during the 25 hours the
Voting Village was open.

## Limitations

There were significant limitations of the work at the Voting Village,
including:

* Participants had no access to source code, operational data or other
  proprietary information that is not otherwise legally available.  An
  actual evildoer might have little difficulty obtaining these
  materials.
* The Voting Village provided only an opportunistic sample of voting
  technologies.  Organizers obtained what they could get their hands on
  quickly and cheaply.  The most recently used system available was
  the AVS WinVote, which was decertified for use in 2014.  A number of
  other systems are still in use (AccuVote-TSx, Sequoia AVC Edge, ES&S
  iVotronic), but most if not all of these machines had ballot data
  from the 2008 general election, so the software is likely quite old.
* The Village had no access to optical scan or DRE systems with a
  VVPAT.  These systems -- and those involving ballot marking devices
  -- are the most software-independent and auditable systems, and are
  increasingly popular and heavily used.
* Finally, there was no access to any backend provisioning, counting,
  or voter registration systems.  These kinds of systems are not
  generally available on the open market.  Note: This is especially
  significant as the evidence from the 2016 election seems to indicate
  strongly that these types of voting technologies -- not voting
  machines themselves -- were the primary target of Russian GRU
  attacks.

## Goals

The goals of the DEF CON Voting Village were:

* Provide examples of working voting systems for security researchers
  to evaluate, attack, and otherwise study.
* Educate the DEF CON community and raise awareness about the
  machinery of US democracy, from the machines to how election
  technology interacts with legal, market, and normative barriers in
  elections that do not exist in general purpose computing contexts.
* Facilitate collaboration between security researchers with different
  skill sets to opportunistically assess software, hardware, and
  network security postures of these systems.
* Start a discussion in the DEF CON community about how security
  researchers and hackers can help to make our election infrastructure
  more safe and secure.

## Equipment in the Village

Village organizers worked hard to procure a variety of voting
equipment, including:

* AVS WinVote DRE -- software version 1.5.4 [hw version N/A]
* Premier AccuVote TSx DRE -- TS unit, model number AV-TSx, firmware 4.7.8
* ES&S iVotronic DRE -- ES&S Code IV 1.24.15.a, hardware revision 1.1
  * PEB version 1.7c-PEB-S
* Sequoia AVC Edge DRE -- version 5.0.24 [unsure if Edge I or II]
* Diebold Express Poll 5000 electronic pollbook -- version 2.1.1
* A Simulation of a "Back-Office" Election Environment

Note: A "[DRE voting machine][12]" is an abbreviation for
"direct-recording electronic" voting machine which designates a voting
system that records votes directly to some sort of digital storage or
memory, in contrast to optical scan systems that read the voter's
marks directly off of ballot paper. Also, an [electronic pollbook][13]
is a system that essentially replaces the spiral-bound lists of
registered voters in every polling place by putting that functionality
into a laptop, tablet, or kiosk-like computing platform.

## Brief Descriptions of Interesting Accomplishments

### AVS WinVote

### Premier AccuVote-TSx

### ES&S iVotronic

### Sequoia AVC Edge

### Diebold ExpressPoll 5000

### Back-Office Simulated Network

## Findings

DEF CON showed that technical minds with no previous knowledge about
voting machines, without even being provided proper tools, can still
learn how to hack the machines within tens of minutes or a few
hours. Part of this is because there were no restrictions or
prohibitions on, for example, disassembling the machines and there was
permission to take risks that may result in the machines being
destroyed in the process. This freedom to take such risks accelerates
the process and can lead to completely new discoveries of new
vulnerabilities.

### Vigorous, Diverse Attendance

The Voting Village expanded the number of people who have now had
first hand experience and knowledge of these systems. By Sunday, the
people who started hacking on Friday were the experts and they were
teaching and helping the new people who just started on Sunday.

In the village we had a number of technology savvy election officials,
who came to hack the very machines they use in their jurisdictions to
run elections.  This was their first opportunity to take a look
themselves into the machines -- machines they were required to use and
manage, but were prohibited to study in depth -- and find answers to
their own questions and learn more about that equipment.

### Discovery of Voter Data on ExpressPoll electronic pollbooks

As covered by the press (["Personal Info of 650,000 Voters Discovered
on Poll Machine Sold on Ebay"][15]), the ExpressPoll electronic
pollbooks that organizers obtained for the Voting Village were not
properly decommissioned, and they had live voter file data covering
654,517 voters from Shelby County, Tennessee circa 2008.  The
[database schema][17] (i.e, the list of elements in the database)
included the following data elements: `status` (voter registration
status), `dateOfBirth`, `precinctId`, `countyId`, `partyId`,
`language`, `idRequired`, `absentee`, `ssnLast4`, `driversLicense`,
`affidavitNumber` (serial number associated with voter registration
application), `houseNumber`, `streetName`, `apartmentNumber`, `city`,
`zip`, `nameFirst`, `nameMiddle`, `nameLast`, `middleInitial`,
`namePrefix`, and `nameSuffix`.  We were unable to verfify this data
extensively, although an initial examination seemed to indicate that
the `ssnLast4` field was not populated.  Full voter residential
addresses seemed valid, posing an uncertain risk to individuals that
require heightened privacy about their physical address, for example,
federal judges, law enforcement officers, victims of domestic violence
and other crimes such as stalking.  We initiated disclosure to Shelby
County, Tennessee although the news of the existence of these records
spread quickly.  We secured one copy of this data with one of the
Village organizers and destroyed the remaining copies and removed them
from the machines in the Village.

### AVS WinVote is a Complete Pwnage Platform

The AVS WinVote did not fare well against the hackers who examined it
during the Village.  Carsten Schürmann was able to remotely compromise
this system completely in about 10 minutes from across the room via WiFi, giving him
complete control over the machine and the data it holds.  This would
allow for simple disruption -- he could turn the machine off remotely
-- targeted voter privacy leakage -- he could mirror the WinVote
display remotely on his own machine -- and, most troublingly, vote
changing attacks -- he could read and write software to the WinVote
and change its vote database without detection.  This became even more
stark when a hacker, Nick, plugged a USB keyboard into the back of the
machine -- this is a proximate attack as opposed to Carsten's remote
attack.  Nick was able to hit the windows command `ctrl-alt-del`, get
access to the Windows CE Task Manager, install WinAMP, and [play a
full video and audio Rick roll][16].

## Next Year

We hope to expand the Voting Village next year to potentially cover a
number of distinct areas in addition to hands-on hacking of voting
equipment, including:

* **Closed-Loop System:** We would like to have a closed-loop system
  on which we can run an entire mock election using actual voting
  technologies. This would include voter registration, ballot
  generation, a mock polling place (with rules of engagement), and
  results reporting.
* **Election Tech Range:** Election officials and voting system
  manufacturers have some of their own security technologies,
  compositions, or solutions that they find work well in defending
  against certain threats.  We would like to invite election officials
  and voting system vendors to come and get advice and even testing of
  their tech. A good example would be if an election official or
  manufacturer would like to get feedback on a particular security
  system and/or challenge security researchers to evaluate it and give
  feedback on how it could be improved.
* **Election Tech Challenges:** There are a number of activities in
  elections that are difficult to secure -- yes, some small fraction
  of votes are cast by email, fax, and web and a larger fraction cast
  on paper through vote-by-mail.  We'd like to set up examples of
  these technologies and challenge Voting Village attendees to
  demonstrate what failures can happen and to what extent those can be
  avoided.
* **Election Technology Usable Security Evaluations:** A secure voting
  system can still be highly usable. We'd like to invite usable
  security researchers to join the village to build up a resource of
  usability and needs assessment conclusions and profiles of past,
  existing, and future voting technologies.

Please get in touch with us if you have other ideas!

[1]: https://josephhall.org/
[2]: https://github.com/rinon
[3]: https://forum.defcon.org/forum/defcon/dc25-official-unofficial-parties-social-gatherings-events-contests/dc25-villages/voting-machine-hacking-village/226138-new-for-def-con-25-voting-machine-hacking-village
[4]: https://nordicinnovationlabs.com/
[5]: http://www.sos.ca.gov/elections/voting-systems/oversight/top-bottom-review/
[6]: https://www.eac.gov/assets/1/28/EVEREST.pdf
[7]: https://www.cambridgeglobal.com/new-page
[8]: http://www.crypto.com/
[9]: https://nordicinnovationlabs.com/team/harri-hursti/
[10]: https://nordicinnovationlabs.com/team/margaret-macalpine/
[11]: https://github.com/josephlhall/dc25-votingvillage-report/blob/master/notes-from-folks-redact.md
[12]: https://en.wikipedia.org/wiki/DRE_voting_machine
[13]: https://en.wikipedia.org/wiki/Electronic_pollbook
[14]: http://www.crypto.com/
[15]: http://gizmodo.com/personal-info-of-650-000-voters-discovered-on-poll-mach-1797438462
[16]: http://www.nbcnews.com/tech/tech-news/hackers-were-able-breach-then-rick-roll-voting-machine-within-n788001
[17]: https://github.com/josephlhall/dc25-votingvillage-report/blob/master/resources/schema.sqlite3
