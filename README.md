# Building NAS with ZFS
**(under construction)**
First off: **RAID IS NOT A BACKUP!**

How to build NAS? What type of hazards are there? 
Do not use raidz aka RAID5. It is just not worth it. After one drive fails and new is inserted it is quite common to see another drive break during resilvering (rebuilding). If you calculate amount of hard drive cost versus time spent used trying to fix and probably lose whole RAID5 pool the extra drive's cost is very quickly paid back.

# Setup
* No hardware RAID
* ZFS as filesystem
* RaidZ2 (aka RAID6) (2 drives can fail) or RaidZ3 (3 drives can fail)
* ECC memory only
* Redundant power
* Multiple SAS/SATA expanders against data loss
* Rack mount case with 24 x 5.25" bays
* UPSes
* OS hard drive is seperated 
 
# Outside hazards
* Thunderstorm
* Power loss
* Fire
* Room too hot
* Room too cold

# Hardware hazards
* PSU failure
* Motherboard failure
* SAS/SATA expander card failure
* SAS/SATA controller card failure
* Backplane failure
* Hard drive failure
* Not knowing about harware what is used
 
# Software hazards
* Viruses
* Malware
* No backup
* Accidental file deletion
* Accidental system commands on wrong computer
* Not knowing about software what is used
* Adding too many drives to one zpool
 
# Hardware setup
* Motherboard
* ECC RAM
* 3 (or more) x SAS expander cards (1 x MiniSAS port for backplane which has 4 slots for SAS/SATA HDDs)
  * RAID6 can lose 2 HDDs
  * Each drive are in different zpool per backplane so there are 4 different zpools
  * If one expander card breaks then 2 backplanes are lost which is recoverable, as each zpools will lose 2 drives at once (8 drives total)
* PSUs connected to two different UPSes against power loss
* Extra spare drive(s) for HDD failure

# What to monitor
* HDD SMART data
* HDD temperatures
* Motherboard temperature
* CPU(s) temperature
* UPS battery
* UPS temperature

# Regular maintenance
* ZFS scrub
* SMART tests
* Backup checking 
  * 
  * Restore process
* RAM check
* I/O performance check

# Why ECC memory?
Your data needs to be written only once to storage as broken and then it will be broken forever. ECC memory protects against this type of corruption.

# Links
* http://arstechnica.com/information-technology/2014/01/bitrot-and-atomic-cows-inside-next-gen-filesystems/
* https://static.googleusercontent.com/media/research.google.com/fi//archive/disk_failures.pdf
* https://en.wikipedia.org/wiki/Data_degradation
* https://www.youtube.com/watch?v=yAuEgepZG_8
