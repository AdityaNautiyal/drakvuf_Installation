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





