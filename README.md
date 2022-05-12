# archlinux_installing along side windows 10

# Download iso file

# create a bootable usb disk
check for the usb
```
sudo fdisk -l

output
Disk /dev/sdb: 14.59 GiB, 15664676864 bytes, 30595072 sectors
Disk model: Cruzer Blade    
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x46a6eddb

Device     Boot Start      End  Sectors  Size Id Type
/dev/sdb1        2048 30595071 30593024 14.6G  b W95 FAT32
```

write the iso to usb

```
sudo dd bs=4M if=archlinux-2022.05.01-x86_64.iso of=/dev/sdb1 status=progress oflag=sync
```

# Prepare Your Computer for Installing Arch Linux

https://www.freecodecamp.org/news/how-to-install-arch-linux/

## 1. disabling secure boot
> The first change that you'll have to make is disabling secure boot in your UEFI configuration. This feature helps prevent malware attacks during boot but it also prevents the Arch Linux installer from booting.

## 2. Turn off fast startup
> The second thing that you should disable is only relevant if you're installing Arch Linux alongside Windows. There is a Windows feature called fast startup that reduces the boot time of your computer by partially hibernating it.

> This is generally a nice feature to have but it prevents any other operating system in a dual boot configuration from accessing the hard disk in the process.

```
--> startMenu 
--> type "choose a power plan" 
--> what the power buttons do (left side) 
--> "Change settings that are currently unavailable" at the top 
--> Untick the "Turn on fast startup (recommended)" option and press the "Save changes" button at the bottom. 
From now on the boot process may take a few moments extra but it's all worth it.
```

# check hard disk on windows

## Check Hard Disk Total Size by System Information

- Step 1: Press Windows Key + R to open a run window. Then type msinfo32 and press Enter.
- Step 2: A new window will pop up. On the left panel, click the plus sign on the left of Components to open collapse menu.
- Step 3: Click the plus sign on the left of Storage and then click Disks. Finally, the detailed specs of hard disk will be shown on the right panel. Here you can check the total size of hard drive.

> Notes: You might be confused about the total space of your hard disk that you found shows less space than advertised. For example, the advertised space of my SSD is 250GB but the result I found is 233.83GB. The reason for this difference lies in the different computing ways between the hard drive manufacturers calculate and advertise and the computers actually use. The hard disk manufacturers use decimal bytes, so, 1GB = 1,000MB = 1,000,000KB = 1,000,000,000 bytes. It has been an industry standard in advertising storage space. If the hard disk manufacturer advertises the capacity of a hard drive is 250GB, it contains 250 * 1000 * 1000 * 1000 = 250,000,000,000 bytes. However, computers use the binary bytes, so the computers convert the number of bytes into gigabytes by dividing by 1024 rather than 1,000.So, 250,000,000,000 bytes = 250,000,000,000 / (1,024 * 1,024 * 1,024) = 233.83GB. That's why the total hard disk space shows less than actual.


## 2. Check If You Have an SSD or HDD Windows 10 with Disk Defragmenter

```
Step 1. You can press Windows + R, type dfrgui, and press Enter to open Disk Defragmenter tool.
Step 2. In Media type column, you can find out if your hard drive is solid state drive or hard disk drive.
```

## 3. disk management

- 1. The easiest way to open Disk Management in Windows 10 is from computer Desktop. Right click on Start Menu (or press Windows+X hotkey) and then select "Disk Management".
- 2. Use Windows+R hotkey to open Run window. Then type "Diskmgmt.msc" and click "OK" or hit "Enter" key.

We will see how many disks are there. 

In a windows we have
- efi (250 MB)  (we can use this no need to create another efi partition)
- system recovery (for windows 500-800 MB)
- Windows OS (500GB)

# Size distribution
- windows os 150GB
- linux / - 75GB
- linux /home - 25GB
- NTFS partition (accessed by both os's) - 225 GB

# why to have swap partition
> Swap space can take the form of a disk partition or a file. Users may create a swap space during installation or at any later time as desired. 

Swap space can be used for two purposes, 
- to extend the virtual memory beyond the installed physical memory (RAM), 
- and also for suspend-to-disk support.

# power off and the power on and press F12
- it shows boot menu
- - select the USB

https://gist.github.com/moral-g/3b02fffc32b114bdacf92f279ccb3479

# Verify the boot mode
```
ls /sys/firmware/efi/efivars
```
If the command shows the directory without error, then the system is booted in UEFI mode.

# Connect to the internet

## Ensure your network interface is listed
```
ip link
```

## iwctl steps for WIFI:

```
iwctl
[iwd] device list
# This will list devices, replace wnated on with FOO for steps below
[iwd] station FOO scan
[iwd] station FOO get-networks
[iwd] station FOO connect SSID

example:
[iwd] station wlan0 connect Xiaomi_AAF3_EXT
Passphrase: 
```

or:

If SSID and passphrase is known
```
# PASS = passphrase ; FOO = device ; SSID = ssid
iwctl --passphrase PASS station FOO connect SSID
```

## Verify connection
```
ping google.com
```

## Ensure the system clock is accurate
timedatectl set-ntp true

## Check the service status
timedatectl status

## Create partitions cfdisk  (disk partitioning)

**NOTE: since for windows we already have ann efi partition. SO dont have to create another efi partition**

```
fdisk -l
```

```
cfdisk /dev/nvme0n1
```

- Highlight the free space and select [ New ] once again. 
- Put the desired partition size. You can use M to denote megabytes, G for gigabytes, and T for terabytes.
- To change the default type, keep the newly created partition highlighted and select [ Type ] from the list of actions.
- The default is Linux filesystem

So finally we have to hit Write

So we have
```
/dev/XXXX1 - 75 GB linux filesystem
/dev/XXXX2 - 996M linux swap
/dev/XXXX3 - 25GB linux filesystem
```

## How To Format the Partitions

- take a final look at your partition list by executing the following command:
```
fdisk -l /dev/nvme0n1
```
For all linux systems partitions use ext4 format

```
mkfs.ext4 /dev/nvme0n1p7
mkfs.ext4 /dev/nvme0n1p9
```

For swap

```
mkswap /dev/nvme0n1p8
```

# Mount the File Systems

```
# mount root to /mnt
mount /dev/nvme0n1p7 /mnt

# make home dir and mount 
mkdir /mnt/home
mount /dev/nvme0n1p9 /mnt/home

# NOTE DONT MOUNT EFI HERE. MOUNT IN BOOT LOADER SECTION

# enable swap partition 
swapon /dev/nvme0n1p8
```

# Select the mirrors
```
vim /etc/pacman.d/mirrorlist

# remove all the mirrors and put
Server = https://archive.archlinux.org/repos/2022/05/01/$repo/os/$arch

# Then update
pacman -Sy
```

# Install essential packages
```
pacstrap /mnt base base-devel linux linux-firmware sudo nano ntfs-3g networkmanager
```
- ntfs-3g – NTFS filesystem driver and utilities required for working with NTFS drives.


# Configure the system

# Generate an fstab file
> The fstab file can be used to define how disk partitions, various other block devices, or remote file systems should be mounted into the file system.

```
genfstab -U /mnt >> /mnt/etc/fstab
```

# Chroot

Change root into the new system:

```
arch-chroot /mnt
```

# Time zone

Set the time zone:

```
ln -sf /usr/share/zoneinfo/Asia/Calcutta /etc/localtime
```

# Run to generate /etc/adjtime

```
hwclock --systohc
```

# Localization

> Edit /etc/locale.gen and uncomment en_US.UTF-8 UTF-8

 Generate the locales by running:
 
 ```
 locale-gen
 ```

> you'll have to tell Arch Linux which one to use by default. To do so, open the /etc/locale.conf file and add the following line to it:
 
 ```
nano /etc/locale.conf

# ADD THE BELOW LINE
LANG=en_US.UTF-8
```

# Network configuration
Create the hostname file:

```
nano /etc/hostname

# ADD THE BELOW LINE
gauranga
```

# Root password

```
passwd
```

# install iwd

```
pacman -S iwd
```

# How To Install and Configure a Boot Loader

> GRUB is one of the most popular bootloaders out there.

To install GRUB, you'll have to first install two packages.

```
pacman -S grub efibootmgr
```

If you're installing alongside other operating systems, you'll also need the os-prober package:

```
pacman -S os-prober
```

> This program will search for already installed operating systems on your system and will make them a part of the GRUB configuration file.

## mount the EFI system partition 

- first create an efi directory
```
mkdir /boot/efi
```
- After creating the directory, you'll have to mount your EFI system partition in that directory.
```
mount /dev/nvme0n1p1 /boot/efi
```

# Install GRUB in the newly mounted EFI system partition:

Now, we'll use the grub-install command to install GRUB in the newly mounted EFI system partition:

# enable os-prober

> If you're installing alongside other operating systems, you'll have to enable os-prober before generating the configuration file. To do so, open the /etc/default/grub file in nano text editor. Locate the following line and uncomment it:

```
#GRUB_DISABLE_OS_PROBER=false
```

> This should be the last line in the aforementioned file so just scroll to the bottom and uncomment it.

# generating the grub configuration file

```
grub-mkconfig -o /boot/grub/grub.cfg
```
