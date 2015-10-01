# Building NAS for home use with ZFS
**(under construction)**

First off: **RAID IS NOT A BACKUP!**

How to build your own NAS for home use which doesn't break immediately? What type of hazards are there? 

# Setup
* No hardware RAID
* ZFS as filesystem
* FreeBSD as operating system
* RaidZ2 (aka RAID6) (2 drives can fail) or RaidZ3 (3 drives can fail)
* ECC memory only
* Redundant power supply
* Multiple SAS controllers against data loss
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
* Cheap components
* Not using ECC memory
* PSU failure
* Motherboard failure
* Cheap SAS cables
* SAS/SATA expander card failure
* SAS/SATA controller card failure
* Backplane failure
* Hard drive failure
* Not knowing about harware what is used
* Dust 
* Moisture
 
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
* ZFS pool status
* ZFS scrub status
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
* Change 3-4+ year old hard drives to new ones
* Change UPS battery

# Why ECC memory?
Your data needs to be written only once to storage as broken and then it will be broken forever. ECC memory protects against this type of corruption.

# Hard drives
* Hard disks will break sooner if temperature changes too much
* SAS drives are better for retrying failed data access; SATA drives usually freezes the whole system

# Why you must use UPS (Uninterruptible Power Supply)
All modern hard drives contains caching. Almost all hard drive manufacturers build hard drives so that it lies to the operating system that data is written. So operating system and filesystem will mark data as written but in actuality it is still in writing process to the hard drive from cache. If power loss occurs at this writing time the data is lost. UPS gives hard drive time to finish writing this data to the disk. In some drives you can disable caching. This makes the disk very slow to write to. Also depending on your SAS/SATA controller, expander and backplane it is possible that some of these components blocks sending all instructions to hard drives. Most common one is that SMART data is blocked. So check that all your components can send and receive raw hard drive data before buying components.

# Good to have
* IPMI or other remote support in motherboard so that you can configure BIOS/UEFI remotely and mount ISO images from local computer
 
# What to check for possible problems

## Buying hardware
* Product is not genuine
  * Example: cheap "new" LSI 9211-8i HBA's from china in ebay aren't usually genuine hardware and won't work
  * Serial number might not be valid, so ask for serial number to confirm genuine product from manufacturer

## SAS Controllers
* Some controllers may be limiting hard drive sizes to for example 2 TB max
  * Example: LSI 1068E limits drive size to 2 TB
* Some controllers may be limiting total size beeing seen of connected drives to for example 256 TB max
* Some controllers may not work on higher speed PCI-e or other slot
* Some controllers may not work on lower speed PCI-e or other slot
* Some controllers may not work if the motherboard is not from same manufacturer
  * Example: Dell H310 may need certain pins on PCI-e slot to be blocked
* Some controllers may not support hot-swap
* Some controller may be incompatible with operating system or operating system's version
* Some controller's firmware may be incompatible with operating system or operating system's version
* Cable might be faulty
* Cable going to expander or backplane might be wrong type

## SAS Expanders
* Some expanders may be limiting hard drive sizes to for example 2 TB max
* Some expanders may be limiting total size beeing seen of connected drives to for example 256 TB max
* Some expanders may not work on higher speed PCI-e or other slot
* Some expanders may not work on lower speed PCI-e or other slot
* Some expanders may not work on non-manufacturer motherboard
* Some expanders may need controller from the same manufacturer for you to be able to update the expander card's firmware
* Some expanders may not support hot-swap 
* Some expanders may not support double bandwidth (two cables from expander to controller) if the controller's manufacturer is not the same as expanders
* Cable might be faulty
* Cable going to backplane or controller might be wrong type

## Backplanes
* Backplane may not receive enough power from PSU's power rail
* Some backplanes may have SAS expander(s) built-in
* Some backplanes may not support SATA hard drives, only SAS drives
* Some backplanes may not support hot-swap
* Cable might be faulty
* Cable going to SAS Expander or controller might be wrong type

## Hard drives
* Some drives may have faulty firmware
  * Example: Samsung HD155UI and HD204UI drive writes corrupted data to the disk if SMART data is being read at the same time
* Drives are connected to backplane so that controller failure can corrupt the whole pool

## Operating system
* OS may have driver bug for SAS controller
* OS may have driver bug for SAS controller's firmware
* ZFS implementation might have a bug
 
## Motherboard
* Motherboard might have IRQ conflict issue that hangs computer or slows it down

# RAID levels
## RAID 0
The zero stands for how many files you will recover if any of the drives fails. Do not use.
## Mirror aka RAID 1
Two or more disks will have exact copy of one disk. Recommended.
## RAIDz aka RAID 5
Do not use raidz aka RAID5. It is just not worth it. After one drive fails and new is inserted it is quite common to see another drive break during resilvering (rebuilding). If you calculate amount of hard drive cost versus time spent used trying to fix and probably lose whole RAID5 pool the extra drive's cost is very quickly paid back. 
## RAIDz2 aka RAID 6
Two drives can fail. Recommended.

# Links
* http://www.solarisinternals.com/wiki/index.php/ZFS_Best_Practices_Guide
* http://arstechnica.com/information-technology/2014/01/bitrot-and-atomic-cows-inside-next-gen-filesystems/
* https://static.googleusercontent.com/media/research.google.com/fi//archive/disk_failures.pdf
* https://en.wikipedia.org/wiki/Data_degradation
* https://www.youtube.com/watch?v=yAuEgepZG_8
* http://www.zdnet.com/article/why-raid-5-stops-working-in-2009/
