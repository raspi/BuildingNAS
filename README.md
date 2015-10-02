# Building NAS for home use with ZFS
**(under construction)**

First off: **RAID IS NOT A BACKUP!**

How to build your own NAS for home use which doesn't break immediately? What type of hazards are there? 

# Setup
* No hardware RAID
* ZFS as filesystem
* FreeBSD as operating system
* RaidZ2 (aka RAID6) (2 drives can fail) or RaidZ3 (3 drives can fail)
* ECC RAM used to eliminate point of failure and finding memory errors  
* Redundant power supply
* Multiple SAS controllers against data loss
* Rack mount case with 24 x 3.5" bays
* UPSes for PSUs
* OS boot drive is seperated
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
* CPU(s) supporting ECC RAM
* 3 (or more) x SAS expander cards (1 x MiniSAS port for backplane which has 4 slots for SAS/SATA HDDs)
  * RAID6 can lose 2 HDDs
  * Each drive are in different zpool per backplane so there are 4 different zpools or vdevs
  * If one controller or expander card breaks then 2 backplanes are lost which is recoverable, as each zpool or vdev will lose 2 drives at once (8 drives total)
* PSUs connected to two different UPSes against power loss
* Extra spare drive(s) against HDD failure
* SSD drive or USB stick for OS

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
* I/O performance check
* Remove dust from drive bays, CPU heatsink(s), motherboard heatsink(s), PSU(s) and cards

# Non-regular maintenance
* RAM check
* Change PSU(s)
* Update BIOS
* Update HW firmware
* Change 3-4+ year old hard drives to new ones
* Change UPS battery

# Why ECC memory?
Your data needs to be written only once to storage as broken and then it will be broken forever. ECC memory protects against this type of corruption. RAM component(s) running too hot will also cause data corruption.

# Why hardware RAID is bad?
* Hardware RAID doesn't protect against bit rot aka silent corruption
  * Over the years disks will return wrong data
    * In ZFS when data is written to disk, also a checksum is written. ZFS scrub checks the data against the checksum and will know if file content is broken. ZFS will correct this problem silently without the need for any additional tools. Same silent fixing will happen if user just reads data and scrub isn't running. This is why ECC RAM is so important.
* Raw passthrough to drives is often limited or prohibited completely or you have to use card manufacturer's own tools 
  * SMART data is very useful for getting information of early failure
  * All SCSI commands should pass so that OS can provide you with all information in failure cases
* Some RAID levels use propietary algorithms (usually RAID6 and some others too)
  * Example: Card dies and card manufacturer has gone bankrupt; you can't access the data with other manufacturer's RAID card because the algorithm used is different
* Information written to disks is propietary
  * You can't access the disk with other manufacturer's card
* Information written to disks is location aware in some cards
  * Example: moving drive from bay #1 to #2 and vice versa, then card may think that everything is broken
* After changing to new drive after failure the whole disk is written full of data and rebuilding may take days to complete
  * ZFS equivalent, resilvering, only writes necessary used data to new drive and is much faster
* Hard to add new drives
* Configuring array might need booting to card's own configuration utility
* If power loss occurs when using for example RAID 5 and card's battery has died (or even worse: there's no battery) whole array will go out of sync and may not be recoverable or might need full rebuild which again may take many days

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
  * Example: cheap "new" LSI 9211-8i HBA's from China on ebay aren't usually genuine hardware and may have issues
  * Serial number might not be valid, so ask for serial number to confirm genuine product from manufacturer
* Wrong kind of product
  * SAS breakout cables used for drives and controller looks alike but are different
     * **Forward** breakout cable for connecting SATA connectors to **drives**
     * **Reverse** breakout cable for connecting SATA connectors to **SAS controller**
  * Incompatible motherboard for enclosure  
  * Incompatible PSU for enclosure 
* Check options
  * Check for bracket option(s) for bigger fan(s)      
* Check space 
  * Unable to close enclosure because heatsink or fan is too big    

## SAS Controllers
* Some controllers may be limiting hard drive sizes
  * Example: LSI 1068E limits drive size to 2 TB
* Some controllers may be limiting total size beeing seen of connected drives to for example 256 TB max
* Some controllers may not work on higher speed PCI-e or other slot
* Some controllers may not work on lower speed PCI-e or other slot
* Some controllers may not work if the motherboard is not from same manufacturer
  * Example: Dell H310 may need certain pins on PCI-e slot to be blocked
* Some controllers may not support hot-swap
* Some controller may be incompatible with operating system or operating system's version
* Some controller's firmware may be incompatible with operating system or operating system's version
* Cable may be faulty
* Cable going to expander or backplane might be wrong type
* Controller may be overheating

## SAS Expanders
* Some expanders may be limiting hard drive sizes to for example 2 TB max
* Some expanders may be limiting total size beeing seen of connected drives to for example 256 TB max
* Some expanders may not work on higher speed PCI-e or other slot
* Some expanders may not work on lower speed PCI-e or other slot
* Some expanders may not work on non-manufacturer motherboard
* Some expanders may need controller from the same manufacturer for you to be able to update the expander card's firmware
* Some expanders may not support hot-swap 
* Some expanders may not support double bandwidth (two cables from expander to controller) if the controller's manufacturer is not the same as expanders
* Cable may be faulty
* Cable going to backplane or controller might be wrong type
* Expander may be overheating

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
* Drives are connected to backplane so that controller or expander failure can corrupt the whole pool
* Drives that have been spinning for years may not be able to start spinning again after spin down
* Drive may be overheating
* Rarely brand new drive might die after just a few days or hours or not work at all
* Some drives may have different sector counts
  * Example: replacing failed 2 TB drive with new 2 TB drive from different manufacturer fails because it has less sectors than rest of the drives    

## Operating system
* OS may have driver bug for SAS controller
* OS may have driver bug for SAS controller's firmware
* ZFS implementation might have a bug
 
## Motherboard
* Motherboard may have IRQ conflict issue that hangs computer or slows it down
 
## Power Supply Unit (PSU)
* PSUs will lose power capacity over time
* PSU close to full capacity will degrade faster 

## Uninterruptible Power Supply (UPS)
* UPS close to full capacity will wear out battery/batteries and UPS' own PSU faster 

## Fan noise
* Rackmount enclosures are usually shipped with "turbine" fans
* Some enclosures may have temperature sensors built-in where fans are plugged in 

## Enclosure
* Space between CPU and enclosure might be too small for heatsink or fan 
  * Example: Noctua NH-D14 SE2011 is too big for 4U enclosure 
 
# RAID levels
## RAID 0
The zero stands for how many files you will recover if any of the drives fails. Do not use.
## Mirror aka RAID 1
Two or more disks will have exact copy of one disk. Recommended.
## RAIDz aka RAID 5
Do not use raidz aka RAID5. It is just not worth it. After one drive fails and new is inserted it is quite common to see another drive break during resilvering (rebuilding). If you calculate amount of hard drive cost versus time spent used trying to fix and probably lose whole raidz pool the extra drive's cost is very quickly paid back. Also include the time it takes to transfer the files from backup to new pool.
## RAIDz2 aka RAID 6
Two drives can fail. Recommended.

# Other notes
## Samba aka CIFS aka Windows share
* It is adviced to use `recycle` VFS object in the config file as it will move deleted files to other directory and thus prevents accidental file deletions
  * Remember to test it!
* You can block some files by using `veto files` option in the samba config file
  * Example: Windows `Thumbs.db` files 

## Network card and bandwidth
* If you want more than 1 Gbps speed with multiple 1 Gbps cards for example between NAS - network switch - your PC then your NAS, switch and PC must support link aggregation (LACP)
  * Depending on the NIC and OS used the NICs usually have to be same cards from same manufacturer

# Links
* http://www.solarisinternals.com/wiki/index.php/ZFS_Best_Practices_Guide
* http://arstechnica.com/information-technology/2014/01/bitrot-and-atomic-cows-inside-next-gen-filesystems/
* https://static.googleusercontent.com/media/research.google.com/fi//archive/disk_failures.pdf
* https://en.wikipedia.org/wiki/Data_degradation
* https://www.youtube.com/watch?v=yAuEgepZG_8
* http://www.zdnet.com/article/why-raid-5-stops-working-in-2009/
