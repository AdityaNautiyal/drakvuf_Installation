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
Make sure to add the bridge stanza, be sure to change dhcp to manual in the iface eth0 inet manual line, so that IP (Layer 3) is assigned to the bridge, not the interface. The interface will provide the physical and data-link layers (Layers 1 & 2) only.

-Restart the Networking service.
```
sudo service network-manager restart
```
- Now turn on the network bridge service.
```
sudo gedit /etc/NetworkManager/NetworkManager.conf
// Now a file will open there change
// manages = true , from false to turn on the network bridge
sudo service network-manager restart
```
- To check available network Vm interfaces.
```
brctl show
```
The output will show something like this.
```
bridge name     bridge id               STP enabled     interfaces
virbr0          8000.4ccc6ad1847d       yes              virbr0-nic
```
The networking service can now be set to start automatically whenever the system is rebooted. Please review the installation instructions once again if you are having trouble getting this type of output.

# Download link for windows 7 iso:
Before proceeding furthur we need 64-bit windows 7 iso image. You can download from anywhere or can find the already tested image from [here](https://getintopc.com/softwares/operating-systems/windows-7-ail-in-one-32-64-bit-iso-may-2019-download-9618001/).

# VM Creation and Configuration
Next step is to edit the xen VM's configuration file.
```
sudo gedit /etc/xen/win7.cfg
```
This template is used for creating the Configurtion for Windows 7 VM from the download ISO file. You can modify the number of cpu, max memory, VM behaviour and other system tuning.
```
arch = 'x86_64'
name = "windows7-sp1"
maxmem = 3000
memory = 3000
vcpus = 2
maxvcpus = 2
builder = "hvm"
boot = "cd"
hap = 1
on_poweroff = "destroy"
on_reboot = "destroy"
on_crash = "destroy"
vnc = 1
vnclisten = "0.0.0.0"
vga = "stdvga"
usb = 1
usbdevice = "tablet"
audio = 1
soundhw = "hda"
viridian = 1
altp2m = 2
shadow_memory = 32
vif = [ 'type=ioemu,model=e1000,bridge=virbr0,mac=48:9e:bd:9e:2b:0d']
disk = [ 'phy:/dev/vg/windows7-sp1,hda,w', 'file:/home/pc-1/Downloads/windows7.iso,hdc:cdrom,r' ]
```
Note: Make changes according to your file path of windows iso image and mac address.
# Clone LibVMI and Installation
- Now, Enter into the LibVMI folder and build it.

```
cd ~/drakvuf/libvmi
autoreconf -vif
./configure --disable-kvm --disable-bareflank --disable-file
```
- Output of the above command should look something this:
```
Feature         | Option
----------------|---------------------------
Xen Support     | --enable-xen=yes
KVM Support     | --enable-kvm=no
File Support    | --enable-file=yes
Shm-snapshot    | --enable-shm-snapshot=no
Rekall profiles | --enable-rekall-profiles=yes
----------------|---------------------------

OS              | Option
----------------|---------------------------
Windows         | --enable-windows=yes
Linux           | --enable-linux=yes


Tools           | Option                    | Reason
----------------|---------------------------|----------------------------
Examples        | --enable-examples=yes
VMIFS           | --enable-vmifs=yes        | yes
```
- Build and install LibVMI with the following command
```
make
sudo make install
sudo echo "export LD_LIBRARY_PATH=\$LD_LIBRARY_PATH:/usr/local/lib" >> ~/.bashrc
```
Other commands to run:
```
cd ~/drakvuf/libvmi
./autogen.sh
./configure –disable-kvm
sudo xl list
```
# Clone Volatility and Installation
```
cd ~/drakvuf/volatility3
python3 ./setup.py build
sudo python3 ./setup.py install
```
Rekall installation
```
cd ~/drakvuf/rekall/rekall-core
sudo pip install setuptools
python setup.py build
sudo python setup.py install
```
#Create VM and Configure VM from DOM0
- Last step of this configuration is to create Windows 7 VM using the following command.
```
xl create /etc/xen/win7.cfg
```
In order to login into the virtual machine you have created, you first have to install the "gvncviewer".
gvncviewer provides a graphical interface for the created virtual machine.
```
sudo apt install gvncviewer
```
#JSON File Creation using LibVMI vmi-win-guid tool and Volatility Framework
- Now we will create the JSON configuration file for the Windows domain. First, we need to get the debug information for the Windows kernel via the LibVMI vmi-win-guid tool. For example, in the following my domain is named windows7-sp1.

You can get the list of available virtual machines with their current running status using
```
sudo xl list
```
will output something like
```
Name                                        ID   Mem VCPUs	State	Time(s)
Domain-0                                     0  4024     4     r-----     848.8
windows7-sp1-x86                             7  3000     1     -b----      94.7
```
- Run following to Get the debug information.
```
sudo vmi-win-guid name windows7-sp1-x86
```
if installation is correct you should see output similar to : 
"make sure to note PDB GUID and kernel filename as it would be required further"
```
Windows Kernel found @ 0x2604000
  Version: 32-bit Windows 7
  PE GUID: 4ce78a09412000
  PDB GUID: 684da42a30cc450f81c535b4d18944b12
  Kernel filename: ntkrpamp.pdb
  Multi-processor with PAE (version 5.0 and higher)
  Signature: 17744.
  Machine: 332.
  # of sections: 22.
    # of symbols: 0.
  Timestamp: 1290242569.
  Characteristics: 290.
  Optional header size: 224.
  Optional header type: 0x10b
  Section 1: .text
  Section 2: _PAGELK
  Section 3: POOLMI
  Section 4: POOLCODE
  Section 5: .data
  Section 6: ALMOSTRO
  Section 7: SPINLOCK
  Section 8: PAGE
  Section 9: PAGELK
  Section 10: PAGEKD
  Section 11: PAGEVRFY
  Section 12: PAGEHDLS
  Section 13: PAGEBGFX
  Section 14: PAGEVRFB
  Section 15: .edata
  Section 16: PAGEDATA
  Section 17: PAGEKDD
  Section 18: PAGEVRFC
  Section 19: PAGEVRFD
  Section 20: INIT
  Section 21: .rsrc
  Section 22: .reloc
  ```
Note: If found error in running the above commands, run following command then rerun the above command.
```
sudo /sbin/ldconfig -v
```
- Copy the following string from the terminal output
```
PDB GUID: f794d83b0f3c4b7980797437dc4be9e71
Kernel filename: ntkrnlmp.pdb
```
- Now run the following commands from the by changing the paramater accordingly to create LibVMI config with Rekall profile:
```
cd /tmp
python3 ~/drakvuf/volatility3/volatility/framework/symbols/windows/pdbconv.py --guid f794d83b0f3c4b7980797437dc4be9e71 -p ntkrnlmp.pdb -o windows7-sp1.json
sudo mv windows7-sp1.json /root
```
- Now generate the reakall profile.
```
sudo su
printf "windows7-sp1 {\n\tvolatility_ist = \"/root/windows7-sp1.json\";\n}" >> /etc/libvmi.conf
exit
```
- Now build the drakvuf using the following commands
```
cd ~/drakvuf
autoreconf -vi
./configure
make
```
For first access to virtual Machine you will be required to setup the installed OS, complete it.
Now login to Virtual Machine and install the windows with giving it login password.
```
gvncviewer localhost
```
### Windows recovery image: Restoration Point
In future it is supposed that you will be perofrming many operations on the virtual machine. In case of any defects it is adviced to keep a system a restore point with fresh windows. Follow the steps to do so.
1. Create a partition of 50G. (A seperate Disk drive)
2. Turn all the firewall off.
3. Create a restore point using the newly created partition (new drive) // Search for “create a restore point” in windows start menu.
4. Goto My Computer --> Right click on new volume you have created --> Security --> provide the full control to all the users.

### Some useful XEN commands
- Run the following to get the PID's of the processes running in a particular VM. "sudo vmi-process-list <vmName>"
```
sudo vmi-process-list windows7-sp1
```
- Xen version:
```
sudo xen-detect
```
- List of VMs:
```
sudo xl list
```
- Destroy VM: " sudo xl destroy <id> "
```
sudo xl destroy 1
```
- CREATE VM:
```
sudo xl create /etc/xen/win7.cfg
```
# Program Execution Tracing Log Generation using Drakvuf
- System tracing:
```
sudo ./src/drakvuf -r /root/windows7-sp1.json -d id
```
Here, id is variable. Put id value of virtual machine (use sudo xl list command) inplace of "id"

- Malware Tracing Command
```
sudo ./src/drakvuf -r /root/windows7-sp1.json -d id -x socketmon -t time -i pid -e "Malware location inside Virtual Machine" > output.txt
```
Here,
"pid" is processId, change pid to value of numerical value of pid according to pid of explorer.exe . "id" is of virtual machine (use sudo xl list command). Location of malware ".exe" file in the created windows VM example : "E:\zbot\zbot_1.exe". output.txt is the output file with trace output. By default is drakvuf location.

- Network Tracing
```
ping -n 10000 www.google.com(from cmd of VM)
sudo tcpdump -w "output.pcap" -i vif1.0-emu   (can be obtained from brctl show)
```
### Other Commands
- Windows json file:
```
sudo vmi-win-guid name windows7-sp1
```
- VMI process list:
```
sudo vmi-process-list windows7-sp1
```
- Enabling the debug:
```
make clean
./configure --enable-debug
make
```









  







