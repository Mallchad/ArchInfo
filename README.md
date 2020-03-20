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
TODO(add install of wifi on install)
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
Note that all drives fall under `/dev` in the filesystem
You need minimum one root partition for the main mount point of the
system and if UEFI is enable an EFI system partition
The root partition is the starting point for filesystem of the installation
A BIOS specific partition is not neccecary but 2M is recommended
A swap patition is not neccecary recommended
Suggested Sizes
`EFI    256-512 MiB`
`SWAP   512 MiB`
root   Remainder of the device
Note that recommended SWAP partition size usually depends on the installed RAM
`4GB` is plenty for most systems with `8GB` or less memory installed whilst if you have
more then you should either match of have a bigger swap partition than installed memory
*e.g.* if you have `16GB` of RAM then `16GB` of swap would be ideal
`$ cfdisk /dev/<device>`

# Format the partitions
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

# Mount the partitions
--------------------
Mount the root pratition
`$ mount /dev/<root_partition> /mnt`
If you have an EFI system mount it
`$ mount /dev/<boot_partition> /mnt/boot`
Initialize swap partition
`$ swapon /dev/<swap_partition>`

# Installation
-------------
You may want to maximize download speed from Arch mirrors to
make the installation as painless as possible
The mirror is found at `/etc/pacman.d/mirrorlist`

Move the most geographically closest mirror to the top to 
increaes its pritority in usage by pacman
`$ nano /etc/pacman.d/mirrorlist`

Alternatively, attempt to install the pacman-contrib package to automatically 
generate a list to sort mirrors by their connection quality and average speed

First backup the existing mirrorlist just in case
`$ cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup`

Edit the newly copied mirrorlist.backup and uncomment (remove hashtag `#`) 
to specify with servers are to be tested
Run the following command if you want to uncomment and rank all servers
`$ ed -i 's/^#Server/Server/' /etc/pacman.d/mirrorlist.backup`

Running the following command will rank and set the top 6 mirrors to be used for installing
packages

`$ rankmirrors -n 6 etc/pacman.d/mirrorlist.backup > /etc/pacman.d/mirrorlist`
This will be added to the installation during the pacstrap proccess so it 
is worth getting it ideal now

Install the essential packages to the root for a usable install
You can use a different package instead of the standard linux kernel if you wish
`s $ pacstrap /mnt base linux linux-firmware`

# Configure the system
--------------------
Generate fstab to auto-mount partitions based on currently mounted 
partitions
`$ genfstab -U /mnt >> /mnt/etc/fstab`

You may want to check genfstab for errors, missing, or unintentional inclusions
`$ nano /mnt/etc/fstab`

Change the aparent root to the newly created system
`$ arch-chroot /mnt`

Here you may want to install a text editor based on personal preference
`$ pacman -S emacs`
`$ pacman -S vim`
Another few useful packages to install at this point would be
`tmux` console windowing
`zsh` shell with better completion and default keybinding
`htop` hardware and proccess monitor
`i7z` extensive cpu monitor
`nvtop` if you use nvidia graphics for gpu proccess and usage monitoring

Set the time zome to your own for the system
`$ lf -sf /usr/share/zoneinfo/<Region>/<City> /etc/localtime`
Generate adjtime
`$ hwclock --systohc`
Language Localization, this is for text rendering for American English
open locale with a text editor and uncomment
`en_US.UTF-8`
`UTF-8`
`$ emacs /etc/locale.gen`
Generate the locale
`$ locale-gen`
Create locale.conf file and set the language accordingly
`$ LANG=en_US.UTF-8 > /etc/locale.conf`

# Network configuration
----------------------
Create a hostname file and set a hostname
`$ <hostname> > /etc/hostname`

# Root password
-------------
Set the root password
**WARNING** Failiure to set a password for at least 1 user may result in an unsable 
install! You have been warned!
If you do fail to perform this step, use the arch live media again, mount the root 
and `arch-chroot` into it
`$ passwd`

# Bootloader
Depending on your system and installation you probably want a boot load 
this guide will cover the installation one of the most popular, the 
GRand Unified Bootloader, GRUB for short

# GRUB
-----------
Install grub
`$ pacman -S grub`

For BIOS just install it straight to the device. not the 
partition, the device itself
`$ grub-install --target=i386-pc /dev/<boot_device>`

For UEFI install efibootmgr as well, os-prober may be of use
if you intend to have multiple operating systems on the hard 
drive, many do so you might as well install it
`$ pacman -S efibootmgr`

Think of a name you want to see in the UEFI settings for
boot devices. Then install grub
`$ grub-install --target=x86_64-efi --efi-directory=/boot -- bootloader-id=GRUB`

Generate the grub configuration file this will search for
other operating systems if you have os-prober installed
these have to be mounted to be detected
If you do want to detect other operating systems then install os-prober
`$ pacman -S os-prober`

Before generating the grub configuration you may want to set the `intel_cstate` 
kernel paramater to `/etc/default/grub` by editing the line starting with 
`GRUB_CMDLINE_LINUX_DEFAULT=`
and adding `intel_cstate=1` within the speech marks, seperating it from other 
paramaters with a space inbetween
Note that this may slightly reduce battery life on laptops, this paramater is 
here because `intel_cstate=1` being to high has been known to soft/hard-lock 
computers with intel cpus on linux
`$ grub-mkconfig -o /boot/grub/grub.cfg`
# Reboot
------
Exit the chroot enviorment
Ctrl-D
or
`$ exit`
Reboot the system
`$ reboot`
Remove the USB the Arch live media is on if neccecary before startup
Login with root on login if all goes well
You should now have a working system, however very barebones

# Xorg
------
You probably want a graphical installation so a display server is neccecary
The most popular two is xorg and wayland, this guide will only cover xorg
xorg-apps may be neccecary depending on your needs
Install `xorg-server`, the `xorg` package group is fairly large so this guide 
explicitly installs the server only
`$ pacman -S xorg-server`

You probably want to have GPU acceleration so first identify the card on your
system then install the appropriate drivers
`$ lspci | grep -e VGA -e 3D`
to identify graphics cards

Note that for nvidia the proprietary drivers are the best performing usually
You may find that you prefer some of the difference of nouveau but I do not
You do not neccecarilly need the lib32 drivers since that is for x32 architecture 
acceleration but many apps still use x32 so it is recommended
`$ pacman -S`
NVidia      xf86-video-nouveau           //open-source supports NVidia optimus
            mesa
            lib32-mesa
Proprietary nvidia      
	        nvidia-utils
            lib32-nvidia-utils           //Stable
Legacy      nvidia-390xx
            nvidia-390xx-utils
            lib32-nvidia-390xx-utils     //Legacy
AMD         xf86-video-amdgpu
            xf86-video-ati
            mesa
            lib32-mesa
Intel       xf86-video-intel
            mesa
            lib32-mesa
			
By default xorg has mouse acceleration, that is pretty nasty so fix that 
with by creating a new file in /etc/X11/xorg.conf.d
The arch wiki recommends naming it `50-mouse-acceleration.conf`
Write the following into the file
`Section "InputClass"
	Identifier "My Mouse"
	MatchIsPointer "yes"
# set the following to 1 1 0 respectively to disable acceleration.
	Option "AccelerationNumerator" "1"
	Option "AccelerationDenominator" "1"
	Option "AccelerationThreshold" "0"
EndSection`

KDE Plasma
-----------
KDE Plasma is a very powerful and stylish modern desktop enviornment
Usually it comes with a lot of apps that you might not neccecarilly need
In true arch spirit this guide will work from the ground up and install 
the most barebones plasma package, `plasma-desktop`
Install relavant packages.

1 of the following
plasma                  //default installation
plasma-meta             //Inter-package dependency installation
plasma-desktop          //Minimal installation

1 of the following
kde-applications        //Full installation of all KDE applications
kde-applications-meta   //KDE install all KDE applications as a dependency
`$ pacman -S plasma-desktop`

Life is going to be difficult without a terminal `konsole` is the recommended
but I prefer simple terminal `st`
`$ pacman -S st`

Life will also be easier with a decent file manager, I like KDE's dolphin
`$ pacman -S dolphine
`
To start Plasma you need to use Xorg xinit/startx or a display manager
A display manager is easiest
Install the recommended display manager SDDM, it requires minimal configuration
`$ pacman -S sddm`

To be able to configure SDDM in system settings install the kcm package
`$ pacman -S sddm-kcm`
TODO(Install network manager for plasma)
TODO(Install bluetooth)
TODO(Install pulse audio)
TODO(Install aur helper)
Users
-----
# useradd -m -G <additional_groups> -s /bin/bash <username>





