# A Pro Audio Tuning Guide for Debian (and other Debian-based distros)
<p align="center">
  <img src="https://github.com/chmaha/DebianProAudio/assets/120390802/e2be6a8e-5462-46d2-ac8d-1b2ebd135ea0">
</p>

**UPDATE: I'm archiving this guide because I no longer use Debian. Much of my [Arch guide](https://github.com/chmaha/ArchProAudio) is applicable simply by switching out the package manager commands. I also include an introductory note for users of other distros for adding the `preempt=full` kernel parameter and manually setting up realtime privileges.** 

Following this guide will allow you to get the best possible performance on Linux for professional audio needs. Even though these steps are well-tested, it is wise to research what each step accomplishes and why (the search engine is your friend :P ). See also https://wiki.linuxaudio.org/wiki/system_configuration and https://wiki.archlinux.org/title/Professional_audio. 

## Fundamentals

To get started after installing Debian, you could try just steps 3 & 4 below. If you need to use windows plugins on Linux also follow step 12 (easy: wine-staging, more advanced but potentially more performance: wine-tkg). Based on your individual pro audio needs, workflows, hardware specifications and more, your mileage may vary. If you are still having audio performance issues, try following the full guide...

### Pipewire?

**December 2024 update:** In general, I feel more confident about recommending pipewire using backports (up to v.1.2.5) or building from git ([official build instructions](https://gitlab.freedesktop.org/pipewire/pipewire/-/blob/master/INSTALL.md?ref_type=heads)). I personally avoid backports as I've found them to be a bit of a pain to use. You have to select each package, manually force the backport version and deal with various potential "broken" packages. For ease of setting up, I still think Debian 13 will be a better time to jump on the pipewire train but if you can stomach backports or have the confidence to build from git, go ahead!

**Advice for less experienced Linux users of Debian 12 and earlier**: In short, no, don't do it if you are a pro audio user. I don't believe those regular repository package versions are ready for primetime.  The following commands should be safe to run on Debian 12 with any desktop environment (or any debian-based distro that ships with pipewire audio as the default) to revert to ALSA + Pulseaudio + JACK:

```shell
sudo apt remove pipewire-alsa pipewire-pulse pipewire-jack
sudo apt install pulseaudio pulseaudio-module-jack jackd2 pavucontrol
sudo apt install pipewire-media-session wireplumber-
sudo reboot
```
You should end up with something like the following if you run `inxi -Aa`:  
![image](https://github.com/chmaha/DebianProAudio/assets/120390802/bad4ddf4-a5fd-4785-a4c1-2123f0870639)


## Full In-depth Guide

### 1. Install Debian 12 (or other favorite Ubuntu-based or Debian-based distro)

To make your life easier, install either Ubuntu Studio or AVLinux. Almost all of the following tweaks are taken care of. Otherwise, pick a regular distro such as Debian, Ubuntu, MXLinux, etc.

### 2. rtcqs (formerly known as realtimeconfigquickscan)
    
```shell
git clone https://codeberg.org/rtcqs/rtcqs.git
cd rtcqs
./src/rtcqs/rtcqs.py
```

### 3. Add user to audio group and configure realtime privileges

I believe that installing jackd2 takes care of the following these days.

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

Reboot to see the effects.

### 4. Kernel tweaks

First, check via `uname -a` to see if you have PREEMPT_DYNAMIC (this is true for Debian 12). If so, add "preempt=full",  "threadirqs" and "cpufreq.default_governor=performance" as kernel parameters:

```shell
sudo nano /etc/default/grub
```
change 
`GRUB_CMDLINE_LINUX=""` to `GRUB_CMDLINE_LINUX="preempt=full threadirqs cpufreq.default_governor=performance"`
    
```shell
sudo update-grub
```
Remember to reboot to see the effects.

If you don't have PREEMPT_DYNAMIC due to an older kernel version, for better performance consider installing `linux-lowlatency` package if on Ubuntu or Liquorix if on Debian via instructions at https://liquorix.net/ and then just add the "threadirqs" and "cpufreq.default_governor=performance" kernel parameters as shown above.
    
### 5. Swappiness

```shell
sudo nano /etc/sysctl.d/99-sysctl.conf
```
add "vm.swappiness=10"
    
### 6. Spectre/Meltdown Mitigations

If you run `rtcqs.py` and it gives you a warning about Spectre/Meltdown Mitigations, you could add `mitigations=off` to GRUB_CMDLINE_LINUX. Warning: disabling these mitigations will make your machine less secure! https://wiki.linuxaudio.org/wiki/system_configuration#disabling_spectre_and_meltdown_mitigations

### 7. Install udev-rtirq (ignore if using pipewire and "pro audio" profile?)

```shell
git clone https://github.com/jhernberg/udev-rtirq.git
cd udev-rtirq
sudo make install
reboot
```

### 8. Jack2 + Jack D-Bus (ignore if using pipewire)

```shell
sudo apt install qjackctl jackd2 pulseaudio-module-jack
```
Enable Jack D-Bus interface:  
![image](https://github.com/chmaha/DebianProAudio/assets/120390802/ba263c8f-9d4c-4cd6-9e3a-38939d2ed0b5)

Select your audio interface:  
![image](https://github.com/chmaha/DebianProAudio/assets/120390802/ac98834b-c369-4e82-b372-0fab6abdbabc)


To record system audio (say from a browser), 1) make sure JACK is started, 2) start the browser playback, 3) open pavucontrol and select "JACK Sink" as the output under the "playback" tab 4) Connect the relevant cables in qjackctl's graph window being careful to ensure that you are not hearing output twice i.e. delete the cables from the sink direct to the playback and only route to your DAW inputs:

![image](https://github.com/chmaha/DebianProAudio/assets/120390802/dc5b7d0c-153e-4466-8152-4752e2e214fc)


### 9. DAW & Plugins

REAPER: 
http://reaper.fm/download.php 

change RT priority to 40 on audio device page?  


Also be sure to check out Bitwig Studio, Tracktion Waveform, Qtractor, LMMS, Rosegarden, Zrythm etc...
https://en.wikipedia.org/wiki/List_of_Linux_audio_software#Digital_audio_workstations_(DAWs)

#### Native plugins
- My JSFX plugin collection (https://forum.cockos.com/showthread.php?t=275301)
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

### 10. Wine-staging or Wine-tkg

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
       
### 11. Install yabridge

i. Please follow the instructions at https://github.com/robbert-vdh/yabridge#usage

To begin, download the latest release from https://github.com/robbert-vdh/yabridge/releases and run:

```shell
tar -C ~/.local/share -xavf yabridge-x.y.z.tar.gz
```

where x.y.z is the version number such as 4.0.1. Don't forget to add yabridgectl to your shell's search path by adding `export PATH="$PATH:$HOME/.local/share/yabridge` to the end of ~/.bashrc. Close then re-open the terminal.

ii. Configure yabridge according to https://github.com/robbert-vdh/yabridge#readme  

iii. Install Windows VST2, VST3 and CLAP plugins  
    
### 12. Check volume levels!

Once everything is set up, don't forget to check that volume levels are set correctly. Run
```
alsamixer
```
to check that output is set to 100 (vertical bars) or gain of 0dB (top left of alsamixer). Use F6 to select the correct soundcard. You can also use your desktop environment's volume controls if you have your interface enabled there but note that numbers don't seem to match alsamixer.

![alsamixer](https://user-images.githubusercontent.com/120390802/209148828-f5654838-eb25-4dd2-9955-4e0e8db99be2.png)

### 13. Other useful tools (all available via the package manager)

**Music Player**: strawberry (can produce bit-perfect playback) <br>
![image](https://user-images.githubusercontent.com/120390802/209885065-970b9ee3-f4ae-48c2-a0d5-a6f52204e682.png) <br>
**Tagger**: kid3, picard <br>
**DDP creation/verification/etc**: ddptools <br>
**Audio Converter**: fre:ac or soundconverter <br>
**CD Ripper**: asunder or cdrdao <br>
**CD Burner**: cdrdao, k3b or nerolinux4

