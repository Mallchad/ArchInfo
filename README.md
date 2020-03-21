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
Run this command if you are unsure of the boot mode support by your motherboard
`$ ls /sys/firmware/efi/efivars`
If the directory does not exist the system may be booted in BIOS or CSM mode

# Connecting to the internet
Ensure your network interface is lsited and enabled
`$ ip link`
Connect to the network by plugging in an ethernet cable
If you are using wireless you need to connect with wireless manager
By default he Arch Live Media should come with the command wifi-menu 
to allow for easy connection to wireless access points
`$ wifi-menu`
Verify connection by attempting to ping a website
`$ ping google.com`
`Ctrl-C` to cancel command
If there is a working connection the ping should say it has recieved 
packges back from address
# Update the System Clock
It could be important to set the system clock to allow for filesystem 
continuity or to avoid package-key out of date errors
`$ timedatectl set-ntp true`
You can check if the service is active by running the following
`$ timedatectl status`

# Paritioning the disks
Verify and identify partitions and devices that you want to use
`$ lsblk`
or
`$ fdisk -l`
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
Mount the root pratition
`$ mount /dev/<root_partition> /mnt`
If you have an EFI system mount it
`$ mount /dev/<boot_partition> /mnt/boot`
Initialize swap partition
`$ swapon /dev/<swap_partition>`

# Installation
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
If you are setup the internet connection with `wifi-menu` you are probably 
relying on a wireless connection, to be able to have a connection until you 
install graphical networking tools install `netctl` and `dialog` to get 
wi-fi menu for the install, note you may have to reinput network password

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

Add x32 Packages to pacman
Uncomment from `/etc/pacman.conf`
`
#[multilib]
#Include = /etc/pacman.d/mirrorlist
`
For fun also remove uncomment `Color` from `/etc/pacman.conf`
to beutify the output
Also adding `ILoveCandy` to the same file will do fun things
# Network configuration
Create a hostname file and set a hostname
`$ <hostname> > /etc/hostname`

# Root password
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
	set the following to 1 1 0 respectively to disable acceleration.
	Option "AccelerationNumerator" "1"
	Option "AccelerationDenominator" "1"
	Option "AccelerationThreshold" "0"
EndSection`

# KDE Plasma
Install
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

### Applications
Life is going to be difficult without a terminal `konsole` is the recommended
but I prefer simple terminal `st`
`$ pacman -S st`

Life will also be easier with a decent file manager, I like KDE's dolphin
`$ pacman -S dolphin`

You are most certainly going to miss a browser, get one now
`$ pacman -S chromium`
Note: Later on this guide goes over the AUR briefly, after that you can install 
the more normal `google-chrome` which has slightly more features that chromium
and is more stable

### Display Manager
To start Plasma you need to use Xorg xinit/startx or a display manager
A display manager is easiest
Install the recommended display manager SDDM, it requires minimal configuration
`$ pacman -S sddm`

To be able to configure SDDM in system settings install the kcm package
`$ pacman -S sddm-kcm`

### Sound 
Personally in the system tray I prefer `kmix to the default` `plasma-pa`
`$ pacman -S kmix`

### Network
To be able to configure and connect to networks from within the desktop 
enviornment you should install `plasma-nm` wifi-menu requires root 
privillages and needs to be connected every boot so that is not ideal
`$ pacman -S plasma-nm`
###Bluetooth
If you want to be able to connect to bluetooth devices from within plasma
then install the bluedevil package
`$ pacman -S bluedevil`

# Other
###Bluetooth
If you did want to use bluetooth then set that up by installing `bluez` 
and `bluez-utils` packages
`$ pacman -S bluez bluez-utils`

Enable the daemon to allow bluetooth to run at startup
`systemctl start bluetooth.service`

###Pulseaudio
If you want to use bluetooth headphones install `pulesaudio` and 
`pulseaudio-bluetooth`
`$ pacman -S pulseaudio pulseaudio-bluetooth`

A nice to have is `pulseeffects` to allow you to control annoying 
and poor quality audio on that video or series you desprately need to watch
`$ pacman -S puleseffects`

# AUR
The arch user repository is a massive collection of community submitted packages,
for many arch users you simply cannot do without it
yay is a popular helper that simplifies the experience to treat the proccess 
more like a standard pacman, but with more commands and mirrors, namely the 
AUR itself

Before you do anything with the AUR you are going to need the `base-devel` package
Note: this has a lot of really useful packages related to software development and 
package compilation, in general most arch uses are going to want it except in the 
most minimal edge-case installations
`pacman -S base-devel`

To get started install yay, first, `wget` it, since you don't have a user setup
`/tmp` will do fine for this
`$ cd /tmp`
`$ wget -O /tmp/yay.tar.gz https://aur.archlinux.org/cgit/aur.git/snapshot/yay.tar.gz`

Extract the tarball
`$ tar xfv yay.tar.gz`
`cd` into the `yay` folder, then make and install the package with dependencies
Note: specify to remove make dependencies if storage space is a concern, it can always 
be cleaned up later with `pacman -Scc`
`makepkg -si`

###Usage
Simply use the `yay` command as if it were pacman, all normal commands will work

# Users
###Creating a User
To use your installation you are going to want to create yourself a user, running 
everything as root is a REALLY REALLY bad idea because it means all apps and 
software that runs will have unrestricted, no-confirmation access to your whole 
computer, leaving a massive gaping security hole in your system and the install 
liable to random and unexpected damage (data damage)
`$ useradd -m -G wheel -s /bin/zsh <new_username>`
Note: `wheel` is the system administrators group and will give you access to certain 
system modifacation without first going through root
zsh is my the shell I like to use, if you are not bothered `bash` or `sh` are 
the preinstalled ones, `sh` I consider to be better than bash

Give your new user a password so you can log in
`$ passwd <new_username>`

# Sudo
It was mentioned before that running everything as root is bad, but every now and again 
you need temporary root privillages
If you did not install the base-devel package install sudo now
`$ pacman -S sudo`

### Setup
Edit the sudo configuration with `visudo`, I recommend changing the EDITOR to nano 
if you cannot get along with vi, I set mine to nano
For first time set the editor by preixing visudo with 
`EDITOR=nano`

To allow users of group `wheel` sudo access make sure that this is uncommented 
somewhere in the file
`%wheel      ALL=(ALL) ALL`

To allow a single user to have full root privillages with sudo add the following 
`<username>      ALL=(ALL) ALL`
Usage
To use sudo simple prefix the desired command with `sudo`
You will be then prompted for the root password, or if you set a specific user 
to have sudo access then it may just ask for the user password

# Bonus
I personally like using a different window manager for plasma, kwin is great, but it is
not quite enough for my needs.
Install the window manager of your choice, I chose `awesome`
Setup a new xsession by first making a copy of `/usr/share/xsessions/plasma.desktop` 
and naming it somebody ending with `.desktop`
Edit the newly created file and modify the line starting with `Exec` to be the following 
`Exec=env KDEWM=/usr/bin/<window_manager_executable> /usr/bin/startplasma-x11`
Then change the name to be something descriptive
To use, when you reboot look in the corner of SDDM, and change the session to the desired

# End
You should now have arch pretty much installed, there will be many more changes to make 
and packages to install, but this is where the guide ends. 
Just remember, arch is only as stable as you make it, and arch does not do partial upgrades
If the system is unstable after a package install, do a full system upgrade with 
`$ pacman -Syu`
If the system is still unstable then try a different graphics driver, and also try 
installing proccessor microcode updates. These are loaded from the kernel at boot time
and do not overwrite any firmware, it is completely safe to do.
