---
tags:
  - Feodra
  - Install
  - Guides
  - Workstation
  - Linux
  - NVIDIA
share_link: https://share.note.sx/i0x8yr6d#JKm0Er+dGa68nZ7GHgzpHK/MSlsTx21P7pCH2ZpTDDA
share_updated: 2024-08-26T20:08:11-04:00
---

![Photo | center | 256 ](../../../fedora.png) 


> [!NOTE] System Requirements
> System Requirements
> To have a good experience, hardware with a 40GB SSD disk and 4GB RAM is recommended for Fedora Workstation. However, it is possible to use hardware with lower specifications.
> 
> Workstation images are available for x86_64 and aarch64 architectures, providing support for Intel, AMD and ARM processors.


# Download and Create A Bootable Drive
[Official Documentation](https://docs.fedoraproject.org/en-US/fedora/latest/preparing-boot-media/)

1: Download [Fedora 40 Workstation](https://fedoraproject.org/workstation/download)
2: Check Download Automatically
3: Run Fedora Media Writer
4: Select Fedora 40 Workstation:
5: Additional [Fedora Spins](https://fedoraproject.org/spins/) or you can use the  [Nabora](https://nobaraproject.org/download-nobara/) spin
6: Select your USB Drive

## BIOS Time

> [!WARNING] DISABLE SECURE BOOT
> There are other ways to do this with secure boot, but we will do it without for simplicity sake 

[ASUS Instructions](https://www.asus.com/support/faq/1049829/): Look up instructions as per your motherboard Vendor/Model

You will be sent to the 

7: Change your boot order for USB DRIVE to be first
8 Save and Exit (*This should initiate a restart*)
9: Select `Start Fedora Workstation 40`
10: You will be brought to the GUI Bootloader, Select `Install Fedora`
11: Select your language
12: Select the install destination

> [!NOTE] RAID Configurations
> If you want to configure a software raid see [here](https://docs.fedoraproject.org/en-US/fedora/f36/install-guide/install/Installing_Using_Anaconda/#sect-installation-gui-manual-partitioning-swraid)

13: Ensure the dive you want to install on has a check mark on it. `Automatic is checked`
14: Confirm Keyboard Layout
15: Confirm Time & Date
16: Click Begin Installation
17: You are on your way to Greatness!
18: INSTALL COMPLETE!
19: Click the power button in the top right and restart
20: It is safe to remove your USB Drive at this point. Welcome to Fedora

Hit your SUPER key (WIN Key) and click on `Terminal` application. If you do not see it, just start typing `Terminal` and select it. Ensure Secure boot is disabled

> [!WARNING] Ensure Secureboot is diabled once loaded into Fedora
> 	
> ```bash
sudo dmesg | grep -i secure
[sudo] password for usr: 
[    0.000000] secureboot: Secure boot disabled
[    0.003288] secureboot: Secure boot disabled

Open up Text Editor
Copy the Below code and paste it into text editor

> [!NOTE] You can edit the script to your liking
>  Especially the flatpaks section. You can view available flatpaks from [flathub](https://flathub.org/)

Click the `...` in the top left Save file as `install-fedora-nvidia.sh`
Save it to your documents
Hit your SUPER key and go to Files Documents
Right Click in **empty space** and select `Open in Terminal`
type `sudo chmod +x ./install-fedora-nvidia.sh`
hit `<enter>`
Type `sudo ./install-fedora-nvidia.sh`

Let the script run.

You will be prompted to select versions for `OBSVkCapture` in which you can select the most recent version `[3]` *at the time of writing this*

hit `y` for any questions asking for a `y|n` response

> [!IMPORTANT] DO NOT RESTART AFTER IT IS COMPLETE!

Verify your nvidia drivers are built in the terminal using:
`nvidia-smi` or `lsmod | grep nvidia`
If you are getting a communication error with `nvidia-smi` go back to your bios and disable secure boot.

Once a version number is populated, reboot.

Enjoy Fedora Linux
# install-fedora-nvidia.sh
```bash
#!/bin/bash

# Set Downloads in DNF
printf "%s" "
fastestmirror=1
max_parallel_downloads=10
countme=false
" | sudo tee -a /etc/dnf/dnf.conf

# Install RPM Fusion Free Repositories
sudo dnf install -y https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm

# Install app stream meta-data
sudo dnf group update core -y

#Update and Upgrade
sudo dnf -y update
sudo dnf -y upgrade --refresh

# GNOME SETTINGS
# Setting Gnome Menu to stay true during scaling
gsettings set org.gnome.desktop.a11y always-show-universal-access-status true
# Setting minimize, maximize, and close buttons for windows
gsettings set org.gnome.desktop.wm.preferences button-layout 'appmenu:minimize,maximize,close'
# Setting clock to show weekday on gnome interface
gsettings set org.gnome.desktop.interface clock-show-weekday true
# Setting clock to show seconds on gnome interface
gsettings set org.gnome.desktop.interface clock-show-seconds true
# Enabling scaling
gsettings set org.gnome.mutter experimental-features "['scale-monitor-framebuffer']"

#Firmware Updates
sudo dnf autoremove -y
sudo fwupdmgr refresh --force
sudo fwupdmgr get-devices
sudo fwupdmgr get-updates -y
sudo fwupdmgr update -y

#Add Flatpak Repositories
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo

# Nvidia Drivers
sudo dnf -y update
sudo dnf install -y akmod-nvidia
sudo dnf install -y xorg-x11-drv-nvidia-cuda 
sudo dnf install -y xorg-x11-drv-nvidia-cuda-libs svt-hevc svt-av1 svt-vp9 nvidia-vaapi-driver libva-utils

#Media Codecs
sudo dnf swap 'ffmpeg-free' 'ffmpeg' --allowerasing
sudo dnf update @multimedia --setopt="install_weak_deps=False" --exclude=PackageKit-gstreamer-plugin
sudo dnf update @sound-and-video
sudo dnf group install Multimedia

# Hardware accelleration
sudo dnf install ffmpeg ffmpeg-libs libva libva-utils
sudo dnf config-manager --set-enabled fedora-cisco-openh264
sudo dnf install -y openh264 gstreamer1-plugin-openh264 mozilla-openh264

# Install Flatpaks --- https://flathub.org/
mokey_flathub () {
	log "mokey_flathub"
	local -a mokey_flathub_install
	mokey_flathub_install=(
	"com.github.finefindus.eyedropper"
	"com.github.tchx84.Flatseal"
	"com.github.wwmm.easyeffects"
	"com.obsproject.Studio"
	"com.obsproject.Studio.Plugin.OBSVkCapture"
	"com.visualstudio.code"
	"com.valvesoftware.Steam"
	"net.lutris.Lutris"
	"network.loki.Session"
	"org.blender.Blender"
	"org.freedesktop.Platform.VulkanLayer.MangoHud"
	"org.freedesktop.Platform.VulkanLayer.OBSVkCapture"
	"org.gnome.World.PikaBackup"
	"org.mozilla.Thunderbird"
	"org.pipewire.Helvum"
	"org.signal.Signal"
	"md.obsidian.Obsidian"
	"com.jgraph.drawio.desktop"
	"org.gimp.GIMP"
	"org.videolan.VLC"
	"io.github.shiftey.Desktop"
	"org.nmap.Zenmap"
	"org.remmina.Remmina"
	"com.teamspeak.TeamSpeak"
	"dev.vencord.Vesktop"
	"com.spotify.Client"
#	---- Optional, Already installed non-flatpaks. Remove RPMs prior to installing---- 
#	"org.getmonero.Monero"
#	"org.gnome.Boxes"
#	"org.gnome.Calculator"
#	"org.gnome.Calendar"
#	"org.gnome.Characters"
#	"org.gnome.Connections"
#	"org.gnome.Contacts"
#	"org.gnome.Evince"
#	"org.gnome.Extensions"
#	"org.gnome.font-viewer"
#	"org.gnome.Logs"
#	"org.gnome.Loupe"
#	"org.gnome.Maps"
#	"org.mozilla.firefox"
)
flatpak install -y flathub ${mokey_flathub_install[*]}
}
mokey_flathub

#verify Nvidia cuda is working
sudo nvidia-smi

echo "if nvidia-smi shows no information, wait up to 5 minutes to validate:"
echo "run nvidia-smi"
echo "or"
echo "lsmod | grep nvidia"

echo "after validation, reboot"
```

Sources
devangshekhawat [Fedora-40-Post-Install-Guide](https://github.com/devangshekhawat/Fedora-40-Post-Install-Guide?tab=readme-ov-file)
Trafotin [os-install-scripts/fedoraworkstation](https://gitlab.com/trafotin/os-install-scripts/-/blob/main/fedora-workstation.sh)
[rpm fusion guide ](https://rpmfusion.org/Howto/NVIDIA)
[Guide to Gaming on Fedora](https://web.archive.org/web/20201107231159/https://minus1over12.theoutsiders.stream/linux/fedora/guide-to-gaming-on-fedora-linux/)
Special Thanks to Fedora Linux Discord @imshubhammisc for all the technical help
Special Thanks to Fedora Linux Discord @Rhea for putting the #Guides together
