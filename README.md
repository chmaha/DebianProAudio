# A Pro Audio Tuning Guide for Debian (and other Debian-based distros)

Following this guide will allow you to get the best possible performance on Linux for professional audio needs. Even though these steps are well-tested, it is wise to research what each step accomplishes and why (the search engine is your friend :P ). See also https://wiki.archlinux.org/title/Professional_audio. 

For the Arch-based guide see https://github.com/chmaha/ArchProAudio.

## Fundamentals

To get started after installing Ubuntu, you could try just steps 2, 4 and 6 below. If you need to use windows plugins on Linux also follow step 12 (easy: wine-staging, more advanced but potentially more performance: wine-tkg). Based on your individual pro audio needs, workflows, hardware specifications and more, your mileage may vary. If you are still having audio performance issues, try following the full guide...

### Pipewire?

In short, no, don't do it if you are a pro audio user. I don't believe it's ready for primetime. In the near future I'll provide some commands to help remove pipewire audio and replace with ALSA + Pulseaudio + JACK. 

## Full In-depth Guide

### 1. Install a flavor of Ubuntu (or other favorite Ubuntu-based or Debian-based distro)

To make your life easier, install either Ubuntu Studio or AVLinux. Almost all of the following tweaks are taken care of. Otherwise, pick a regular distro such as Ubuntu, MXLinux, Debian etc.

### 2. Install a low-latency kernel (Ubuntu-based)

```shell
sudo apt update && sudo apt upgrade -y
sudo apt install linux-lowlatency
reboot
```
Or, for even better performance:

#### Liquorix (Debian-based or Ubuntu-based)

```shell
sudo apt update && sudo apt upgrade -y
sudo apt install software-properties-common apt-transport-https wget ca-certificates gnupg2 ubuntu-keyring -y
sudo add-apt-repository ppa:damentz/liquorix -y
sudo apt update
sudo apt-get install linux-image-liquorix-amd64 linux-headers-liquorix-amd64 -y
reboot
```

### 3. rtcqs (formerly known as realtimeconfigquickscan)
    
```shell
git clone https://codeberg.org/rtcqs/rtcqs.git
cd rtcqs
./src/rtcqs/rtcqs.py
```

### 4. Add user to audio group and configure realtime privileges

I believe that installing jackd2 takes care of the following these days. It is always worth double-checking especially if using pipewire.

```shell
sudo nano /etc/security/limits.d/audio.conf
```
Add the following lines:

```shell
@audio   -  rtprio     95
@audio   -  memlock    unlimited
```

Then create an audio group (if it doesn't exist already) and add your user to it:

```shell
sudo groupadd audio 
sudo usermod -a -G audio $USER
```

Log out/in or reboot...

### 5. Add "threadirqs" as kernel parameter

```shell
sudo nano /etc/default/grub
```
change 
`GRUB_CMDLINE_LINUX=""` to `GRUB_CMDLINE_LINUX="threadirqs"`
    
```shell
sudo update-grub
```

### 6. Set governor to "performance"

i. Temporary:

```shell
sudo cpupower frequency-set -g performance
```
ii. Permanent:

Add `cpufreq.default_governor=performance` as a kernel parameter:

```shell
sudo nano /etc/default/grub
```
LIne should now read: 

`GRUB_CMDLINE_LINUX="cpufreq.default_governor=performance threadirqs"`

```shell
sudo update-grub
```
or, for kernels < 5.9:

```shell
sudo nano /etc/default/cpupower # uncomment governor and change to performance
systemctl enable --now cpupower.service
systemctl start cpupower.service
```
    
### 7. Swappiness

```shell
sudo nano /etc/sysctl.d/99-sysctl.conf
```
add "vm.swappiness=10"
    
###  8. Spectre/Meltdown Mitigations

If you run `rtcqs.py` and it gives you a warning about Spectre/Meltdown Mitigations, you could add `mitigations=off` to GRUB_CMDLINE_LINUX. Warning: disabling these mitigations will make your machine less secure! https://wiki.linuxaudio.org/wiki/system_configuration#disabling_spectre_and_meltdown_mitigations

### 9. Install udev-rtirq

```shell
git clone https://github.com/jhernberg/udev-rtirq.git
cd udev-rtirq
sudo make install
reboot
```

### 10. Jack2 + Jack D-Bus (__skip this step if you switched to Pipewire__)

```shell
sudo apt install qjackctl jackd2
```
Enable Jack D-Bus interface:  
![image](https://user-images.githubusercontent.com/79659262/124497122-51218300-ddb2-11eb-8cb8-4bf873e026cd.png)


### 11. DAW & Plugins

REAPER: 
http://reaper.fm/download.php 

change RT priority to 40 on audio device page?  


Also be sure to check out Bitwig Studio, Tracktion Waveform, Qtractor, LMMS, Rosegarden, Zrythm etc...
https://en.wikipedia.org/wiki/List_of_Linux_audio_software#Digital_audio_workstations_(DAWs)

#### Native plugins
- airwindows-git (http://www.airwindows.com/)  
- lsp-plugins  (https://lsp-plug.in/)
- zam-plugins  (http://www.zamaudio.com/?p=976)
- distrho-ports (https://distrho.sourceforge.io/ports.php)
- dpf-plugins (https://distrho.sourceforge.io/plugins.php)
- ElephantDSP Room Reverb (https://www.elephantdsp.com/)
- dragonfly-reverb (https://michaelwillis.github.io/dragonfly-reverb/)
- Aether (https://dougal-s.github.io/Aether/)
- Bertom Denoiser (https://www.bertomaudio.com/denoiser.html)
- sfizz / sfizz-git (https://sfz.tools/sfizz/)
- Chowdhury DSP (https://chowdsp.com/products.html)

A brilliant resource for Debian- and Ubuntu-based distros is  https://kx.studio/. Add the repo by downloading and installing the [repo](https://launchpad.net/~kxstudio-debian/+archive/kxstudio/+files/kxstudio-repos_11.1.0_all.deb).

### 12. Wine-staging or Wine-tkg

Perhaps start with vanilla wine-staging and see how you fare in terms of performance. If your workflows rely heavily on VSTi like Kontakt, you may find better performance with wine-tkg (fsync enabled). 

#### Wine-staging

```shell
sudo apt install wine-staging
```

Or, install a particular version that you know is compatible (in this case v6.14):

```shell
version=6.14
variant=staging
codename=$(shopt -s nullglob; awk '/^deb https:\/\/dl\.winehq\.org/ { print $3; exit }' /etc/apt/sources.list /etc/apt/sources.list.d/*.list)
suffix=$(dpkg --compare-versions "$version" ge 6.1 && ((dpkg --compare-versions "$version" ge 6.17 && echo "-2") || echo "-1"))
sudo apt install --install-recommends {"winehq-$variant","wine-$variant","wine-$variant-amd64","wine-$variant-i386"}="$version~$codename$suffix"
```

To prevent the package being updated:

```shell
sudo apt-mark hold winehq-staging
```
and in case you want to ever re-enable updating:
```shell
sudo apt-mark unhold winehq-staging
```
Check https://github.com/robbert-vdh/yabridge#tested-with for up-to-date info.

OR...for the more adventurous:
   
#### Wine-tkg
   
Either download a wine-tkg build from https://github.com/Frogging-Family/wine-tkg-git/actions/workflows/wine-arch.yml or follow the instructions to git clone and install latest version: https://github.com/Frogging-Family/wine-tkg-git/tree/master/wine-tkg-git#quick-how-to-

If using wine-tkg, set the WINEFSYNC environment variable to 1 according to https://github.com/robbert-vdh/yabridge#environment-configuration (depends on your display manager and login shell)
       
### 13. Install yabridge

i. Please follow the instructions at https://github.com/robbert-vdh/yabridge#usage

To begin, download the latest release from https://github.com/robbert-vdh/yabridge/releases and run:

```shell
tar -C ~/.local/share -xavf yabridge-x.y.z.tar.gz
```

where x.y.z is the version number such as 4.0.1. Don't forget to add yabridgectl to your shell's search path by adding `export PATH="$PATH:$HOME/.local/share/yabridge` to the end of ~/.bashrc. Close then re-open the terminal.

ii. Configure yabridge according to https://github.com/robbert-vdh/yabridge#readme  

iii. Install Windows VST2, VST3 and CLAP plugins  
    
### 14. Check volume levels!

Once everything is set up, don't forget to check that volume levels are set correctly. Whether using pipewire-alsa, pipewire-jack, vanilla ALSA or JACK, run
```
alsamixer
```
to check that output is set to 100 (vertical bars) or gain of 0dB (top left of alsamixer). Use F6 to select the correct soundcard. You can also use your desktop environment's volume controls if you have your interface enabled there but note that numbers don't seem to match alsamixer.

![alsamixer](https://user-images.githubusercontent.com/120390802/209148828-f5654838-eb25-4dd2-9955-4e0e8db99be2.png)

### 15. Other useful tools (all available via the package manager)

**Music Player**: strawberry (can produce bit-perfect playback) <br>
![image](https://user-images.githubusercontent.com/120390802/209885065-970b9ee3-f4ae-48c2-a0d5-a6f52204e682.png) <br>
**Tagger**: kid3, picard <br>
**DDP creation/verification/etc**: ddptools <br>
**Audio Converter**: fre:ac or soundconverter <br>
**CD Ripper**: asunder or cdrdao <br>
**CD Burner**: cdrdao, k3b or nerolinux4

