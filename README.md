# drakvuf_Installation

# About
DRAKVUF® is a virtualization based agentless black-box binary analysis system. DRAKVUF® allows for in-depth execution tracing of arbitrary binaries (including operating systems), all without having to install any special software within the virtual machine used for analysis.
This repository contains the installation procedure of the Drakvuf tool.

For more details about it, visit [here](https://drakvuf.com/).

# Installation

# Operating System Setup

- The setup of XEN has been configured with Ubuntu as Domain0.
- It is always preferred to install latest stable version of UBUNTU OS. 
- Insert the boot drive and proceed with installation.
- Select the preferred language.
- In Installation type menu, select the Something else option.
- Proceed to Space allocation menu, If you already have some pre-installed lvm partition you can run the following command to delete it. " sudo lvremove <partitionName> "
- Now create the "swap" space, efi space and the main system space for DOM0 XEN installation. 
![Operating System setup](/images/1.png)
![Operating System setup](/images/2.png)
![Operating System setup](/images/3.png)
![Operating System setup](/images/4.png)
![Operating System setup](/images/5.png)
![Operating System setup](/images/6.png)
![Operating System setup](/images/7.png)
![Operating System setup](/images/8.png)

# Dependencies and Packages Installation
Run the following commands in your system shell.
These commands works fine with Debian based linux distro. We have used the Ubuntu 20.04 Focal Fossa operating system. First install the required dependencies.
 
- Make sure to Update your linux system before proceeding installation of XEN environment:
```
sudo apt update
sudo apt upgrade -y
```
- Now install the required Dependencies.
```
sudo apt-get install wget git bcc bin86 gawk bridge-utils iproute2 libcurl4-openssl-dev bzip2 libpci-dev build-essential make gcc clang libc6-dev linux-libc-dev zlib1g-dev libncurses5-dev patch libvncserver-dev libssl-dev libsdl-dev iasl libbz2-dev e2fslibs-dev git-core uuid-dev ocaml libx11-dev bison flex ocaml-findlib xz-utils gettext libyajl-dev libpixman-1-dev libaio-dev libfdt-dev cabextract libglib2.0-dev autoconf automake libtool libjson-c-dev libfuse-dev liblzma-dev autoconf-archive kpartx python3-dev python3-pip golang python-dev libsystemd-dev nasm -y
```
- pip3 command is used to install those dependency packages which old and cannot be installed from apt command.
```
sudo pip3 install pefile construct
```
### Cloning Drakvuf from official repository
Cloning the Drakvuf directory from the official Github Repository.
```
cd ~
git clone https://github.com/tklengyel/drakvuf
cd drakvuf
git submodule update --init
cd xen
./configure --enable-githttp --enable-systemd --enable-ovmf --disable-pvshim
make -j4 dist-xen
sudo apt-get install -y ninja-build
make -j4 dist-tools
make -j4 debball
```
# Xen Installation
- Now we have to install "Xen" with dom0 getting 4GB RAM assigned and two dedicated CPU cores. You can modify these configuration according to your need. At last update the Grub and reboot the system.
```
sudo su
apt-get remove xen* libxen*
dpkg -i dist/xen*.deb
echo "GRUB_CMDLINE_XEN_DEFAULT=\"dom0_mem=4096M,max:4096M dom0_max_vcpus=4 dom0_vcpus_pin=1 force-ept=1 ept=ad=0 hap_1gb=0 hap_2mb=0 altp2m=1 hpet=legacy-replacement smt=0\"" >> /etc/default/grub
echo "/usr/local/lib" > /etc/ld.so.conf.d/xen.conf
ldconfig
echo "none /proc/xen xenfs defaults,nofail 0 0" >> /etc/fstab
echo "xen-evtchn" >> /etc/modules
echo "xen-privcmd" >> /etc/modules
systemctl enable xen-qemu-dom0-disk-backend.service
systemctl enable xen-init-dom0.service
systemctl enable xenconsoled.service
update-grub
```
Note: Make sure that you are running a relatively recent kernel. In Ubuntu 20.04 the 5.10.0-1019-oem kernel has been verified to work, anything newer would also work. Older kernels in your dom0 will not work properly.

```
uname -r
```
- After above step make the XEN to boot before the linux kernal.
```
cd /etc/grub.d/;
mv 20_linux_xen 09_linux_xen
```
- Update the changes in the Grub and then reboot the system.
```
update-grub
reboot
```
- Verify the XEN installation. The output will show the "Running in PV context on Xen v4.7" message on the screen
```
sudo xen-detect
```
- This command will list the running VM.
```
sudo xl list
```
The output should have to be similar to this.
```
  Name                                        ID   Mem    VCPUs 	 State	 Time(s)
Domain-0                                       0  4096     2       r-----    614.0
```
# Logical Volume Manager(LVM Setup and Installation)
- First install the lvm2
```
sudo apt-get install lvm2 -y
```
Note: Before creating physical volume (PV), Go inside Disk partition and create a volume. Never give the whole path of disk like /dev/sda otherwise your os will be crashed down. So when you will create a volume then it has named like /dev/sd2 or /dev/sd3 and so on. So pick only a free volume then move ahead.

- List the empty disk using the following command:
```
lsblk
```
- Create physical volume. Hera "sda" is disk volume, it can vary accordingly.
```
pvcreate /dev/sda2
```
- Create one volume Group.
```
vgcreate vg /dev/sda2
```
- Create logical volume group. You can specify the size, here we allocating 110 GB space, you can change it by altering the the size of '110'.
```
lvcreate -L110G -n windows7-sp1 vg
```
# Install the VMM Utility tool And Networking tool.
- Now Install the VMM utility from the ubuntu software software. From ubuntu softwares download : " Virtual Machine Manager "
### Newtworking Configuration
Now install the networking tool.

Next we need to set up our system so that we can attach virtual machines to the external network. This is done by creating a virtual switch within dom0. The switch will take packets from the virtual machines and forward them on to the physical network so they can see the internet and other machines on your network.

The piece of software we use to do this is called the Linux bridge and its core components reside inside the Linux kernel. In this case, the bridge acts as our virtual switch. The Debian kernel is compiled with the Linux bridging module so all we need to do is install the control utilities:
```
sudo apt-get install bridge-utils
```
Management of the bridge is usually done using the brctl command. The initial setup for our Xen bridge, though, is a "set it once and forget it" kind of thing, so we are instead going to configure our bridge through Debian’s networking infrastructure. It can be configured via " /etc/network/interfaces ".

Open this file with the editor of your choice. If you selected a minimal installation, the nano text editor should already be installed. Open the file:

Note: nano is a text file editor for linux. You can also use the vi, vim editor also. If command show error then run " sudo apt install nano -y "

```
sudo nano /etc/network/interfaces      //open the interface file
```
This file is very simple. Each stanza represents a single interface.

Breaking it down,

1. “auto eth0” meeans that eth0 will be configured when ifup -a is run (which happens at boot time). This means that the interface will automatically be started/stopped for you. ("eth0 is its traditional name - you'll probably see something more current like "ens1", "en0sp2" or even "enx78e7d1ea46da")

2. “iface eth0” then describes the interface itself. In this case, it specifies that it should be configured by DHCP - we are going to assume that you have DHCP running on your network for this guide. If you are using static addressing you probably know how to set that up.

We are going to edit this file so it resembles such:

Note: Change according to your network interface (run “ifconfig”).

Copy the following text and paste it in the interfaces files.
```
auto lo
  iface lo inet loopback

  auto eth0
  iface eth0 inet manual
  
  auto xenbr0
  iface xenbr0 inet dhcp
       bridge_ports eth0
```








  







