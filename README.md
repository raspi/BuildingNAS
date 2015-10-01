# Building NAS for home use with ZFS
**(under construction)**

First off: **RAID IS NOT A BACKUP!**

How to build your own NAS for home use which doesn't break immediately? What type of hazards are there? 
Do not use raidz aka RAID5. It is just not worth it. After one drive fails and new is inserted it is quite common to see another drive break during resilvering (rebuilding). If you calculate amount of hard drive cost versus time spent used trying to fix and probably lose whole RAID5 pool the extra drive's cost is very quickly paid back. Also do not use cheap components, especially PSU. Never use non-ECC RAM.

# Setup
* No hardware RAID
* ZFS as filesystem
* RaidZ2 (aka RAID6) (2 drives can fail) or RaidZ3 (3 drives can fail)
* ECC memory only
* Redundant power supply
* Multiple SAS/SATA expanders against data loss
* Rack mount case with 24 x 5.25" bays
* UPSes for PSUs
* OS hard drive is seperated
* Good quality components
 
# Outside hazards
* Thunderstorm
* Power loss
* Fire
* Water
* Humidity
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
* Dust 
 
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
  * Restore process
* RAM check
* I/O performance check
* Remove dust from drive bays, CPU heatsink(s), motherboard heatsink(s), PSU(s) and cards

# Non-regular maintenance
* Change PSU(s)
* Update BIOS
* Update HW firmware
* Change 3-4 year old hard drives to new ones

# Why ECC memory?
Your data needs to be written only once to storage as broken and then it will be broken forever. ECC memory protects against this type of corruption.

# Hard drives
* Hard disks will break sooner if temperature changes too much
* SAS drives are better for retrying failed data access; SATA drives usually freezes the whole system

# Why you must use UPS (Uninterruptible Power Supply)
All modern hard drives contains caching. Almost all hard drive manufacturers build hard drives so that it lies to the operating system that data is written. So operating system and filesystem will mark data as written but in actuality it is still in writing process to the hard drive from cache. If power loss occurs at this writing time the data is lost. UPS gives hard drive time to finish writing this data to the disk. In some drives you can disable caching. This makes the disk very slow to write to. Also depending on your SAS/SATA controller, expander and backplane it is possible that some of these components blocks sending all instructions to hard drives. Most common one is that SMART data is blocked. So check that all your components can send and receive raw hard drive data before buying components.

# Problems

## SAS Controllers
* Some controllers may be limiting hard drive sizes to for example 2 TB max
  * Example: LSI 1068E limits drive size to 2 TB
* Some controllers may be limiting total size beeing seen of connected drives to for example 256 TB max
* Some controllers may not work on higher speed PCI-e or other slot
* Some controllers may not work on lower speed PCI-e or other slot
* Some controllers might 

## SAS Expanders
* Some expanders may be limiting hard drive sizes to for example 2 TB max
* Some expanders may be limiting total size beeing seen of connected drives to for example 256 TB max
* Some expanders may not work on higher speed PCI-e or other slot
* Some expanders may not work on lower speed PCI-e or other slot

## Hard drives
* Some drives may have faulty firmware
  * Example: Samsung HD155UI and HD204UI drive writes corrupted data to the disk if SMART data is being read at the same time

# Links
* http://www.solarisinternals.com/wiki/index.php/ZFS_Best_Practices_Guide
* http://arstechnica.com/information-technology/2014/01/bitrot-and-atomic-cows-inside-next-gen-filesystems/
* https://static.googleusercontent.com/media/research.google.com/fi//archive/disk_failures.pdf
* https://en.wikipedia.org/wiki/Data_degradation
* https://www.youtube.com/watch?v=yAuEgepZG_8
