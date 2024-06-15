---
id: 782
title: 'Adventures with Arch Linux on an Old Laptop - Dell Latitude E5570'
summary: 'Adventures with Arch Linux on an Old Laptop - Dell Latitude E5570'
date: '2024-03-17T13:55:06+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=782'
permalink: /2024/03/17/revitalize-dell-latitude-e5570-arch-linux/
categories:
    - Linux
---

## Adventures with Arch Linux on an Old Laptop - Dell Latitude E5570 (oh, I use arch btw moment)

I have an old laptop, a Dell Latitude E5570, and I wanted to revitalize it. Unfortunately, Windows 11 is not supported on this model without registry hacks, and I preferred not to install an unsupported product. Therefore, I decided to try Arch Linux, as the documentation indicated it should be supported:  
[Laptop/Dell – ArchWiki (archlinux.org)](https://wiki.archlinux.org/title/Laptop/Dell)

![Dell Latitude E5570](https://cdn.porotnikov.com/media/2024/03/13115438/image-1024x62.png)

## Installation

I started to follow the official installation guide:  
[Installation guide – ArchWiki (archlinux.org)](https://wiki.archlinux.org/title/Installation_guide)

Created bootable flash drive with dd:  
```
ls -l /dev/disk/by-id/usb-* dd bs=4M if=path/to/archlinux-version-x86_64.iso of=/dev/disk/by-id/usb-My_flash_drive conv=fsync oflag=direct status=progress
```

Booted into a new installation, and confirmed I’ve booted into 64bit UEFI:  
```
cat /sys/firmware/efi/fw_platform_size
```

Then, I used iwictl to connect to wifi:  
```
iwctl
device list
device device set-property Powered on
station device scan
station device get-networks
station device connect SSID
```

Verified that internet is working by pinging external resource.  
Checked with `lsblk` that my disk is `/dev/sda` and partitioned it as following:

```
fdisk /dev/sda
/dev/sda1 as EFI boot
/dev/sda2 as swap
/dev/sda3 as ext4
```

Created filesystems:  
```
mkfs.fat -F32 /dev/sda1
mkswap /dev/sda2
swapon /dev/sda2
mkfs.ext4 /dev/sda3
```

Now, considering that laptop has the following specs:

```plaintext
- CPU Intel® Core™ i5-6440HQ Processor  
- iGPU Intel® HD 530  
- Memory 16Gb (2 x Micron 8Gb 2133MHz)  
- Storage SSD 256GB SanDisk X300s m2 SATA  
- Display IPS 15.6 inch 1920×1080  
- Wifi & Bluetooth Intel® Wireless-AC 8260 Dual Band  
- LAN Intel® Gigabit Ethernet, 10/100/1000  
- Touchpad ALPS V8  
- Audio Realtek ALC293  
- External ports 3 x USB 3.0, 1 x RJ45, 1 x SD Card Reader, 1 x HDMI, 1 x 3.5 headphone/microphone combo, 1 x Smart Card Reader
```

I’ve installed those packages:  
```
pacman -S networkmanager alsa-utils pulseaudio sof-firmware xf86-input-libinput xf86-video-intel mesa man-db man-pages texinfo nano
```

Set the timezone:  
```
ln -sf /usr/share/zoneinfo/Europe/Warsaw /etc/localtime
```

And installed the bootloader like so:  
```
pacman -S grub efibootmgr intel-ucode
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

Then rebooted to the newly installed system. Success! The system boots ok.

## Post Boot

I’ve created a new user:  
```
useradd -m -G wheel,audio,video,network,storage,optical -s /bin/bash dmitry
passwd dmitry
EDITOR=nano visudo
```
Uncommented:  
`%wheel ALL=(ALL) ALL`

Connected to wifi using network manager:
```
sudo systemctl start NetworkManager
nmcli device
nmcli device wifi list
nmcli device wifi connect SSID password Password
nmcli connection show
```

Installed KDE desktop:
```
pacman -S plasma sddm konsole dolphin
systemctl enable sddm.service
pacman -S networkmanager plasma-nm
systemctl enable NetworkManager.service
```

Rebooted to see a desktop, but instead the whole screen was garbled with artifacts. There was apparently a graphics driver issue.

## Troubleshooting Graphics Driver Issue

I’ve played a lot with Hackintosh back in the day, and the screen looked oddly similar to a situation when an Intel gfx card is failing to load the driver in OS X.  
I checked the xorg log file, and observed this message:  
```
AIGLX error dlopen of /ust/lib/dri/i965_dri.so failed (no such file or directory)
AIGLX error: unable to load driver i965
```

It seems there was an issue related to the Mesa drivers, particularly the `i965` driver, which is used for older Intel integrated GPUs.

But I was sure I installed Mesa. What is in play here?  
Running `ls /usr/lib/dri/ | grep i965_dri.so` gives no output, meaning the .so file is not there. It turns out that the drivers for legacy Intel GFX cards were shifted out of Mesa to the `mesa-amber` package:  
**[Mesa-Amber Package Link](https://archlinux.org/packages/extra/x86_64/mesa-amber/)**

After installing this package (it conflicts with Mesa, Mesa has to be removed) the graphics started to work.  
There is also another graphics card in the laptop, by AMD.  
So I’ve installed:
```
sudo pacman -S xf86-video-amdgpu vulkan-radeon
```
and it seemed to enable this card as well.  
According to the documentation, PRIME offloading with recent versions of Xorg and the amdgpu driver should work out of the box.   
At least `DRI_PRIME=1 glxgears` gave more FPS 😉

## Enabling AUR

I wanted to have the possibility to install packages from AUR, so in the end what I did is:  
```
sudo pacman -S --needed base-devel git
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

In the end, I got a very fast, responsive Linux system on an aging Dell E5570 laptop.
