# Common-WSL-Fixes

# Enable WSL2

+ dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
+ wsl --set-default-version 2
+ wsl --set-version <distribution name> <versionNumber> # Preferably Ubuntu. Debian is less feature-rich

# Quickly Trigger Hyper-V Off

This makes a secondary boot-entry so you can switch in like 2 seconds, vs going into the Windows Features and remembering which ones to trigger.

It's worth doing this, promise. 

```
// Run powershell as Admin. (different commands for cmd)
bcdedit --% /copy {current} /d "No Hyper V"
bcdedit --% /set  {OUTPUT ID FROM TERMINAL} hypervisorlaunchtype off
// Reboot
```

# gnome-boxes Display Fix in wslg

```
gnome-boxes --display=
// No idea why this works, but I think it just hates Wayland(as do I)
```

# Expose virtualization extensions in virt-manager/gnome-boxes

This also applies to Hyper-V in some cases, but I'm not sure how. There is a psh line you can google to trigger it for HV.

## Create C:\Users\<YourName>\.wslconfig

I would stick to doing this per-vm, because it will break. Remember to wsl.exe --shutdown for it to take effect.
If process hangs - Open Task Manager > Services > Restart lxssmanager
Any problems just delete the .wslconfig and reboot. 

```
# Exposing nested-virtualization extensions

[wsl2]
nestedVirtualization=true
kernel=C:\\Users\\<YOU>\\bzImage # You may need to compile a kernel for this.
  
# Limiting the vm's resources

[wsl2]
memory=7GB # Limits VM memory in WSL 2 to 4 GB
processors=2 # Makes the WSL 2 VM use two virtual processors /etc/wsl.conf
```

# VMWare Player Being Annoying

VMware Player is actually configurable in a lot of ways, but they go out of their way to act like it isn't.

```
Open an Elevated Text Editor and cd to --
"C:\ProgramData\VMware\VMware Workstation\settings.ini"

# Copy and paste these

ulm.disableMitigations = "TRUE" # Sidechannel Mitigations
printers.enabled = "FALSE"
vmplayer.showChrome = "FALSE" 
# I believe this makes that annoying full-screen bar die --and they act like it's a 'premium' feature lol.
pref.vmplayer.fullscreen.nobar = "TRUE" # But there is something weird about it that I can't recall so this option may be on a 'per' vm basis
```

# Cpu Tests

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
  
 # Docker Vendor Completions
  If your terminal is outputting '/usr/share/zsh/vendor-completions/_docker not found' or similar each time you launch a terminal
  + Open Docker Desktop
  + Preferences > Resources > WSL Integration
  + Untick 'Enable integration with my default WSL distro'
  + Restart WSL and it should be fixed 'wsl.exe --shutdown'
  + Question the ambiguity of that setting, and why it hasn't been rephrased or annotated.
