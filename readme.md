# Arch Linux installation Cheat Sheet

## Confirming boot mode

```
ls /sys/firmware/efi/efivars
```

if the file doesn't exists, BIOS mode, else UEFI.

## Setup Network
```
$ iwctl
```
```
[iwd]# help
```
Connect to a network

First, if you do not know your wireless device name, list all Wi-Fi devices:
```
[iwd]# device list
```
Then, to scan for networks:
```
[iwd]# station device scan
```
You can then list all available networks:
```
[iwd]# station device get-networks
```
Finally, to connect to a network:
```
[iwd]# station device connect SSID
```
Tip: The user interface supports autocomplete, by typing station and Tab Tab, the available devices are displayed, type the first letters of the device and Tab to complete. The same way, type connect and Tab Tab in order to have the list of available networks displayed. Then, type the first letters of the chosen network followed by Tab in order to complete the command.

If a passphrase is required, you will be prompted to enter it. Alternatively, you can supply it as a command line argument:
```
$ iwctl --passphrase passphrase station device connect SSID
```

Test Conection:

```
ifconfig

ping -c 2 google.com
```


## Partition Disk

```
fdisk -l
```

### BIOS

*/boot – 1024 MB*

*swap – 4 GB (min 512 MB) - Label: Linux Swap/ Solaris (82)*  

*/ – ~ 1000 GB (remaining space)*
### UEFI

*/efi – 1024 MB* - Label EFI (EF)

*swap – 4 GB (min 512 MB)*

*/ – ~ 1000 GB (remaining space)*

```
fdisk /dev/sda
```

Use `p` print, `n` new, `d` delete, `m` help

After defining the starting sector, select the size like `+100G` or `+100000M`...
Then use `t` to change label ( boot, EFI, swap)

After confirming everything press `w` to save it. It can be verified again with `fdisk -l`.

## Create Filesystem

**/boot or /efi (/dev/sda1)** 

- `EXT2` or `EXT3` for *BIOS*

- `Fat32` for *UEFI* 

**Swap (/dev/sda2)** as swap

**/ (/dev/sda3)** as EXT4 filesystem.

### BIOS

    mkfs.ext2 /dev/sda1

    mkfs.ext4 /dev/sda3

    mkswap /dev/sda2

### UEFI

    mkfs.fat -F32 /dev/sda1

    mkfs.ext4 /dev/sda3

    mkswap /dev/sda2

## Mount Partitions
```
mount
```
```/``` (root) partition must be mounted on ```/mnt``` directory.
```/boot``` partition needs to be mounted on ```/mnt/boot```.
Then initialize the swap partition.

### BIOS

    mount /dev/sda3 /mnt

    mkdir /mnt/boot

    mount /dev/sda1 /mnt/boot

    swapon /dev/sda2

### UEFI

    mount /dev/sda3 /mnt

    mkdir /mnt/efi

    mount /dev/sda1 /mnt/efi

    swapon /dev/sda2


## Select Mirrors

```
/etc/pacman.d/mirrorlist
```
Use the `reflector` to retrieve the latest mirror from the Arch Linux mirror status, filter the up-to-date mirrors, sort them by speed and update the mirror list file.

Backup the existing mirror list.

```cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup```

Then, update the mirrorlist file with 10 mirrors by download speed.

```reflector --verbose --latest 10 --sort rate --save /etc/pacman.d/mirrorlist```

## Install Arch Linux Base System

```pacstrap /mnt/ base linux linux-firmware net-tools networkmanager openssh vi sudo```

## Create fstab
Use `genfstab` command.
```genfstab -U /mnt >> /mnt/etc/fstab```

Verify:
```cat /mnt/etc/fstab```

## Arch Linux System Configuration

The `chroot` changes the root directory for the current running process, and their children. It will be like you go in your installation.
```arch-chroot /mnt```

## Set System Language

edit `/etc/locale.gen` uncommenting the required languages.

```vi /etc/locale.gen```


Uncomment `en_US.UTF-8 UTF-8` for *American-English* and then generate *locales* by running:
```locale-gen```

Set the `LANG` variable in `/etc/locale.conf` file.

```echo "LANG=en_US.UTF-8"  > /etc/locale.conf```

## Set Timezone

Now, configure the system time zone by creating a symlink of your timezone to the /etc/localtime file.

```# ln -sf /usr/share/zoneinfo/Zone/SubZone /etc/localtime```
Also, set the hardware clock to UTC.

```hwclock --systohc --utc```

### Arch Wiki Copy and Paste:
- To check the current zone defined for the system:

```$ timedatectl status```

- To list available zones:

```$ timedatectl list-timezones```

or all the available timezones are found under `/usr/share/zoneinfo` directory

- To set your time zone:

```# timedatectl set-timezone Zone/SubZone```

- Example:

```# timedatectl set-timezone Canada/Eastern```

This will create an /etc/localtime symlink that points to a zoneinfo file under /usr/share/zoneinfo/. 
In case you choose to create the link manually (for example during chroot where timedatectl will not work), keep in mind that it must be a symbolic link, as specified in archlinux(7):

```# ln -sf /usr/share/zoneinfo/Zone/SubZone /etc/localtime```

## Set Hostname

Place the system hostname in /etc/hostname file.

```echo "your-choosen-system-hostname" > /etc/hostname```

## Set root password

Use the passwd command in the terminal to set the root password.

```passwd```

## Install GRUB Boot Loader

Arch Linux requires a boot loader to boot the system. You can install the grub boot loader using the below commands.
### BIOS

    pacman -S grub

    grub-install /dev/sda

    grub-mkconfig -o /boot/grub/grub.cfg

### UEFI

    pacman -S grub efibootmgr

    grub-install --efi--directory=/efi

    grub-mkconfig -o /boot/grub/grub.cfg

## Reboot
```
exit

reboot
```

## Login to Arch Linux

Once the reboot is complete, you would get the Arch Linux login prompt. Log in as the root user and the password you set during the os installation.

## Turn on Internet

``` # systemctl start NetworkManager.service```

NetworkManager comes with nmcli(1) and nmtui(1).
nmcli examples

List nearby wifi networks:

```$ nmcli device wifi list```

Connect to a wifi network:

```$ nmcli device wifi connect SSID_or_BSSID password password```

example: `nmcli device wifi connect MY_OFFICE_WIFI password 123SECRETWIFIPASSWORD`

Connect to a hidden network:

```$ nmcli device wifi connect SSID_or_BSSID password password hidden yes```

Connect to a wifi on the wlan1 wifi interface:

```$ nmcli device wifi connect SSID_or_BSSID password password ifname wlan1 profile_name```

Disconnect an interface:

```$ nmcli device disconnect ifname eth0```

Reconnect an interface marked as disconnected:

```$ nmcli connection up uuid UUID```

Get a list of UUIDs:

```$ nmcli connection show```

See a list of network devices and their state:

```$ nmcli device```

Turn off wifi:

```$ nmcli radio wifi off```


Login as Root to Arch Linux and permantly set the keymap (keyboard)
-------------------------------------------------------------------

Now we want to make our keyboard layout permanent:

```bash
localectl set-keymap --no-convert dk
```

I obviously have "dk" for Danish layout. Substitute with your own layout.

NOTE: If you are a bloody American, you don't need to do this. US is default.

Check for updates, there's probably none
----------------------------------------

```bash
pacman -Syy
pacman -Syu
```

Add User and set Password
-------------------------

```bash
useradd -m -g users -G lp,scanner,audio,video,optical,network,games,wheel -s /bin/bash username
passwd username
```

Change sudoers file using nano
------------------------------

```bash
EDITOR=nano visudo
```

Uncomment wheel group.

`# %wheel ALL=(ALL:ALL) ALL`

Logout of Root
--------------

```bash
exit
```

Login as your username and test sudo with pacman
------------------------------------------------

```bash
sudo pacman -Syy
sudo pacman -Syu
```


1. Installing X server, Desktop Environment and Display Manager

Before installing a desktop environment (DE), you will need to install the X server which is the most popular display server.

```sudo pacman -S xorg```

Once it’s completed, use any of the below commands to install your favorite desktop environment.

You will also need a display manager to log in to your desktop environment. For the ease, you can install LXDM.

```pacman -S lxdm```

Once installed, you can enable to start each time you reboot your system.

```systemctl enable lxdm.service```

Enable fonts as well, example here`:

```sudo pacman -Syu ttf-dejavu``` 

* Installing X
    - Choose fastest mirror in /etc/pacman.d/mirrorlist
        + vim /etc/pacman.d/mirrorlist
        + Put the below line at the top:
            Server = http://mirror.nus.edu.sg/archlinux/$repo/os/$arch
    ```sudo pacman -S xorg
    sudo pacman -S xorg-twm xorg-xclock xterm
    startx```
    ##### Take note that: never try to create /etc/xorg.conf file!!! Try to create
          this file will make X not able to start!!!


* Sound:
    Sound is muted by default, so need to unmuted it:
    https://wiki.archlinux.org/index.php/Advanced_Linux_Sound_Architecture#Unmuting_the_channels
    sudo pacman -S alsa-utils

* Disable beep sound:
    - sudo rmmod pcspkr
    - black list to prevent loading at boot:
        sudo echo "blacklist pcspkr" > /etc/modprobe.d/nobeep.conf



# Some useful programs to install

**Arch Repo**
-   ARandr *GUI for Display Config*
-   pcmanfl *File Manager*
-   sxiv  *Image Viewer*
-   zathura *Document reader*
-   xf86-input-synaptics *Most laptops touchpad*
-   xterm *x terminal*


**AUR** 
first install base-devel and git:

```
pacman -S --needed git base-devel 
git clone https://aur.archlinux.org/yay.git 
cd yay makepkg -si
```

-   zathura-pdf-poppler *plugin for Zathura open pdf*
-   st *suckless terminal*


 - References: https://gist.github.com/dhhdev/a8cc055ce7ba04a85528, https://wiki.archlinux.org, https://www.itzgeek.com/how-tos/linux/arch-linux/install-arch-linux-2021.html
`