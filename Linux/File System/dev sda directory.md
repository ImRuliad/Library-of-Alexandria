## ***Table of Contents***





## ***Introduction***
---
*/dev/sda is the hard disk of the computer we are using.*
*It is a block device that has a directory in the /dev directory*





## ***/dev***
---
*/dev is a directory in the root folder that contains all device files*

- Linux creates these files during installation
- These must be available during the boot process
- *Can* list devices in this directory using `ls -l` but a better way is by using the devices Major and Minor numbers
- disk devices have a Major number of 8, which assigns them as SCSI block devices
- SCSI subsystem manages all PATA and SATA Hard Drives
- `sd[a-z]` is the most used way to refer to hard disks
- SCSI is a microprocessor controlled smart bus that allows up to 15 peripheral devices to the computer
- Includes hard drives, scanners, USB, printers, etc...

## ***/dev/sda***
---
*/dev/sda is a block device in the /dev directory*

Running `ls -l /dev | grep "sda"` gives us
```
ls -l /dev | grep "sda"
brw-rw----  1 root disk        8,   0  Apr 29 22:33 sda
brw-rw----  1 root disk        8,   1  Apr 29 22:33 sda1
brw-rw----  1 root disk        8,   2  Apr 29 22:33 sda2
brw-rw----  1 root disk        8,   3  Apr 29 22:33 sda3
brw-rw----  1 root disk        8,   4  Apr 29 22:33 sda4
brw-rw----  1 root disk        8,   5  Apr 29 22:33 sda5
brw-rw----  1 root disk        8,   6  Apr 29 22:33 sda6
```

- During installation, Linux takes the first hard disk found and assigns it the value `sda`
- subsequent disks found are appended to this `sda[1-15]`
- If multiple partitions in the hard disk, system consecutively appends a number starting from `sda1` to `sda15`
- Can have a max of 15 partitions

## ***More Info About /dev/sda***
---

#### **fdisk command**
*shows us the capacity of our hard disks, the partitions within it, and their respective sizes.*
```
sudo fdisk -l
Disk /dev/sda: 465.76 GiB, 500107862016 bytes, 976773168 sectors
Device     Boot     Start       End   Sectors   Size Id Type
/dev/sda1  *         2048    104447    102400    50M  7 HPFS/NTFS/exFAT
/dev/sda2          104448 484152681 484048234 230.8G  7 HPFS/NTFS/exFAT
/dev/sda3       484155392 485249023   1093632   534M 27 Hidden NTFS WinRE
/dev/sda4       485251070 976771071 491520002 234.4G  5 Extended
/dev/sda5       485251072 974772223 489521152 233.4G 83 Linux
/dev/sda6       974774272 976771071   1996800   975M 82 Linux swap / Solaris
```

#### **lsblk**
*Shows us available block devices connected to the system, except RAM. Displays output in a tree-like structure.*
```
lsblk
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda      8:0    0 465.8G  0 disk 
├─sda1   8:1    0    50M  0 part 
├─sda2   8:2    0 230.8G  0 part 
├─sda3   8:3    0   534M  0 part 
├─sda4   8:4    0     1K  0 part 
├─sda5   8:5    0 233.4G  0 part /
└─sda6   8:6    0   975M  0 part [SWAP]
sr0     11:0    1  1024M  0 rom  
```
