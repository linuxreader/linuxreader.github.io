+++ 
type = " "
categories = ["explanation"] 
description = "My Fedora Setup" 
title = "My Fedora Setup" 
weight = 1 
+++
When you first download Fedora Workstation, it's going to be a little hard to figure out how to make it usable. Especially if you've never tinkered with Linux before.

This is because Fedora with Gnome desktop is a blank canvas. The point is to let you customize it to your needs. When I first install Fedora, I pull my justfile to install most of the programs I use:

	`curl -O https://raw.githubusercontent.com/davidvargasxyz/dotfiles/main/dot_justfile >> ~/.justfile`

## Install just and run the justfile

To run the just file, I then install the just program and run it on the justfile:

`dnf install just`

`just ~/.justfile`

This is my current .justfile:
```bash
first-install:
# Install flatpacks
	flatpak install --noninteractive \
      flathub com.bitwarden.desktop \
      flathub com.brave.Browser \
      flathub com.discordapp.Discord \
      flathub org.gimp.GIMP \
      flathub org.gnome.Snapshot \
      flathub org.libreoffice.LibreOffice \
      flathub org.remmina.Remmina \
      flathub com.termius.Termius \
      flathub net.devolutions.RDM \
      flathub com.slack.Slack \
      flathub org.keepassxc.KeePassXC \
      flathub md.obsidian.Obsidian \
      flathub com.calibre_ebook.calibre \
      flathub org.mozilla.Thunderbird \
      flathub us.zoom.Zoom \
      flathub org.wireshark.Wireshark \
      flathub com.nextcloud.desktopclient.nextcloud \
      flathub com.google.Chrome \
      flathub io.github.shiftey.Desktop \
      flathub io.github.dvlv.boxbuddyrs \
      flathub com.github.tchx84.Flatseal \
      flathub io.github.flattool.Warehouse \
      flathub io.missioncenter.MissionCenter \
      flathub org.gnome.World.PikaBackup \
      flathub com.github.rafostar.Clapper \
      flathub com.mattjakeman.ExtensionManager \
      flathub com.jgraph.drawio.desktop

# Install Homebrew
    sudo dnf -y groupinstall \
      "Development Tools"

    /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
    echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> ~/.bash_profile
    eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"

# Configure dnf for faster speeds
    sudo bash -c 'echo "fastestmirror=True" >> /etc/dnf/dnf.conf'
    sudo bash -c 'echo "max_parallel_downloads=10" >> /etc/dnf/dnf.conf'
    sudo bash -c 'echo "defaultyes=True" >> /etc/dnf/dnf.conf'
    sudo bash -c 'echo "keepcache=True" >> /etc/dnf/dnf.conf'

# Other software, updates, etc. 
    sudo dnf -y update
    sudo dnf install -y gnome-screenshot
    sudo dnf -y groupupdate core
    sudo flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
    sudo dnf install -y wireguard-tools
    sudo dnf install gnome-tweaks
    sudo dnf -y install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm \
    https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
    sudo dnf -y update
    sudo dnf install gnome-themes-extra
    gsettings set org.gnome.desktop.interface gtk-theme "Adwaita-dark"
    sudo dnf install -y go
    echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
    source ~/.bashrc
    gsettings set org.gnome.mutter experimental-features "['scale-monitor-framebuffer']"
```

## Virt Manager
```bash
sudo dnf install @virtualization
```

```bash
sudo vi /etc/libvirt/libvirtd.conf
```

uncomment: `unix_sock_group = "libvirt"`

Adjust the UNIX socket permissions for the R/W socket

`unix_sock_rw_perms = "0770"`

Start the service `systemctl enable --now libvirtd`

Add user to group:
```bash
sudo usermod -a -G libvirt $(whoami) && sudo usermod -a -G kvm $(whoami)
```


# Configure Howdy 

```bash
sudo dnf copr enable principis/howdy
sudo dnf --refresh install -y howdy
```
https://copr.fedorainfracloud.org/coprs/principis/howdy/
https://github.com/boltgolt/howdy


# Seahorse
To fix Login Keyring error
https://itsfoss.com/seahorse/
`sudo dnf -y install seahorse && seahorse`

Applications > Passwords and Keys > Passwords > Right-click Login > Change Password to blank

# then run `just homebrew` after a reboot to install packages with brew
homebrew:
    brew install \
      chezmoi \
      onedrive \
      hugo \
      virt-manager

## Initialize Chezmoi

Chezmoi let's you easy sync your dotfiles with github and your other computers. Just init chezmoi and add your github ussername. This assumes your dotfiles in github are saved in the proper format. 

`chezmoi init --apply davidvargasxyz`

## Add badname user (if needed)

This is just here

`$ adduser --badname firstname.lastname`
`$ sudo usermod -aG wheel username`

```
# uncomment this line in the visudo file 
$ sudo visudo
%wheel ALL=(ALL) ALL
```

Delete other user:

`$ userdel username`

## DNF Config (for speed)

DNF is one of the package managers that comes with Fedora. You use it to install most software and applications.

The official doc is [Here](https://dnf.readthedocs.io/en/latest/conf_ref.html#description](https://dnf.readthedocs.io/en/latest/conf_ref.html#description "https://dnf.readthedocs.io/en/latest/conf_ref.html#description")
Just open up your DNF config file with your favorite text editor.

```
sudo vim /etc/dnf/dnf.conf
```

Then add the following:

```
fastestmirror=True  
max_parallel_downloads=10  
defaultyes=True  
keepcache=True
```

## Clear cache (do this occasionally)

`sudo dnf clean dbcache`
or `sudo dnf cleanall`

## Update DNF

`sudo dnf -y update`

## DNF commands

[https://docs.fedoraproject.org/en-US/fedora/latest/system-administrators-guide/package-management/DNF/](https://docs.fedoraproject.org/en-US/fedora/latest/system-administrators-guide/package-management/DNF/ "https://docs.fedoraproject.org/en-US/fedora/latest/system-administrators-guide/package-management/DNF/")

## RPM Fusion

More accessibility to various software packages.

[https://rpmfusion.org/](https://rpmfusion.org/ "https://rpmfusion.org/")

Fedora with dnf:

```
sudo dnf -y install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```

## AppStream metadata

to enable users to install packages using Gnome Software/KDE Discover

```
sudo dnf -y groupupdate core
```

flatpak

lhttps://flatpak.org/setup/Fedora

```
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

## set a hostname

`sudo hostnamectl set-hostname "New_Custom_Name"`
this will show after next reboot

## Make it yours

in the software center

## gnome tweaks

add minimize and maximize

## Flatseal

![](Pasted%20image%2020240905105809.png)

## Input Leap

![](Pasted%20image%2020240905105932.png)

sudo dnf install git cmake make gcc-c++ xorg-x11-server-devel \
                 libcurl-devel avahi-compat-libdns_sd-devel \
                 libXtst-devel qt5-qtbase qt5-qtbase-devel  \
                 qt5-qttools-devel libICE-devel libSM-devel \
                 openssl-devel libXrandr-devel libXinerama-devel

## Virtual machine Manager

![](Pasted%20image%2020240905110024.png)

## Box Buddy

![](Pasted%20image%2020240905110223.png)
## Warehouse

![](Pasted%20image%2020240905110143.png)

## Mission Center 

![](Pasted%20image%2020240905110500.png)

## Pika Backup

![](Pasted%20image%2020240905110555.png)
## Clapper

![](Pasted%20image%2020240905110319.png)

## Extentions Manager (blue puzzle piece)

![](Pasted%20image%2020240905105624.png)

![](Pasted%20image%2020240905105517.png)


s.gnome.org/extension/1112/screenshot-tool/](https://extensions.gnome.org/extension/1112/screenshot-tool/ "https://extensions.gnome.org/extension/1112/screenshot-tool/")

ran gnome-screenshot tool in command prompt and it asked to install

## Airpods not pairing Issue

remove from the pairing list, force them in pairing mode and pair them back.

This can be made easy with bluetoothctl.

1/ Just in case, restart the bluetooth service:

```
$ sudo systemctl restart bluetooth
$ systemctl status bluetooth 
```

2/ bluetoothctl is easier than the UI imo

```
$ bluetoothctl
[bluetooth] $ devices
Device 42:42:42:42:42:42 My AirPods <-- grab the name here
[bluetooth] $ remove 42:42:42:42:42:42 
```

Now, make sure your airpods are in the charging case, close the lid, wait 15 seconds, then open the lid. Press and hold the setup button on the case for up to 10 seconds. The status light should flash white, which means that your AirPods are ready to connect

3/ Pair them back

```
[bluetooth] $ pair 42:42:42:42:42:42 <--- add the same device from earlier 
```

## Enable Fractional Scaling

This lets you change display scaling in smaller increments.
Make sure Wayland is turned on >

run:
`gsettings set org.gnome.mutter experimental-features "['scale-monitor-framebuffer']"`

then Reboot

## Remove pasted characters ^[](200~')
`vim ~/.inputrc`

```bash
"\C-v": ""
```

## Install gimp

$ sudo dnf install gimp
enable in screenshot tool after install for gimp {f}

Install Arrows
https://graphicdesign.stackexchange.com/questions/44797/how-do-i-insert-arrows-into-a-picture-in-gimp

```
Go to your home folder
Go to .config/GIMP
Go to the folder with a version number (2.10 for me)
Go to scripts
Download the arrow.scm file and place it here. Don't forget to unzip.
Open GIMP and draw a path
```

From Tools menu, select Arrow

= h.265 main 10 profile media codec error =


## Distrobox

See [distrobox](distrobox.md) 

## ZSH For Humans

[GitHub - romkatv/zsh4humans: A turnkey configuration for Zsh](https://github.com/romkatv/zsh4humans)

## Install Starcraft on Fedora

https://www.youtube.com/watch?v=eefsL9K2w4k

 1. Install your latest gpu driver https://github.com/lutris/docs/blob/master/InstallingDrivers.md

I am just running off of built in AMD graphics. So we just need to install support for Vulkan API
`sudo dnf install vulkan-loader vulkan-loader.i686`


Install Wine
`$ sudo dnf -y install wine`

Install Lutris
Install the Flatpak version in software center.

## Fedora Hotkeys

## Terminal

Close Terminal
shift + c + q

Previous Tab
c + Page Up

Next Tab
c + Page Down

Move to Specific Tab
Alt + \#

Full Screen
F11

New Window
Shift + Ctrl + t

Close Tab
Shift + Ctrl + w


## Desktop

Run a command
super + F2

Switch Between Applications
Alt + Esc

Move Window to Left Monitor
Shift + Super + <- 

Move Window to Right Monitor
Shift + Super + ->

Minimize Current Window
Super + H

Close Current Appllication
Ctrl + Q

## Browser

### Firefox

Switch Between Tabs
Ctrl + Tab

Switch Between Tabs in Reverse
Ctrl + Shift + Tab

#### Detach Tab Extension
https://addons.mozilla.org/en-US/firefox/addon/detach-tab/

Detach Tab
Ctrl _ Shift _ Space

Reattach Tab
Ctrl + Shift + v

## Slack

Installing via package manager because of screen sharing issue. 

Upgrade dnf and download the slack rpm from the website. 

**Screen Sharing in Slack:**

```bash
vim /usr/share/applications/slack.desktop
```

Update the exec line to:
```bash
Exec=/usr/bin/slack --enable-features=WebRTCPipeWireCapturer %U
```

## Actual Budget

https://github.com/actualbudget/actual
