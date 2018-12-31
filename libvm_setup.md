# Lets reverse malwares!
## tl;dr
> Useing Xen, LibVMI and Drakvuf to carefully trace the execution of Windows and Linux  guests  without impacting them

## Prerequest
- you should have an ubuntu 16.04 *( but it will probably work on any debian based os.)*
-  legacy bios boot ( UEFI version will not be covered here )
-  An uncrypted partition
- make sure you have an empty partition of at least 90Gb
- a basic understanding of how computer and operating system work internaly
- some spare time ;)

## Installation
>Most of this part was took from [Drakvuf](https://drakvuf.com/) installation tutorial.
#### Installing the required pakages:
```sh
sudo apt-get install wget git bcc bin86 gawk bridge-utils iproute libcurl3 libcurl4-openssl-dev bzip2 pciutils-dev build-essential make gcc clang libc6-dev libc6-dev-i386 linux-libc-dev zlib1g-dev python-dev python-pip libncurses5-dev patch libvncserver-dev libssl-dev libsdl-dev iasl libbz2-dev e2fslibs-dev git-core uuid-dev ocaml libx11-dev bison flex ocaml-findlib xz-utils gettext libyajl-dev libpixman-1-dev libaio-dev libfdt-dev cabextract libglib2.0-dev autoconf automake libtool check libjson-c-dev libfuse-dev checkpolicy liblzma-dev autoconf-archive kpartx python-capstone lvm2
```
#### Building:
```sh
cd ~
git clone https://github.com/tklengyel/drakvuf
cd drakvuf
git submodule init
git submodule update
cd xen
./configure --enable-githttp
make -j4 dist-xen
make -j4 dist-tools
```
#### Seting it up:
Edit the grub command line according your ram and cpu
```sh
sudo su
make -j4 install-xen
make -j4 install-tools
echo "GRUB_CMDLINE_XEN_DEFAULT=\"dom0_mem=1664M,max:4096M dom0_max_vcpus=4 dom0_vcpus_pin=true hap_1gb=false hap_2mb=false altp2m=1 flask_enforcing=1\"" >> /etc/default/grub
echo "/usr/local/lib" > /etc/ld.so.conf.d/xen.conf
ldconfig
echo "none /proc/xen xenfs defaults,nofail 0 0" >> /etc/fstab
echo "xen-evtchn" >> /etc/modules
echo "xen-privcmd" >> /etc/modules
update-rc.d xencommons defaults 19 18
update-rc.d xendomains defaults 21 20
update-rc.d xen-watchdog defaults 22 23

```
Remove the simlinks in */boot/* 
Edit */etc/default/grub* by removing hidden timeout and add a non 0 value to grub timeout.
Make sure you are not using Nvidia proprietary drivers.
```
update-grub
reboot
```
Once you booted into xen you can try a simple test
```
sudo xen-detect
```
if it's good, let's continue with libvmi :)
```sh
cd ~/drakvuf/libvmi
./autogen.sh
./configure --disable-kvm --enable-shm-snapshot
make
sudo make install
sudo echo "export LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:/usr/local/lib" >> ~/.bashrc
```
now rekall *( it's a fork of volatility by google  we need it to get the offets of  "windows sytemcalls")*
```sh
cd ~/drakvuf/rekall/rekall-core
sudo pip install setuptools
python setup.py build
sudo python setup.py install
```
#### Create a bridge 

click on network > edit connextion > add a new connexion > briged
add slave connexion > ethernet > choose interface and save
save
remove wired connection

#### installing windows
'member the empty partition?
```sh
sudo pvcreate /dev/sda3
sudo vgcreate vg /dev/sda3
sudo lvcreate -L90G -n windows7-sp1 vg
```
make sure you have a windows 7 iso in your folder.
if you don't 'member where you puted it, you can use one of my old friend trick to find it back by googeling this:
```sh
"indef of /" "X17-59183.iso"
```
create a template file windows7-sp1.conf with the folloing parameters *(edit accordingly to your configuration)*
i use SDL and an usb wifi device but you can use a server with a bridge and vnc
```
arch = 'x86_64'
name = "windows7-sp1"
maxmem = 2048
memory = 2048
vcpus = 2
maxcpus = 2
builder = "hvm"
boot = "cd"
hap = 1
acpi = 1
on_poweroff = "destroy"
on_reboot = "restart"
on_crash = "destroy"
sdl=1
vnc=0
vnclisten="0.0.0.0"
#monitor=1
#usb = 1
#usbdevice = ['host:148f:3070']
altp2m = 2
shadow_memory = 16
audio=1
soundhw='hda'
vif = [ 'type=ioemu,model=e1000,bridge=bridge0,mac=00:06:5B:BA:7C:01' ]
disk = [ 'phy:/dev/vg/windows7-sp1,hda,w', 'file:X17-59183.iso,hdc:cdrom,r' ]
```
intall windows as you usually do..
```sh
sudo xl create windows7-sp1.conf
```
##### Little tip:
> When it's installed, to update properly, download Google Chrome using IE then download IE 11 and install it.
> It will fix most of the issues you can encounter with Windows Update.

### Backup your windows
```sh
sudo dd if=/dev/vg/windows7-sp1 conv=fdatasync | bzip2 -9f > windows7-sp1.img.bz2
