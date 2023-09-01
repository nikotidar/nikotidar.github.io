---
layout: post
title: "How To Fix `unable to lock database` Arch Linux"
author: "rndtx"
---

The other day, I was about to update my Arch Linux repository. When I issued the following command to update Arch Linux repository, it showed error, and won't let me to install any package or remove any package

```sh
$ pacman -Syu
```

Sample output

```sh
:: Synchronizing package databases...
error: failed to update core (unable to lock database)
error: failed to update extra (unable to lock database)
error: failed to update community (unable to lock database)
error: failed to update multilib (unable to lock database)
error: failed to synchronize any databases
error: failed to init transaction (unable to lock database)
error: could not lock database: File exists
 if you're sure a package manager is not already
 running, you can remove /var/lib/pacman/db.lck
 ```

The beauty of Linux distros is sometimes they will the explicitly display the solution along with the error message. As you can see in the above output, it says: **“..you can remove /var/lib/pacman/db.lck”**.

So, I simply deleted aforementioned file with command:

```sh
$ sudo rm /var/lib/pacman/db.lck
```

Voila! It worked. I can then able to update, install and remove without any problems.

```sh
$ sudo pacman -Syu
```

Sample output

```sh
:: Synchronizing package databases...
 core is up to date
 extra is up to date
 community is up to date
 multilib is up to date
 endeavouros is up to date
:: Starting full system upgrade...
resolving dependencies...
looking for conflicting packages...

Packages (64) alsa-lib-1.2.3.2-1  cdrtools-3.02a09-3  code-1.47.0-1  electron7-7.1.14-6  endeavouros-theming-5-2.2  firefox-78.0.2-1  fribidi-1.0.10-1  gjs-2:1.64.4-1
              gnome-boxes-3.36.5-1  gnome-clocks-3.36.2-2  gnome-control-center-3.36.4-1  gnome-desktop-1:3.36.4-1  gnome-multi-writer-3.32.1-2  gnome-music-1:3.36.4.1-1
              gnome-packagekit-3.32.0-2  gnome-shell-1:3.36.4-1  gnome-usage-3.33.2-2  grub-tools-1.3-1  hardinfo-0.5.1.816.g877ea2b-2  hidapi-0.9.0-2  i3lock-2.12-2
              i3status-2.13-3  jq-1.6-3  js68-68.10.0-1  jshon-20131105-4  keyutils-1.6.3-1  lib32-libpng-1.6.37-3  libconfig-1.7.2-3  libfdk-aac-2.0.1-2  libgcrypt-1.8.6-1
              libiptcdata-1.0.4-5  liblrdf-0.6.1-4  libmfx-20.2.0-1  libnfs-4.0.0-4  libopenraw-0.1.3-2  libosinfo-1.8.0-1  libpng-1.6.37-3  libsodium-1.0.18-2
              libutempter-1.2.1-1  lsb-release-1.4-16  lxappearance-0.6.3-3  lxinput-gtk3-0.3.5-3  mesa-20.1.3-1  mutter-3.36.4-1  ndisc6-1.0.4-2  os-prober-1.77-2
              osinfo-db-20200529-1  php-7.4.8-2  php-gd-7.4.8-2  plank-0.11.89-2  python-sphinx-3.1.2-1  re2-1:20200706-1  sdl2-2.0.12-2  simple-scan-3.36.4-1
              sqlitebrowser-3.11.2-2  thunderbird-extension-enigmail-2.1.7-3  ttf-ubuntu-font-family-0.83-6  usbredir-0.8.0-2  webkit2gtk-2.28.3-1  wmctrl-1.07-6
              wvdial-1.61-8  xl2tpd-1.3.15-2  xterm-357-1  zeromq-4.3.2-2

Total Download Size:     5.59 MiB
Total Installed Size:  820.45 MiB
Net Upgrade Size:       -0.78 MiB
```
