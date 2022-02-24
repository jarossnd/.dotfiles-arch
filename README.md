# Arch Dotfiles & Configs

## Arch Installation

> Note: This article assumes you are using a US keyboard layout. You should also read the official Arch installation guide https://wiki.archlinux.org/index.php/installation_guide.

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

5. fdisk -l

6. fdisk /dev/sda (your drive may be different from sda)

7. Create EFI partition:
    - Type `n` for new
    - Type `p` for primary
    - Partition number: `1`
    - First sector: Use default
    - Last sector: `+300m`
    - Type `t` for type
    - Type `ef` for EFI system

8. Create SWAP partition
    - Type `n` for new
    - Type `p` for primary
    - Partition number: `2`
    - First sector: Use default
    - Last sector: +2G
    - Type `t` for type
    - Partition number: `2`
    - Type `82` for Linux Swap

9. Create Root partition
    - Type `n` for new
    - Type `p` for primary
    - Partition number: `3`
    - First sector: Use default
    - Last sector: Use default (to use the rest of the drive)
    - Type `t` for type
    - Partition number: `3`
    - Type `83` for Linux
    - Type `w` to write the partitions

10. Create the filesystem:
    - `fdisk -l` to view the partitions for the next step
   - `mkfs.fat -F32 /dev/sda1`
   - `mkswap /dev/sda2`
   - `mkfs.ext4 /dev/sda3`

11. Mount the file system
    - `mount /dev/sda3 /mnt`

12. Enable Swap
    - swapon /dev/sda2

13. Install Arch linux base packages:
    - Use the following if you want to include VIM:
    - `pacstrap -i /mnt base linux linux-firmware sudo vim`

14. Generate the `/etc/fstab` file:
    - `genfstab -U /mnt >> /mnt/etc/fstab`

15. Chroot into installed system:
    - `arch-chroot /mnt`

16. Find available timezones:
    - `timedatectl list-timezones`

17. Set the timezone:
    - `timedatectl set-timezone America/Chicago`

18. Update the Hardware clock:
    - `hwclock --systohc`

19. Localization
    - `vim /etc/locale.gen`
    - Uncomment `en_US.UTF-8 UTF-8` and save + quit
    - locale-gen
    - `vim /etc/locale.conf`
    - Add `LANG=en_US.UTF-8` to the file then save + quit

20. Set your hostname
    - `echo myhostname > /etc/hostname`
    - Update `/etc/hosts` with the following:

21. Set the root password
    - `passwd`

22. Let's enable the network

    - If you don't do this before your reboot you won't have internet connection
    - `pacman -S networkmanager`
    - `systemctl enable NetworkManager`

23. Install boot manager and other needed packages:

    - `pacman -S grub efibootmgr dosfstools openssh os-prober mtools linux-headers linux-lts linux-lts-headers`

24. Create EFI boot directory:

    - `mkdir /boot/EFI`
    - `mount /dev/sda1 /boot/EFI`

25. Install GRUB on EFI mode:

    - `grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck`

26. Setup locale for GRUB:

    - `cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo`

27. Write GRUB config:

    - `grub-mkconfig -o /boot/grub/grub.cfg`

28. Exit, unount and reboot:
    - `exit`
    - `umount -R /mnt`
    - `reboot`

## Post Arch Installation

### Before you begin

If it has been awhile between the Arch installation and the next setup, you should update the system before proceeding:

```
sudo pacman -Syu
```

### User Account Creation

We need to create ourselves a user account and we will do that now:

```
useradd -m username
passwd username
usermod -aG wheel,video,audio,storage username
```

We need `sudo` to give our account root privileges:

`pacman -S sudo`

Not recommended: We will now edit the `/etc/sudoers` file to allow our account to execute any command:

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
git clone https://github.com/jarossnd/.dotfiles-arch.git
cp -r .dotfiles-arch/.xmonad ~/
```

Reboot system


### Misc

sudo pacman -S xmobar
cp -r .dotfiles-arch/.config ~/.config



