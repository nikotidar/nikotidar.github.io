---
layout: post
title: "KISS Linux and ThinkPad = <3"
author: "rndtx"
---

# Install KISS
The installation is very similiar to Gentoo's stage 3 tarballs.

An archive is used which contains a full KISS system minus the boot-loader and kernel. The provided archive contains all of the tooling needed to rebuild itself as well as the remaining packages needed for an installation.

You will need an existing Linux distribution to use as a base for the installation. It does not matter what kind of distribution it is nor does it matter what ```lib``` it uses.

For the purpose of this guide I will be using another Linux distribution's Live-CD to bootstrap KISS. The Live-CD *must* include ```tar``` (with ```xz``` support), ```mountpoint``` and other basic utilities.

From this point on, the guide assumes you have booted a Live-CD and have an *internet connection*.

# Index
* Setting up disks
* Install KISS
* Enable repository signing
* Rebuild KISS
* Build userspace tools
* Configure and build the kernel
    * Download the kernel sources
    * Extract the kernel sources
    * Configure the kernel
    * Build the kernel
    * Install the kernel
* Install grub
* Install init scripts
* Enable the community repository

# Setting up disks
Get the disks ready for the installation. This involves creating a partition table, partitions and the desired file-systems. This step will differ depending on whether or not you are doing a BIOS or UEFI installation.

The guide will *not* cover this step. If you require assistance with this step; read one of the links below. This step is not unique to KISS and there are tried and tested resources for all kinds of disk layout online.

* [Gentoo wiki](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Disks)
* [Archlinux wiki](https://wiki.archlinux.org/index.php/Installation_guide#Partition_the_disks)

# Install KISS
At this stage your disks should be setup and mounted to ```/mnt```.

Download the latest release.

```sh
$ wget https://dl.getkiss.org/kiss-chroot.tar.xz

# recommend: verify the download.
$ wget https://dl.getkiss.org/kiss-chroot.tar.xz.sha256
$ sha256sum kiss-chroot.tar.xz | diff kiss-chroot.tar.xz.sha256 - && echo good

# recommend: verify the download's signature.
$ wget https://dl.getkiss.org/kiss-chroot.tar.xz.asc
$ gpg --verify kiss-chroot.tar.xz.asc kiss-chroot.tar.xz
```

Download the ```chroot``` helper
```sh
$ wget https://dl.getkiss.org/kiss-chroot

# inspect the script before you execute it below
$ vi kiss-chroot

# ensure the script is executable
$ chmod +x kiss-chroot
```
Unpack the ```tarbal``` (Install KISS)
```sh
# make sure disks are first mounted to /mnt
$ mount -t ext4 /dev/sdaX /mnt
$ tar xvf kiss-chroot.tar.xz -C /mnt --strip-components 1
```
Enter the ```chroot```
```sh
./kiss-chroot /mnt
```
# Enable repository signing
This step is ***entirely optional*** and can also be done after the installation. See [#60](https://github.com/kisslinux/kiss/issues/60) for more information.

Build and install gnupg1
```sh
$ kiss build gnupg1
$ kiss install gnupg1
```
Import Dylanarap's key
```sh
$ gpg --keyserver keys.gnupg.net --recv-key 46D62DD9F1DE636E
```
Trust Dylanarap's key
```sh
$ echo trusted-key 0x46d62dd9f1de636e >> /root/.gnupg/gpg.conf
```
Go to the system-wide repository
```sh
$ cd /var/db/kiss/repo
```
Enable signature verification
```sh
$ git config merge.verifySignature true
```
# Rebuild KISS
This step is ***entirely optional*** and you can just use the supplied binaries from the downloaded ```chroot```. This step can also be done after the installation.

Modify compiler options (Optional)
```sh
$ export CHOST="x86_64-pc-linux-gnu"
$ export CFLAGS="-march=core-avx2 -mavx2 -O2 -pipe"
$ export CXXFLAGS="${CFLAGS}"
$ export MAKEFLAGS="-j3"

# note: this is for ThinkPad x240
```
Update all base packages to the latest version
```sh
$ kiss update
```
Start rebuilding all packages
```sh
$ kiss build
```
# Build userspace tools
```sh
# required for mounting drives
$ kiss build e2fsprogs eudev

# requires for connecting to WiFi
$ kiss build wpa_supplicant
$ kiss install wpa_supplicant

# required for connecting to the internet
# (wifi and ethernet) (dynamic ip addressing)
$ kiss build dhcp
$ kiss install dhcpcd
```
# Configure and build the kernel
This step involces configuring and building your own Linux kernel. If you have not done this before, below are a few guides to get you started.
* [KernelNewbies](https://kernelnewbies.org/KernelBuild)
* [Gentoo wiki](https://wiki.gentoo.org/wiki/Kernel/Gentoo_Kernel_Configuration_Guide)
* [Linuxtopia](https://www.linuxtopia.org/online_books/linux_kernel/kernel_configuration/index.html)

The Linux kernel is ***not*** managed by the package manager. The kernel is managed manually by the user.

***NOTE***: KISS does not support booting using an ```initramfs```. When configuring your kernel ensure that all required file-system, disk controller and USB drivers are built with [*] (Yes) and ***not***[m] (Module).

# Download the kernel sources
You can find the latest version at [https://kernel.org](https://kernel.org)

```sh
$ wget https://cdn.kernel.org/pub/linux/kernel/v5.x/linux-5.3.6.tar.xz
```
# Extract the kernel sources
```sh
$ tar xvf linux-5.3.6.tar.xz

# change directory to the kernel sources
$ cd linux-5.3.6
```
# Configure the kernel
***NOTE:*** You can determine which drivers you need to enable by Googling your hardware.
***NOTE:*** If you require firmware blobs, the drivers you enable must be enabled as [m] (modules). You can also optionally include the firmware in the kernel itself.

```sh
# install 'linux-firmware' if you require firmware
# blobs for your hardware
$ kiss build linux-firmware
$ kiss install linux-firmware

# generate a default config with *most* drivers
# compiled as `[*]` (not modules)
$ make defconfig

# open an interactive menu to edit the generated
# config, enabling everything extra you may need.
$ make menuconfig

# store the generated config for reuse later
$ cp .config /path/to/somewhere
```

Here's my kernel config that i used for ```ThinkPad x240```

```sh
Processor type and features  --->
  Processor family (Intel Core AVX2)
  <*> CPU microcode loading support
  [*] Intel microcode loading support
Power management and ACPI options  --->
  [*] ACPI (Advanced Configuration and Power Interface) Support  --->
    <*> Battery
    <*> Video
  CPU Frequency scaling  --->
    Default CPUFreq governor (performance)
    <*> 'performance' governor
    x86 CPU frequency scaling drivers  --->
      [*] Intel P state control
[*] Networking support  --->
  <*> Bluetooth subsystem support  --->
    Bluetooth device drivers  --->
      <*> HCI USB driver
  <*> Wireless  --->
    <*> cfg80211 - wireless configuration API
    <*> Generic IEEE 802.11 Networking Stack (mac80211)
  <*> RF switch subsystem support
Device Drivers  --->
  Misc devices  --->
    <*> Intel Management Engine Interface
    <*> ME Enabled Intel Chipsets
  SCSI device support  --->
    <*> SCSI disk support
    [*] Probe all LUNs on each SCSI device
  <*> Serial ATA and Parallel ATA drivers  --->
    <*> AHCI SATA support
  [*] Network device support  --->
    [*] Ethernet driver support  --->
      [*] Intel devices
        <*> Intel(R) PRO/1000 PCI-Express Gigabit Ethernet support
    [M] Wireless LAN  --->
      <M> Intel Wireless WiFi Next Gen AGN - Wireless-N/Advanced-N/Ultimate-N (iwlwifi)
      <*> Intel Wireless WiFi MVM Firmware support
  <*> I2C support  --->
    I2C Hardware Bus support  --->
      <*> Intel 82801 (ICH/PCH)
  <*> Hardware Monitoring support  --->
    <*> Intel Core/Core2/Atom temperature sensor
  [*] Watchdog Timer Support  --->
    <*> Intel TCO Timer/Watchdog
  Multifunction device drivers  --->
    <*> Intel ICH LPC
    <*> Realtek PCI-E card reader
  <*> Multimedia support  --->
    [*] Media USB Adapters  --->
      <*> USB Video Class (UVC)
  Graphics support  --->
    <*> Direct Rendering Manager (XFree86 4.1.0 and higher DRI support)
    <*> Intel 8xx/9xx/G3x/G4x/HD Graphics
      [*] Enable modesetting on intel by default
      [*] Enable legacy fbdev support for the modesettting intel driver
    Console display driver support  --->
      <*> Framebuffer Console support
  <*> Sound card support  --->
    <*> Advanced Linux Sound Architecture  --->
      [*] PCI sound devices  --->
        <*> Intel HD Audio  --->
          [*] Build HDMI/DisplayPort HD-audio codec support
  [*] USB support  --->
    <*> xHCI HCD (USB 3.0) support
    <*> EHCI HCD (USB 2.0) support
  <*> MMC/SD/SDIO card support
    <*> Realtek PCI-E SD/MMC Card Interface Driver
  [*] X86 Platform Specific Device Drivers  --->
    <*> ThinkPad ACPI Laptop Extras
  [*] IOMMU Hardware Support  --->
    [*] Support for Intel IOMMU using DMA Remapping Devices
    [*]  Enable Intel DMA Remapping Devices by default
    [*] Support for Interrupt Remapping
[*] Cryptographic API  --->
  <*> AES cipher algorithms (AES-NI)
```
```sh
Device Drivers ->
  Character devices ->
    <*> TPM Hardware Support ->
      <*> TPM Interface Specification 1.2 Interface / TPM 2.0 FIFO Interface
```
# Build the kernel
```sh
# '-j $(nproc)' does a parallel build using all cores
$ make -j "$(nproc)"
```
# Install the kernel
***NOTE:*** Ignore the LILO error, it's harmless.

```sh
# install the built modules
# this installls directly to ```/lib``` (symlonk to ```/usr/lib```).
$ make modules_install

# install the built kernel
# this install directly to ```/boot```
$ make install

# rename the kernel
#  substitute VERSION for the kernel version you have built
$ mv /boot/vmlinuz /boot/vmlinuz-KERNELVERSION
$ mv /boot/System.map /boot/System.map-KERNELVERSION
```
# Install grub
Build and install grub

```sh
$ kiss build grub
$ kiss install grub

# also needed for UEFI.
$ kiss build efibootmgr
$ kiss install efibootmgr
```
Setup grub

```sh
# BIOS
$ gtub-install --target=i386-pc /dev/sdX
$ grub-mkconfig -o /boot/grub/grub.cfg

# UEFI
# replace 'X' with its mount point
$ grub-install --target=x86_64-efi --efi-directory=/path/to/mounted/efi --bootloader-id=GRUB
$ grub-mkconfig -o /boot/grub/grub.cfg
```
# Install init scripts
This is the final "mandatory" step.

```sh
$ kiss build baseinit
$ kiss install baseinit
```
# Enable the community repository
The KISS community repository is maintained by users of the distribution and contains packages which aren't int he main repositories. This repository is disabled by default since it's not maintained by the KISS developers.

```sh
# clone the repository to a location of your choosing.
$ git clone https://github.com/kisslinux/community.git

# add the repository to the system-wide 'KISS_PATH'.
# the 'KISS_PATH' variable works exactly like 'PATH'.
# each repository is split by ':' and is checked in
# the order they're written.
#
# add the full path to the repository you cloned
# above.
#
# NOTE: The subdirectory must also be added.
# example: export KISS_PATH=/var/db/kiss/repo/core:/var/db/kiss/repo/extra:/var/db/kiss/repo/xorg:/path/to/community/community
$ vi /etc/profile.d/kiss_path.sh

# spawn a new login shell to access this repository
# immediately.
$ sh -l
```
---
Sreenshot

![KISS OpenBox](https://camo.githubusercontent.com/7a060a9d0cea23574b166f912f8649519124c507/68747470733a2f2f692e696d6775722e636f6d2f567234656f32782e6a7067)