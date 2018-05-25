# Minimal i3 Ubuntu 18.04
#### TL,DR
> A fully fonctional and good-looking linux for less than 256mb of ram 

![mod+return to open a terminal ;)](https://transfer.sh/iqdPs/sc.png)

## Setting Up
####  Downloading and installing 
Get the "mini.iso" image from [archive.ubuntu.com](http://archive.ubuntu.com/ubuntu/dists/bionic/main/installer-amd64/current/images/netboot/)
#### Legacy boot
Just flash an usb drive with it as follow.
```bash
sudo dd bs=4M if=mini.iso of=/dev/sd<?> conv=fdatasync && sync
```
#### UEFI boot
You have to download the server image from [cdimage.ubuntu.com](http://cdimage.ubuntu.com/ubuntu-server/daily/current/bionic-server-arm64.iso)
Use a clean Fat32 usbdrive where you copy the content of the ```mini.iso``` image from the legacy boot section and add the ```EFI``` folder from the server image you just downloaded.
You should be all set to boot from the usbdrive.
If you're using only Ubuntu on this computer, clear all keys from secure boot before installation and add the  "shim" pe executable to the trusted boot chain after installation.
#### Installation
Do it as usual and be careful to not select any additional packet to be installed on the system.
*'member that the swap partition is depreciated, Ubuntu will now use a file if it's not present.*
## First Boot
#### Prompting tty1
Press ```shift``` on boot to access grub and ```e``` to edit options.
Remove ```$vt_handoff``` from the ```linux``` line and press ```f10``` to boot.
Once logged, edit ```/etc/default/grub``` and remove ```quiet splash``` from ```GRUB_COMMAND_LINE_DEFAULT```.
Update grub
```bash
sudo update-grub
```
#### Installing packages
Install required packages ( edit accordingly )
```bash
sudo apt install i3 ubuntu-drivers-common mesa-utils mesa-utils-extra gnupg numlockx xautolock scrot xorg xserver-xorg wget unzip wpasupplicant
```
I like to install Google's Chrome so I can open PDFs , MKVs, MP3s etc... without the need to install lots of packages. ( plus it's a really secure one ;) ) 
```bash
wget "https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb"
sudo dpkg -i google-chrome-stable_current_amd64.deb
sudo apt install -f
```
If you don't want to use cool-retro-term skip this part and edit the next part accordingly
```bash
echo "deb http://ppa.launchpad.net/vantuz/cool-retro-term/ubuntu bionic main" | sudo tee -a /etc/apt/source.list
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys BDB05D75
sudo apt update
sudo apt install cool-retro-term
```
To use certain glyphs on my status bar i like to use fontawesome, feel free to use your own:
```bash
wget "https://use.fontawesome.com/releases/v5.0.13/fontawesome-free-5.0.13.zip"
unzip fontawesome-free-5.0.13.zip
sudo cp fontawesome-free-5.0.13/use-on-desktop/* /usr/local/share/fonts/
fc-cache -f -v
#the next command help see the name to use
#here Font Awesome 5 Free
fc-list | grep -i "awe" 
```
'member to comment out the ```/boot``` and ```/EFI``` mountpoints from ```/etc/fstab```

#### Setting up the network
Find your wireless card and copy wifi example:
```bash
ls /sys/class/net/
sudo cp /usr/share/doc/netplan.io/examples/wireless.yaml /etc/netplan/01-netcfg.yaml
```
Edit ```/etc/netplan/01-netcfg.yaml``` and tune it for your network. here an example configuration:
```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp3s0:
      dhcp4: yes
      dhcp6: no
      gateway4: 192.168.0.1
      nameservers:
          addresses: [ 1.1.1.1, 8.8.8.8]
  wifis:
    wlp3s0:
     dhcp4: yes
     dhcp6: no
     gateway4: 192.168.0.1
     nameservers:
          addresses: [ 1.1.1.1, 8.8.8.8]
     access-points:
        "MY_HOME_ROOTER":
          password: "azerty1234"

```
Put your iptable scripts ```/usr/lib/networkd-dispatcher/routable.d/00-iptable-flush```:
```bash
#!/bin/sh
/sbin/iptables -F
/sbin/ip6tables -F
```
Block anything goining on ith chrome mdns server for example ```/usr/lib/networkd-dispatcher/routable.d/50-iptable-mdns``` :
```bash
#!/bin/sh
/sbin/iptables -A INPUT -p udp --dport 5353 -j DROP
/sbin/iptables -A OUTPUT -p udp --dport 5353 -j DROP

/sbin/iptables -A INPUT -p udp --sport 5353 -j DROP
/sbin/iptables -A OUTPUT -p udp --sport 5353 -j DROP

/sbin/ip6tables -A INPUT -p udp --dport 5353 -j DROP
/sbin/ip6tables -A OUTPUT -p udp --dport 5353 -j DROP

/sbin/ip6tables -A INPUT -p udp --sport 5353 -j DROP
/sbin/ip6tables -A OUTPUT -p udp --sport 5353 -j DROP
``` 
Make them executable:
```bash
sudo chmod +x /usr/lib/networkd-dispatcher/routable.d/00-iptable-flush
sudo chmod +x /usr/lib/networkd-dispatcher/routable.d/50-iptable-mdns
```
Then execute ```sudo netplan apply``` to connect 
#### Tweeking the i3 desktop
First of all we need to run it automatically each time we login.
Let's edit ```~/.bash_profile``` and add the following to it:
```bash
if [[ -z $DISPLAY ]] && [[ $(tty) = /dev/tty1 ]]; then
  startx
fi
```
Let's edit ```~/.xinitrc``` to:
```bash
export TERMINAL = "cool-retro-term"
alias open="xdg-open"
i3
```
Now, start i3 by running ```startx``` command.
Use ```modkey+d``` and type the name of a program you wich to start
Open a terminal and edit "~/.config/i3/config" file
I like to do the following:
+ comment the line ```font pango:monospace 8``` 
+ add  ```font pango:DejaVu Sans Mono, Font Awesome 5 Free 8```
+ remove the windows border:
```bash
#new_window 1pixel
for_window [class="^.*"] border pixel 1
# class                 border  backgr. text    indicator
client.focused          #000000 #555555 #ffffff #000000
client.focused_inactive #000000 #000000 #ffffff #000000
client.unfocused        #000000 #000000 #aaaaaa #000000
client.urgent           #000000 #000000 #ffffff #000000
```
+ add support for audio key and basic setup
```bash
#audio control
bindsym XF86AudioRaiseVolume exec "amixer -q sset Master,0 1+ unmute"
bindsym XF86AudioLowerVolume exec "amixer -q sset Master,0 1- unmute"
bindsym XF86AudioMute exec "amixer -q sset Master,0 toggle"
#set touch enable ( change according to your setup )
exec xinput set-prop 13 282 1
# numlock on boot
exec_always --no-startup-id numlockx on
#locking the screen ( should be the exact size of the screen )
bindsym $mod+p exec i3lock -i "PATH_TO_IMAGE.png"
exec xautolock -time 15 -locker i3lock -i "PATH_TO_IMAGE.png"
#screenshot
bindsym --release Print exec scrot -m "SOME_PATH/%s_%H%M_%d.%m.%Y_$wx$h.png"

```
+ I like my bar on top with a specific configuration:
```bash
bar {
	position top
	status_command i3status --config ~/.config/i3/i3status
}
```
Create ```.config/i3/i3status``` and add the following ( tweek it according  to your needs )
```bash
# i3status configuration file.
# see "man i3status" for documentation.

# It is important that this file is edited as UTF-8.
# The following line should contain a sharp s:
# ß
# If the above line is not correctly displayed, fix your editor first!

general {
        colors = true
        interval = 5
}


order += "disk /"
order += "wireless _first_"
order += "ethernet _first_"
order += "run_watch DHCP"
order += "run_watch VPN"
order += "ipv6"
order += "volume master"
order += "load"
order += "battery 1"
order += "tztime local"

wireless _first_ {
        format_up = " (%essid - %quality) %ip"
        format_down = ""
        color_bad = "#777777"
}

ipv6 {
	format_down = "IPv6"
	color_bad = "#777777"
}

ethernet _first_ {
	format_up = " %ip"
	format_down = ""
	color_bad = "#777777"
}

volume master {
        format = " %volume"
        format_muted = " (%volume)"
        device = "default"
        mixer = "Master"
        mixer_idx = 0
}     

battery 1 {
        format = "%status %percentage %remaining"
        status_bat = ""
        status_chr = ""
        status_full = ""
        low_threshold = 30
        threshold_type = percentage
        integer_battery_capacity = true
        color_good = "#0000FF"
}

run_watch DHCP {
	pidfile = "/var/run/dhclient*.pid"
	format = "DHCP"
	format_down = "DHCP"
	color_bad = "#777777"
}

run_watch VPN {
	pidfile = "/var/run/vpnc/pid"
	format = "VPN"
	format_down = "VPN"
	color_bad = "#777777"
}

tztime local {
        format = " %h %d |  %I:%M"
}

load {
        format = " %1min"
}

disk "/" {
        format = " %avail"
}

cpu_usage {
	format = " %usage"
}
```
#### Aditional stuff
You can install the intel microcode updates to avoid spectre and meltdown for intel processors.
```bash
sudo apt-get install microcode.ctl intel-microcode
sudo poweroff --reboot
dmesg | grep "updated"
```

For Nvidia users you can install there drivers.
```bash
echo "deb http://ppa.launchpad.net/graphics-drivers/ppa/ubuntu bionic main" | sudo tee -a /etc/apt/sources.list
sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 1118213C
sudo apt update
sudo ubuntu-drivers autoinstall
```

You can add ```@daily apt update && apt upgrade -y``` to crontab using ```sudo crontab -e``` to make you computer look and install update daily.

### What's next ?
Well you have now a cool-looking and light Linux running on your computer.
You can install steam and play videogames while saving ram :)
You can make this your own distribution to share with others using [respin](http://www.linuxrespin.org) 
And basically what ever you feel like doing with a Linux  ;)
