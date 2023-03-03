# Arch Linux - Customized Installation Guide
**Note:** For installing Arch on a Macbook machine, you would need to boot ARCH_2019XX installation drive by adding `nomodeset` to the Kernel line in the grub menu.

**Steps to Cover**
```
1- Partitions Configuration
2- System Configuration
3- Keyboard Configuration -OPTIONAL-
4- GRUB Bootloader
5- Graphical User Interface
6- Desktop Environment
7- Time Zone Set
8- Protect Your Linux
```

----------------------------------
# **1- Partitions Configuration**
----------------------------------
At below an example of my partition's table:
```
$ cfdisk /dev/sda
```

```
    Device                             Start                 End             Sectors            Size Type
>>  /dev/sda1                           2048              206847              204800            100M EFI System               ## Only required for UEFI Systems
    /dev/sda2                         206848              208895                2048              1M BIOS boot                ## Only required for UEFI Systems
    /dev/sda3                         208896           178466815           178257920             85G Linux filesystem
    /dev/sda4                      178466816           468862094           290395279          138.5G Linux filesystem
```

***Format Partitions:***

```
$ mkfs.fat /dev/sda1                   ## Only required for UEFI Systems
$ mkfs.fat /dev/sda2                   ## Only required for UEFI Systems
$ mkfs.ext4 -L “Arch” /dev/sda3
$ mkfs.ext4 -L “Home” /dev/sda4
$ mount /dev/sda3 /mnt
$ mkdir /mnt/home && mount /dev/sda4 /mnt/home
```

***For UEFI Systems:***
```
$ mkdir -p /mnt/boot/efi && mount /dev/sda1 /mnt/boot/efi
```

-OPTIONAL- If Linux Swap partition is implemented:
```
$ mkswap -L “Linux Swap” /dev/sdaX
$ swapon /dev/sdaX
```

***For Installing Basic OS:***
```
$ pacstrap /mnt base base-devel linux linux-firmware linux-headers vi vim dhcpcd networkmanager ntfs-3g mlocate net-tools openssh bind-tools nmap
$ genfstab -U /mnt >> /mnt/etc/fstab
```
## *End - Partitions Configuration*


------------------------------
# **2- System Configuration**
------------------------------
```
$ arch-chroot /mnt /bin/bash
$ passwd root
$ echo arch-linux > /etc/hostname
```

***Add User:***
```
$ useradd -m -g users -G wheel -s /bin/bash -m -c "username's full name" [username] && passwd [username]
```
or maybe you want to create the user and add this one to the common groups `$ useradd -G audio,lp,scanner,optical,storage,video,wheel,games,power,http -s /bin/bash -m -c “[username]'s full name” [username] && passwd [username]`.

***Add Created User to Sudoers:***
Edit `/etc/sudoers` file and uncomment the next line:
```
%wheel ALL=(ALL) ALL
```
if you are using VI, save the file with `:wq!`

***To delete user accounts (Only if it's needed):***
```
# userdel -r [username]
```

***Add user to a group (Only if it's needed):***
```
# gpasswd -a [user] [group]
```

***Remove user from a group (Only if it's needed):***
```
# gpasswd -d [user] [group]
```

Locale is set by edit `vi /etc/locale.gen` to uncomment the desired locales:
```
en_US.UTF-8 UTF-8
es_CO.UTF-8 UTF-8
```
```
$ locale-gen
$ echo LANG=en_US.UTF-8 > /etc/locale.conf
$ echo KEYMAP=us > /etc/vconsole.conf
$ export LANG=en_US.UTF-8
```

## *End - System Configuration*


-------------------------------------------
# **3- Keyboard Configuration** -OPTIONAL-
-------------------------------------------
You can use the following command to view the current keyboard configurations, amongst other localized settings:
```
$ localectl status
   System Locale: LANG=en_GB.utf8
                  LC_COLLATE=C
       VC Keymap: cz-qwertz
      X11 Layout: cz
```

For a list of all the available keymaps, use the command:
```
$ localectl list-keymaps
```

***Temporary configuration***
Of course it is possible to set a keymap just for current session. This is useful for testing different keymaps, solving problems etc.
The loadkeys tool is used for this purpose, it is used internally by systemd when loading the keymap configured in `/etc/vconsole.conf`. It can be used very simply for this purpose, per example for Latin America use `la-latin1`:
```
$ loadkeys la-latin1
```

For permanet set add the line into `/etc/vconsole.conf`:
```
KEYMAP=la-latin1
```

***To change the type of the console font***: KBD package gives the needed tools to change font types for the virtual console and map characters. The font types are included into `/usr/share/kbd/consolefonts/`.
Edit `/etc/vconsole.conf` to include the proper **FONT** and **FONT_MAP** for your choice, for permanent set add the lines into the file. Per example below:
```
FONT=Lat2-Terminus16
FONT_MAP=8859-15
```

## *End - Keyboard Configuration*


-------------------------
# **4- GRUB Bootloader**
-------------------------
***For Legacy BIOS:***
```
$ pacman -S grub
$ grub-install /dev/sda
```

***For UEFI Systems:***
```
$ pacman -S grub efibootmgr freetype2 dosfstools libisoburn mtools
$ grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Arch
```

***Now generate the boot***
```
$ grub-mkconfig -o /boot/grub/grub.cfg
```

***Then, exit chroot and umount all partitions and reboot the machine***
```
$ exit
$ umount /mnt/home
$ umount /mnt/boot/efi          # Optional if UEFI
$ umount /mnt
$ reboot
```

***Note: Don't forget to remove the Arch Linux Installer at this point.***

## *End - GRUB Bootloader*


--------------------------------
# **5- Graphical User Interface**
--------------------------------
Edit and customize `vi /etc/pacman.conf` and uncomment out `[multilib]`, then review how is called the network interface
```
$ ls /sys/class/net
```

Then, active the network interface
```
$ ip link set enp2s0 up
$ dhcpcd enp2s0
```

Test the internet connection:
```
$ ping -c3 google.com
```

Update repo databases:
```
$ pacman -Syu
```

***Install NetworkManager***
```
$ pacman -S networkmanager
```

***Now, Configure Public DNS and Start the NetworkManager applet***
```
$ systemctl enable NetworkManager
$ systemctl start NetworkManager
$ nmcli con mod <connectionName> ipv4.dns "8.8.8.8 8.8.4.4"       ## Optional
$ systemctl restart NetworkManager
```

Before reboot the machine, don't forget to enable onboot from `vi /etc/NetworkManager/system-connections/<connectionName>.nmconnection` in `autoconnect=true` parameter.

***Xorg***
- Nvidia Graphic Card:
```
$ pacman -S xorg xorg-server xorg-apps xorg-xinit xorg-twm vim xterm unrar unzip p7zip nvidia nvidia-utils nvidia-settings opencl-nvidia lib32-nvidia-utils
```

If you have old video card Nvidia: `pacman -S nvidia-340xx-lts nvidia-340xx-utils`

- Installing Arch Linux with dual graphics cards:
https://wiki.archlinux.org/index.php/Bumblebee#Installing_Bumblebee_with_Intel.2FNVIDIA

***Note: don't forget to test the Graphic User Interface before rebooting the machine.***
```
$ startx
```

## *End - Graphical User Interface*


-----------------------------
# **6- Desktop Environment**
-----------------------------
Install the the desktop as you choice, mine is KDE Plasma.

***Installing KDE:***
```
$ pacman -S plasma kde-applications k3b emovix normalize sox transcode opusfile vcdimager firefox firefox-i18n-en-us xdg-user-dirs
```

Creating default directories (***you need to be logged as user no root to run the next command***):
To create a full suite of localized default user directories within the $HOME directory, enter the following commands:
```
$ xdg-user-dir && xdg-user-dirs-update
```

Start KDE:
```
$ systemctl enable sddm
$ reboot
```

***Chinese optional repo, a lot of AUR packages are already compiled and included here*** (***unoficial repo***)
Add the following to `vi /etc/pacman.conf` at the end of the file:
```
[archlinuxcn]
Server = http://repo.archlinuxcn.org/$arch
## or install archlinuxcn-mirrorlist-git and use the mirrorlist
#Include = /etc/pacman.d/archlinuxcn-mirrorlist
```

For mirrors (mainly in China), see https://github.com/archlinuxcn/mirrorlist-repo. To add PGP Keys:
```
sudo pacman -Syy && sudo pacman -S archlinuxcn-keyring
```

Also, install `yay` (yaourt or aurman replacement) to be able to install AUR packages:
```
$ pacman -S yay
```

***For Laptops and WiFi:***
Use wireless network, it needs a working network to install the broadcom-wl-dkms. Then run wifi-menu to connect to WIFI. It works with great performance!
```
$ yay -S broadcom-wl-dkms
```
or
```
$ yay -S b43-firmware
```
Then, you need to add the module to the /etc/mkinitcpio.conf "b43"
Then, you need to run
```
$ modprobe b43
$ mkinitcpio -p linux
```

***Multimedia:***
```
$ yay -S vlc mplayer pulseaudio pulseaudio-equalizer pulseaudio-gconf pulseaudio-jack pulseaudio-lirc
```

***Installing Amarok:***
```
yay -S amarok libgpod loudmouth ifuse libmygpo-qt clamz gst-libav gtk-sharp-2
```

***Installing KVM/QEMU/Virt-Manager (VirtualBox replacement)***

```
yay -S qemu virt-manager ebtables dnsmasq bridge-utils openbsd-netcat firewalld dmidecode virt-viewer swtpm
```
For drivers support in Windows OS Guest, install the `virtio-win` AUR package and once Windows OS is installed, mount `/usr/share/virtio/virtio-win.iso` to install Windows OS drivers:
```
yay -S virtio-win
```
Prepare Virt-Manager to work with the user:
```
sudo usermod -G libvirt -a [user]
```
```
sudo systemctl enable libvirtd && sudo systemctl start libvirtd
```
```
sudo systemctl status libvirtd
```
How to convert VirtualBox images to be used in Virt-Manager:
```
sudo qemu-img convert -f vdi -0 qcow2 vitual_machine.vdi virtual_machine.qcow2
```

***Installing LibreOffice and Office Components:***
```
$ yay -S libreoffice-fresh libreoffice-fresh-es libreoffice-extension-texmaths libreoffice-extension-writer2latex hunspell hunspell-es hunspell-en hyphen hyphen-es hyphen-en libmythes mythes-en mythes-es languagetool
```

***Installing Internet Browsers and Flash Player support on Opera/Chromium/Chrome/Vivaldi***
```
$ yay -S opera opera-ffmpeg-codecs firefox vivaldi chromium google-chrome pepper-flash ttf-dejavu ttf-freefont ttf-liberation ttf-bitstream-vera ttf-linux-libertine ttf-droid ttf-ubuntu-font-family ttf-oxygen noto-fonts ttf-croscore ttf-ms-fonts terminus-font flashplugin
```

***Installing useful apps and common components***
```
$ yay -S octopi gist skype teamviewer dropbox turtl sublime-text lib32-alsa-plugins pavucontrol lib32-libcanberra xclip xsel lib32-jack lib32-libsamplerate lib32-speex lib32-libcanberra-pulse openshot libopenshot-audio openshot frei0r-plugins libquicktime libavc1394 faac jack jack-rack python-opengl python-dbus qt5-serialport python-pyqt4 aic94xx-firmware wd719x-firmware trillian filezilla
```

***Installing a Very Nice theme which is called Papirus:***
```
$ yay -S papirus-aurorae-theme papirus-color-scheme papirus-gtk-theme papirus-icon-theme-kde papirus-konsole-colorscheme papirus-look-and-feel papirus-plasma-theme papirus-sddm-theme arc-dark-suite papirus-bomi-skin papirus-k3b-theme papirus-kmail-theme papirus-libreoffice-theme papirus-qtcurve-theme papirus-smplayer-theme papirus-vlc-theme papirus-wallpapers papirus-yakuake-theme
```

## *End - Desktop Environment*


-----------------------
# **7- Time Zone Set**
-----------------------
Clock shows a value that is neither UTC nor local time. This might be caused by a number of reasons. For example, if your hardware clock is running on local time, but timedatectl is set to assume it is in UTC, the result would be that your timezone's offset to UTC effectively gets applied twice, resulting in wrong values for your local time and UTC.

To force your clock to the correct time, and to also write the correct UTC to your hardware clock, follow these steps:
```
$ timedatectl				                            ## Check the current time zone status.
$ timedatectl set-local-rtc 1                   ## **OPTIONAL** Set your hardware clock to local timezone.
$ timedatectl set-timezone America/Bogota	      ## Set your time zone correctly.
$ timedatectl set-ntp true                      ## To start automatic time synchronization with remote NTP server, type the following command at the terminal.
$ timedatectl				                            ## Check the new current time zone status and look for "System clock synchronized: yes".
```

- Reference: https://www.tecmint.com/set-time-timezone-and-synchronize-time-using-timedatectl-command/

## *End - Time Zone Set*


----------------------------
# **8- Protect Your Linux**
----------------------------
***NOTE: Perform all commands below as ROOT.***
Install a Firewall and start/enable it:
```
$ pacman -S ufw
$ systemctl enable ufw
$ systemctl start ufw
$ ufw enable
```

Make some rules:
***NOTE: VERY RECOMMENDED is add SSH access to a different port that the default one to avoid connection attacks when the standard 22 is open***, don't forget include the new port into `/etc/ssh/sshd_config` file and also if it is required make the port forwarding entry into the router.
```
$ ufw deny SSH comment 'Default SSH port (22) blocked'
$ ufw allow [new SSH port] comment 'For NEW SSH port assigned'
$ ufw allow CIFS comment 'For Shared Folders (SMB)'
```
more instructions at https://wiki.archlinux.org/index.php/Uncomplicated_Firewall.

Finally, check status of your new Firewall:
```
$ ufw status numbered
```

## *End - Protect Your Linux*
