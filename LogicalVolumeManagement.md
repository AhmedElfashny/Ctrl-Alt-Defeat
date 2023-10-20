## LVM [Logical Volume Management]
is a storage management solution that provides a layer of abstraction  
between the operating system and the physical storage devices.  
LVM enables more flexible and dynamic allocation of disk space by  
allowing administrators to create logical volumes from one or more  
physical storage devices. This provides benefits such as easier volume  
resizing, snapshots, and improved management of storage resources.  
  
### Identify the Disks:
```bash
lsblk
```
```plaintext
NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0    7:0    0     4K  1 loop /snap/bare/5
loop1    7:1    0  63.4M  1 loop /snap/core20/1974
loop2    7:2    0  63.5M  1 loop /snap/core20/2015
loop3    7:3    0  73.9M  1 loop /snap/core22/858
loop4    7:4    0  73.9M  1 loop /snap/core22/864
loop5    7:5    0 237.2M  1 loop /snap/firefox/2987
loop6    7:6    0 349.7M  1 loop /snap/gnome-3-38-2004/143
loop7    7:7    0 485.5M  1 loop /snap/gnome-42-2204/126
loop8    7:8    0   497M  1 loop /snap/gnome-42-2204/141
loop9    7:9    0  91.7M  1 loop /snap/gtk-common-themes/1535
loop10   7:10   0  12.3M  1 loop /snap/snap-store/959
loop11   7:11   0  53.3M  1 loop /snap/snapd/19457
loop12   7:12   0  40.8M  1 loop /snap/snapd/20092
loop13   7:13   0   452K  1 loop /snap/snapd-desktop-integration/83
sda      8:0    0    30G  0 disk 
├─sda1   8:1    0     1M  0 part 
├─sda2   8:2    0   513M  0 part /boot/efi
└─sda3   8:3    0  29.5G  0 part /var/snap/firefox/common/host-hunspell
                                 /
sdb      8:16   0    10G  0 disk 
sdc      8:32   0    10G  0 disk 
sr0     11:0    1  1024M  0 rom  
```  

Here I have two disks, **sdb** and **sdc**.These two disks will be used   
to create a Volume Group to be utilized by LVM.

### Create Physical Volumes:  
Initialize each virtual disk as a physical volume for LVM  

```bash
pvcreate /dev/sdb
pvcreate /dev/sdc
```
"Create Physical Volumes" in the context of Logical Volume Management (LVM)   
refers to preparing your physical storage devices  
(usually hard disks or partitions) to be used within the LVM system.  
  
Here's what creating physical volumes means:

* **Initialization:** When you create physical volumes,   
you use the **pvcreate command** to initialize your physical storage devices.   
This command prepares the disk or partition for use with LVM.  
Essentially, it marks the device as a candidate for inclusion in an LVM setup.

* **Metadata:** The pvcreate command writes metadata to the device, which is  
used by LVM to track and manage the physical volume.This metadata includes  
information about the size of the physical volume and its location.

* **Integration into LVM:** After creating physical volumes, you can then add  
them to a Volume Group (VG). The VG acts as a pool that combines one or  
more physical volumes. This pooling allows you to manage and allocate  
storage space more flexibly and efficiently.

* **Dynamic Allocation:** Once a physical volume is part of a VG, you can allocate  
space from the VG to create logical volumes (LVs) that are used for storing  
data. LVs can be resized dynamically, and they are not tied to a specific  
physical disk, which offers greater flexibility in storage management.  
  
### Create a Volume Group:
Create a volume group that includes both of the physical volumes

```bash
vgcreate myvg /dev/sdb /dev/sdc
```  
### Create Logical Volumes:
With the volume group established, you can create logical volumes  
```bash
lvcreate -n mylv1 -L 10G myvg
```  
### Format and Mount:
Format the logical volumes with the desired file system (e.g., ext4) and mount  
them to the desired location

```bash
mkfs.ext4 /dev/myvg/mylv1
mkdir /mnt/mylv1
mount /dev/myvg/mylv1 /mnt/mylv1
```

### Automount on Boot:
To ensure that your logical volumes are mounted at boot, add entries to  
the /etc/fstab file  

Here You Go The Output After Creating Group Volume to the Two Disks,  
Check **sdb** and **sdc**
  
```
AME         MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0          7:0    0     4K  1 loop /snap/bare/5
loop1          7:1    0  63.4M  1 loop /snap/core20/1974
loop2          7:2    0  63.5M  1 loop /snap/core20/2015
loop3          7:3    0  73.9M  1 loop /snap/core22/858
loop4          7:4    0  73.9M  1 loop /snap/core22/864
loop5          7:5    0 237.2M  1 loop /snap/firefox/2987
loop6          7:6    0 349.7M  1 loop /snap/gnome-3-38-2004/143
loop7          7:7    0 485.5M  1 loop /snap/gnome-42-2204/126
loop8          7:8    0   497M  1 loop /snap/gnome-42-2204/141
loop9          7:9    0  91.7M  1 loop /snap/gtk-common-themes/1535
loop10         7:10   0  12.3M  1 loop /snap/snap-store/959
loop11         7:11   0  53.3M  1 loop /snap/snapd/19457
loop12         7:12   0  40.8M  1 loop /snap/snapd/20092
loop13         7:13   0   452K  1 loop /snap/snapd-desktop-integration/83
sda            8:0    0    30G  0 disk 
├─sda1         8:1    0     1M  0 part 
├─sda2         8:2    0   513M  0 part /boot/efi
└─sda3         8:3    0  29.5G  0 part /var/snap/firefox/common/host-hunspell
                                       /
sdb            8:16   0    10G  0 disk 
└─myvg-mylv1 253:0    0    10G  0 lvm  /mnt/mylv1
sdc            8:32   0    10G  0 disk 
└─myvg-mylv1 253:0    0    10G  0 lvm  /mnt/mylv1
sr0           11:0    1  1024M  0 rom  

```
### Manage and Monitor:
To check the size of your logical volume
```bash
lvdisplay /dev/myvg/mylv1
```  
```plaintext
  --- Logical volume ---
  LV Path                /dev/myvg/mylv1
  LV Name                mylv1
  VG Name                myvg
  LV UUID                S0LZUe-AvRV-L70j-feKz-scRC-mSYF-L2p27P
  LV Write Access        read/write
  LV Creation host, time ahmed-VirtualBox, 2023-10-20 14:56:27 +0300
  LV Status              available
  # open                 1
  LV Size                10.00 GiB
  Current LE             2560
  Segments               2
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0

```

To increase the size of your logical volume (LV), you can follow these steps.  
* Check Available Space:
```bash
vgdisplay myvg
```  
* Resize the Logical Volume:  
For example, to increase the size of mylv1 by 5 gigabytes  
After resizing the LV, you need to extend the file system within it   
to use the newly allocated space
```bash
lvresize -L +5G /dev/myvg/mylv1
resize2fs /dev/myvg/mylv1
lvdisplay /dev/myvg/mylv1
```  

### Increasing the Volume
To increase the volume of a logical volume (LV) in a Logical Volume   
Management (LVM) setup **using a new hard disk**  
  
For Example: I want to add sdd to my LVM  
```plaintext
NAME         MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
loop0          7:0    0     4K  1 loop /snap/bare/5
loop1          7:1    0  63.4M  1 loop /snap/core20/1974
loop2          7:2    0  63.5M  1 loop /snap/core20/2015
loop3          7:3    0  73.9M  1 loop /snap/core22/858
loop4          7:4    0  73.9M  1 loop /snap/core22/864
loop5          7:5    0 237.2M  1 loop /snap/firefox/2987
loop6          7:6    0 349.7M  1 loop /snap/gnome-3-38-2004/143
loop7          7:7    0 485.5M  1 loop /snap/gnome-42-2204/126
loop8          7:8    0   497M  1 loop /snap/gnome-42-2204/141
loop9          7:9    0  91.7M  1 loop /snap/gtk-common-themes/1535
loop10         7:10   0  12.3M  1 loop /snap/snap-store/959
loop11         7:11   0  53.3M  1 loop /snap/snapd/19457
loop12         7:12   0  40.8M  1 loop /snap/snapd/20092
loop13         7:13   0   452K  1 loop /snap/snapd-desktop-integration/83
sda            8:0    0    30G  0 disk 
├─sda1         8:1    0     1M  0 part 
├─sda2         8:2    0   513M  0 part /boot/efi
└─sda3         8:3    0  29.5G  0 part /var/snap/firefox/common/host-hunspell
                                       /
sdb            8:16   0    10G  0 disk 
└─myvg-mylv1 253:0    0    10G  0 lvm  
sdc            8:32   0    10G  0 disk 
└─myvg-mylv1 253:0    0    10G  0 lvm  
sdd            8:48   0    20G  0 disk 
sr0           11:0    1  1024M  0 rom  

```  
* Add a New Physical Volume (PV)
```bash
pvcreate /dev/sdd
```
* Extend the Volume Group (VG):
```bash
vgextend myvg /dev/sdd
```

### Removing the Volume
* Unmount the Logical Volume:
```bash
umount /dev/myvg/mylv1
```  
* Remove the Logical Volume:
```bash
lvremove /dev/myvg/mylv1
```
* Remove Volume Group:
```bash
vgremove MyVolume
```












