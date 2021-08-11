# Common-WSL-Fixes

- [Enable WSL2](#enable-wsl2) 
- [Quickly Trigger Hyper-V Off](#quickly-trigger-hyper-v-off)
- [Disable wslg System-Wide](#disable-wslg-system-wide)
- [Move a Linux Installation](#move-a-linux-installation)
- [Docker Vendor Completions](#docker-vendor-completions)
- [Dealing with Xservers in wslg (failed to launch xsession, etc)](#dealing-with-xservers-in-wslg-failed-to-launch-xsession-etc)
- [Expose virtualization extensions in virt-manager/gnome-boxes](#expose-virtualization-extensions-in-virt-managergnome-boxes)
- [Exposing nested-virtualization extensions](#exposing-nested-virtualization-extensions)
- [Limiting the vm's resources](#limiting-the-vms-resources)
- [VMWare Player Being Annoying](#vmware-player-being-annoying)
- [Cpu Tests](#cpu-tests)
- [Linux Benchmarks](#linux-benchmarks)
- [Export Installed Packages in Windows (winget)](#export-installed-packages-in-windows-winget)
- [Scrolling is Not Smooth](#scrolling-is-not-smooth)

Firstly, many of the common fixes will be found here in this ultra-secret microsoft documentation. 
https://github.com/MicrosoftDocs/wsl/blob/master/WSL/wsl-config.md
Everything in this guide is related to quality-of-life and hyper-v shenanigans breaking virtualbox, vmware, and some common fixes with the wsl cli.

## Enable WSL2

+ dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
+ wsl --set-default-version 2
+ wsl --set-version <distribution name> <versionNumber> # Preferably Ubuntu. Debian is less feature-rich

## Quickly Trigger Hyper-V Off

This makes a secondary boot-entry so you can switch in like 2 seconds, vs going into the Windows Features and remembering which ones to trigger.

It's worth doing this, promise. 

  ```
// Run powershell as Admin. (different commands for cmd)  
bcdedit --% /copy {current} /d "No Hyper V"  
bcdedit --% /set  {OUTPUT ID FROM TERMINAL} hypervisorlaunchtype off  
// Reboot  
  ```  
  
## Disable wslg System-Wide  
Append to C:\Users\<USER>\.wslconfig
  
  ```
[wsl2] 
guiApplications=false
  ```
  wsl.exe --shutdown  
  
I have noticed some serious background cpu-smashing by wslg, so it's best to disable it unless you specifically need something. Remember that this a wayland layer so expect weirdness and breakage if you have it enabled.  

## Move a Linux Installation  
  
- list distro and version  
wsl -l -v  
  
- OOM
  This exports to stdio, and requires a lot of memory to compress to a .tar. In my case, it looped infinitely.
wsl --export Ubuntu-20.04 - | wsl --import Moved-Ubuntu-20.04 M:\wsl -

Finally I ended up just mounting the linux container in a new distro installation because I'm guessing nixpkg/nix broke the exporting in some way. The return was 'undefined error' so that's my best guess. This lead to a lot of lost time, so if you encounter this, just mount ~/ in a new container and migrate. The .tar took roughly 5-10 mins for a 13 gb vhdx, import failed after 5, and I tried importing 4 times.  

- Without stdio  
wsl --export Ubuntu C:\Path\Truebuntu.tar  
wsl --import <Distro new name> <C:\Path\to\export>   

  - Import  
wsl -- import <The undefined name of your new distro> <Installation Path - M:\wsl\Truebuntu> C:\Path\To\Tar  

- MS Docs  
https://docs.microsoft.com/en-us/windows/wsl/use-custom-distro  

-  First solution / OOM  
https://stackoverflow.com/questions/62533122/how-to-move-a-windows-10-wsl-2-linux-distribution-to-another-location  

  - Cleanup  
Not until you are sure, buddy.    
wsl --unregister Ubuntu  

 - If you're logging in as root instead of a user by default.
 Append to /etc/wsl.conf  
 ``` 
 [user]  
 default=YourUserName  
  
  ```

## Docker Vendor Completions
  If your terminal is outputting '/usr/share/zsh/vendor-completions/_docker not found' or similar each time you launch a terminal
  + Open Docker Desktop
  + Preferences > Resources > WSL Integration
  + Untick 'Enable integration with my default WSL distro'
  + Restart WSL and it should be fixed 'wsl.exe --shutdown'
  + Question the ambiguity of that setting, and why it hasn't been rephrased or annotated.  
  
## Dealing with Xservers in wslg (failed to launch xsession, etc)
  + Consider using gwsl if you need 'stability'*
  + If you're unable to launch something, remember that it uses wayland as the display server, so you can alternatively use Xnest or Xephyr to nest an xserver within Wayland.
  + wsl handles certain environment variables in a very unfriendly way. Check the error logs in ~/ and make sure ~/.profile wasn't set to do something strange. Also check your .bashrc or alternative for $DISPLAY being modified.

## Expose virtualization extensions in virt-manager/gnome-boxes

This also applies to Hyper-V in some cases, but I'm not sure how. There is a psh line you can google to trigger it for HV.

## Exposing nested-virtualization extensions  

[wsl2]  
nestedVirtualization=true  
kernel=C:\\Users\\<YOU>\\bzImage # You may need to compile a kernel for this.  
  
## Limiting the vm's resources

```
[wsl2]  
memory=7GB # Limits VM memory in WSL 2 to 4 GB  
processors=2 # Makes the WSL 2 VM use two virtual processors /etc/wsl.conf
```

  
## VMWare Player Being Annoying

VMware Player is actually configurable in a lot of ways, but they go out of their way to act like it isn't.

```
Open an Elevated Text Editor and cd to --
"C:\ProgramData\VMware\VMware Workstation\settings.ini"

 Copy and paste these

ulm.disableMitigations = "TRUE" # Sidechannel Mitigations
printers.enabled = "FALSE"
vmplayer.showChrome = "FALSE" 
# I believe this makes that annoying full-screen bar die --and they act like it's a 'premium' feature lol.
pref.vmplayer.fullscreen.nobar = "TRUE" # But there is something weird about it that I can't recall so this option may be on a 'per' vm basis
```

## Cpu Tests

Check CPU Features/Capabilities

```
egrep '^flags.*(vmx|svm)' /proc/cpuinfo
```

## Linux Benchmarks

Single Threaded

```
sudo apt-get install sysbench
sysbench --test=cpu --cpu-max-prime=20000 run
```

Multi-Threaded

```sudo apt install stress-ng
stress-ng --cpu-method which
stress-ng --cpu 2 --cpu-method matrixprod  --metrics-brief --perf -t 60
```

## Export Installed Packages in Windows (winget)
  If you format consistently, don't like to deal with cumbersome installers, or just need portability - you need to be using winget. It allows you to save a huge amount of time by letting you install and search software from the cli. It is also safer because the repositories are more trustworthy than *googling installers clumsily*. It's not often that I find something that *isn't* available for me as a package. (Even Steam, Discord, Firefox, and Visual Studio are available.) 
  + winget export filename # Export current Packages
  + winget import filename # Restore Previous Packages
  + Keep in mind that this improves securtity, but also limits it to the user's environment to accomplish that. So if you have multiple users on your setup, the other users will also have to import the package list to have the same access rights systemwide.
  + Always install windows terminal
  + For now, grab winget from Microsoft's github, as it is completely broken in the Windows store.
  
## Scrolling is Not Smooth
If you're having issues with scrolling not being smooth in Windows generally, Browser, Electron Applications (vscode, discord, typora, etc.)
  + Ensure 'smooth scroll-list boxes' and 'animate controls and elements inside windows' are ticked in Performance Options.
  + Ensure you don't have wonky 'scroll acceleration' settings if your computer has a configuration for mouse, touchpad, etc.
  + Pray
  + If that didn't work, also check browser options for 'smooth scrolling' 'and hardware accel' Messing with those should fix it.
  ( edge://flags chrome://flags or wherever in firefox.)
  + One other solution seems to be setting 'Lines to Scroll At A Time' to '1' instead of '3' under Bluetooth And Mouse. (Redundant setting in the old mouse options, so check that too.) 
Windows seems to detick this for some people, regardless of hardware. If you've disabled 'animations' inside of Accessibility Settings, that'll break it too. 
