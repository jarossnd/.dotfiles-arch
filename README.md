# Arch Dotfiles & Configs

## Arch Installation

> Note: This article assumes you are using a US keyboard layout and we will also be setting up swap as a file instead of a partition. You should also read the official Arch installation guide https://wiki.archlinux.org/index.php/installation_guide. 

1. Verify boot mode:
    - This is an important set when working with a UEFI system. Run the below command and if the directory does not exist, the system may not be booted in UEFI mode.
    - `ls /sys/firmware/efi/efivars` (If the directory exist your computer supports EFI)

2. Ping some site on the Internet to verify connection:
    - Use `ip link` to verify your network devices
    - If necessary, use `iwctl` to setup your wireless
    - Run `ping` to check to see if you already have an internet connection
    - `ping archlinux.org -c 5`

3. Update system clock:
    - View the current status of your clock by running `timedatectl status`
    - Run the following command to ensure your system clock is correct `timedatectl set-ntp true`
    - This will set your date and time to update remotely using network time protocal

4. Go to [https://archlinux.org/mirrorlist](https://archlinux.org/mirrorlist) and find the closest mirror that supports HTTPS:
    - Add the mirrors on top of the `/etc/pacman.d/mirrorlist` file.
    - `Server = https://mirror.arizona.edu/archlinux/$repo/os/$arch` (United States)

5. cfdisk

6. Create EFI partition:
    - New
    - 300M
    - Type
    - EFI System
    - Write

7. Create `/root` partition:
    - Select free space
    - New
    - 30G
    - Linux filesystem
    - Write

8. Create `/home` partition:
    - Select free space
    - New
    - Use rest of the space
    - Linux filesystem
    - Write

9. Create the filesystems:
    - `fdisk -l` to view the partitions for the next step
    - `mkfs.fat -F32 /dev/sda1`
    - `mkfs.ext4 /dev/sda2`
    - `mkfs.ext4 /dev/sda3`
    - 

10. Create the `/root` and `/home` directories:
    - `mount /dev/sda2 /mnt`
    - `mkdir /mnt/home`
    - `mount /dev/sda3 /mnt/home`

11. Install Arch linux base packages:
    - Use the following if you want to include VIM:
    - `pacstrap -i /mnt base linux linux-firmware sudo vim`
    - Use the following if you want to also include Nano:
    - `pacstrap -i /mnt base linux linux-firmware sudo nano vim`

12. Generate the `/etc/fstab` file:
    - `genfstab -U -p /mnt >> /mnt/etc/fstab`

13. Chroot into installed system:
    - `arch-chroot /mnt`

14. Find available timezones:
    - `timedatectl list-timezones`
    
15. Set the timezone:
    - `ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime`
    - or `timedatectl set-timezone America/Chicago`

16. Update the Hardware clock:
    - `hwclock --systohc`

17. Set your hostname and configure hosts
    - `echo myhostname > /etc/hostname`
    - Update `/etc/hosts` with the following:

```
127.0.0.1	localhost
::1		localhost
127.0.1.1	myhostname.localdomain	myhostname
```

18. Set the root password
    - `passwd`

19. Let's enable the network
    - If you don't do this before your reboot you won't have internet connection
    - `pacman -S networkmanager`
    - `systemctl enable NetworkManager`

20. Install boot manager and other needed packages:
    - `pacman -S grub efibootmgr dosfstools openssh os-prober mtools linux-headers linux-lts linux-lts-headers`

21. Set locale:
    - `sed -i 's/#en_US.UTF-8/en_US.UTF-8/g' /etc/locale.gen` (uncomment en_US.UTF-8)
    - `locale-gen`
    - `vim /etc/locale.conf`
    - `LANG=en_US.UTF-8`

22. Create EFI boot directory:
    - `mkdir /boot/EFI`
    - `mount /dev/sda1 /boot/EFI`

23. Install GRUB on EFI mode:
    - `grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck`

24. Setup locale for GRUB:
    - `cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo`

25. Write GRUB config:
    - `grub-mkconfig -o /boot/grub/grub.cfg`

26. Create swap file:
    - `fallocate -l 2G /swapfile`
    - `chmod 600 /swapfile`
    - `mkswap /swapfile`
    - `echo '/swapfile none swap sw 0 0' | tee -a /etc/fstab`

27. Exit, unount and reboot:
    - `exit`
    - `umount -R /mnt`
    - `reboot`


## Post Arch Installation

### User Account Creation

We need to create ourselves a user account and we will do that now:

```
useradd -m username
passwd username
usermod -aG wheel,video,audio,storage username
```

We nee4d `sudo` to give our account root privileges:

`pacman -S sudo`

We will now edit the `/etc/sudoers` file to allow our account to execute any command:

```
## Uncomment to allow members of group wheel to execute any command
# %wheel ALL=(ALL) ALL
```

Reboot and confirm you can login

### Wireless

You may have installed Arch Linux while you computer was plugged into the internet. At this point, you may want to start using Wifi. Even if you used `iwctl` during the Arch setup, the program is no longer available since we rebooted. Since we previously installed `NetworkManager`, we can now use `nmcli` to connect to a Wifi network:

```
# List  available networks
nmcli device wifi list
# Connect to a Wifi network
nmcli device wifi connect YOUR_SSID password YOUR_PASSWORD
```

There are more `nmcli` examples that can be found on [this page](https://wiki.archlinux.org/index.php/NetworkManager#nmcli_examples)

### Window Server

We need to install Xorg which will be our Window Server

```
sudo pacman -S xorg
```

## Login and Window Manager

### Xmonad Installation

```
sudo pacman -S lightdm lightdm-gtk-greeter xmonad xmonad-contrib xterm code firefox dmenu nitrogen picom
```

Start the `lightdm` service:

```
sudo systemctl enable lightdm
reboot
```

Reboot and you should get the lightdm graphical login screen. Once you login, Xmonad should be running. You will know because you will be at a completely black screen with a cursor. Use Alt + Shift + Enter to launch a terminal window for the next steps.

We need to install `git` so that we can pull down dotfiles.

`sudo pacman -S git`

We will now clone our repository and copy the config files

```
git clone https://github.com/jarossnd/dotfiles.git
cp -r dotfiles/.xmonad ~/
```

Reboot system

