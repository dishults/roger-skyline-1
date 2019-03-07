# roger-skyline-1.5
### Initiation to system and network administration.
Please note:
- project is done in **Ubuntu Server 18.04**, so some stuff might be different from Debian
- done only the mandatory part: **VM + Network and Security**
## Prerequisites
* VirtualBox
* Ubuntu Server 18.04
* A disk size of 8 GB.
* Have at least one 4.2 GB partition.
* Up to date with all the packages installed to meet the demands of this project.
## Install Ubuntu updates
```bash
sudo apt-get update           # Fetches the list of available updates
sudo apt-get upgrade -y       # Strictly upgrades the current packages
sudo apt-get dist-upgrade -y  # Installs updates (new ones)
sudo apt-get autoremove -y    # removes packages that were automatically installed to satisfy dependencies for some package and that are nomore needed.
sudo apt-get autoclean -y     # clears out the local repository of retrieved package files. The difference is that it only removes package files that can no longer be downloaded, and are largely useless. This allows a cache to be maintained over a long period without it growing out of control.
sudo reboot                   # if needed
    -y, --yes, --assume-yes
               Automatic yes to prompts;
```
### To just list the updates without installing them
    apt list --upgradable
## [Configuring Static IP Addresses On Ubuntu Servers](https://websiteforstudents.com/configure-static-ip-addresses-on-ubuntu-18-04-beta/)
To be able to connect to your machine from mac’s terminal first **in your VirtualBox** - Ubuntu - Settings - Network - Bridged Adapter - Attached to - set the interface through which you’re connected to the Internet at the moment, mine is Wi-FI:

[image_is_coming]

> PS. If you leave the **default Attached to NAT** you will be able to connect to your machine from mac’s terminal but only through loopback interface (127.0.0.1) not your static ip! And also for that to work you’d need to click **Advanced - Port Forwarding** and add in **Host Port** and in **Guest Port** your ```ssh port```.
---
### Continue **on your machine**.
First, do:
```
netplan ip leases enp0s3
```
Which gave me:

	ADDRESS=192.168.86.34
	ROUTER=192.168.86.1
	DNS=192.168.86.1

My static IP should be out of range of IPs that my router is reserved for assigning. To check that, just enter in your web browser your router’s (gateway) address, in my case it’s 192.168.86.1. Then go to LAN settings - DHCP Address Pool and there for me it states Starting IP 192.168.86.20 - Ending IP 192.168.86.250.

Then, according to [this guide](http://droptips.com/cidr-subnet-masks-and-usable-ip-addresses-quick-reference-guide-cheat-sheet) section **/30 - Usable IPs - 2**. Meaning that in netmask /30 I can assign my static IP not greater than number 2. As 1 already reserved (192.168.86.**1**) by the router for gateway then my static IP in netmask /30 could only be 192.168.86.**2** (which is also out of range of my router’s ips). Also, 3 will be my broadcast address.

So I should change this file, which is in [YAML format](https://yaml.org/start.html): Use spaces, no tabs !
```
sudo nano /etc/netplan/50-cloud-init.yaml
```

Into following:
```bash
network:
   ethernets:
      enp0s3:
         addresses: [192.168.86.2/30] #static IP and netmask in /30
         gateway4: 192.168.86.1
         nameservers:
           addresses: [192.168.86.1]
         dhcp4: no
   version: 2
```
then:
```bash
sudo netplan apply
#or
sudo netplan --debug apply
```
To check do:

    sudo apt-get update   
> if it works than my static ip is assigned correctly

To make it permanent, follow the steps described in 50-cloud-init. Write a file:

    /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg

With the following:

    network: {config: disabled}
