# ArchInfo
This is a personal installation guide to aid with the fresh installation 
of Arch Linux. 
I do not recommend following this if you do not know me 
personally, however you are of course free to do so if you wish. 
This guide 
is not actively maintained so it will not always be up to date, check the 
date of the latest commit to see if it's worth using

# Notes
For the purposes of this guide a dollar sign '$' will be used to mark 
the start of a command, do not include this in the actual command

# Set The Keyboard Layout
You probably will not be happy with the default keyboard layout so 
let us change that
List availible keyboard layouts with this command
`$ ls /usr/share/kbd/keymaps/**/*.map.gz`
Modify the layout by adding the filename to the loadkeys command
$ ls /usr/share/kbd/i386/querty

# Verify the boot mode. BIOS|UEFI
-------------------------------
Run this command if you are unsure of the boot mode support by your motherboard
`$ ls /sys/firmware/efi/efivars`
If the directory does not exist the system may be booted in BIOS or CSM mode

# Connecting to the internet
---------------------------
Ensure your network interface is lsited and enabled
`$ ip link`
Connect to the network by plugging in an ethernet cable
If you are using wireless you need to connect with wireless manager
By default he Arch Live Media should come with the command wifi-menu 
to allow for easy connection to wireless access points
`$ wifi-menu`

# Update the System Clock
It could be important to set the system clock to allow for filesystem 
continuity or to avoid package-key out of date errors
`$ timedatectl set-ntp true`
You can check if the service is active by running the following
`$ timedatectl status`

# Paritioning the disks
----------------------------
Verify and identify partitions and devices that you want to use
# lsblk
or
# fdisk -l
Results ending in `rom`, `loop`, and `airroot` should be ignored as they 
are part of the installation media
You need minimum one root partition for the main mount point of the
system and if UEFI is enable an EFI system partition
The root partition is the starting point for filesystem of the installation
A BIOS specific partition is not neccecary but 2M is recommended
A swap patition is not neccecary recommended
Suggested Sizes
EFI    256-512 MiB
SWAP   512 MiB
root   Remainder of the device
Note that recommended SWAP partition size usually depends on the installed RAM
4GB is plenty for most systems with 8GB or less memory installed whilst if you have
more then you should either match of have a bigger swap partition than installed memory
e.g. if you have 16GB of RAM then 16GB of swap would be ideal
`$ cfdisk /dev/<device>`

Format the partitions
---------------------
Format the root partition as ext4 filesystem
`ext4` is the recommended filesystem for normal linux installations
There are other potential options such `zsh` and `btrfs`, this guide will
only be using the recommended ones by the arch, `fat` and `ext4`
`$ mkfs.ext4 /dev/<root_partition>`
Format the swap partition
`$ mkswap /dev/<swap_partition>`
Format the EFI boot partiton if you have one
`$ mkfs.fat /dev/<boot_partition>`

Mount the partitions
--------------------
Mount the root pratition
`$ mount /dev/<root_partition> /mnt`
If you have an EFI system mount it
`$ mount /dev/<boot_partition> /mnt/boot`
Initialize swap partition
#swapon /dev/<swap_partition>

Installation
-------------
You may want to maximize download speed from Arch mirrors to
make the installation as painless as possible
The mirror is found at /etc/pacman.d/mirrorlist
Move the most geographically closest mirror to the top to 
increaes its pritority in usage by pacman
this is temproary as it is written to the installation media
Install the base and extra base-devel package groups to root
# pacstrap /mnt base base-devel

Configure the system
--------------------
Generate fstab to auto-mount partitions based on current
mountings
# genfstab -U /mnt >> /mnt/etc/fstab
Change the root to the newly created system
# arch-chroot /mnt
Here you may want to install a text editor based on tastes
# pacman -S emacs
# pacman -S vim
Set the time zome to respective
# lf -sf /usr/share/zoneinfo/<Region>/<City> /etc/localtime
Generate adjtime
#hwclock --systohc
Language Localization, for American English open locale with
a text editor and uncomment
en_US.UTF-8
UTF-8
# emacs /etc/locale.gen
# locale-gen
Create locale and set the language accordingly
# LANG=en_US.UTF-8 > /etc/locale.conf

Network configuration
----------------------
Create a hostname file and set a hostname
<hostname> > /etc/hostname

Root password
-------------
Set the root password
passwd

GRUB
-----------
Install grub
# pacman -S grub
For BIOS just install it straight to the device. not the 
partition, the device itself
# grub-install --target=i386-pc /dev/<boot_device>
For UEFI install efibootmgr as well, os-prober may be of use
if you intend to have multiple operating systems on the hard 
drive, many do so you might as well install it
# pacman -S efibootmgr
Think of a name you want to see in the UEFI settings for
boot devices. Then install grub
# grub-install --target=x86-efi --efi-directory=/boot -- bootlaoder-id=<bootloader_name>
Generate the grub configuration file this will search for
other operating systems if you have os-prober installed
these have to be mounted to be detected
# grub-mkconfig -o /boot/grub/grub.cfg
Reboot
------
Exit the chroot enviorment
Ctrl-D
or
# exit
Reboot the system
# reboot
Login with root on login if all goes well
You should now have a working system, however very barebones

Xorg
----
Install xorg-server
xorg-apps may be neccecary
For both xorg-server and xorg-apps plus fonts just
install xorg
# pacman -S xorg

You probably want to have GPU acceleration so first
identify the card on your system then install the 
appropriate drivers
#pacman -S
NVidia      xf86-video-nouveau      //open-source supports NVidia optimus
        mesa
        lib32-mesa
Proprietary nvidia      
        nvidia-utils
        lib32-nvidia-utils      //Stable
Legacy      nvidia-390xx
        nvidia-390xx-utils
        lib32-nvidia-390xx-utils    //Legacy
AMD     xf86-video-amdgpu
        xf86-video-ati
        mesa
        lib32-mesa
Intel       xf86-video-intel
        mesa
        lib32-mesa
Install xorg-xinit
# pacman -S xorg-xinit
Copy the default init file to your home
cp /etc/X11/xinit/xinitrc ~/.xinitrc

KDE Plasma
-----------
Install relavant packages.
plasma          //default installation
plasma-meta     //Dependency free installation
plasma-desktop      //Minimal installation
kde-applications    //Full kde applications
kde-applications-meta   //kde applications dependency free
# pacman -S
To start Plasma you need to use Xorg xinit/startx
Append exec startkde to your .xinitrc found in your home folder
# exec startide >> ~/.xinitrc
Run Xorg as a regular user
# startx

Users
-----
# useradd -m -G <additional_groups> -s /bin/bash <username>





