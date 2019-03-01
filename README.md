# roger-skyline-1.5
Initiation to system and network administration
## Prerequisites
* VirtualBox
* Ubuntu Server 18.04
* A disk size of 8 GB.
* Have at least one 4.2 GB partition.
* Up to date with all the packages installed to meet the demands of this project.
## Install Ubuntu updates
```
sudo apt-get update           # Fetches the list of available updates
sudo apt-get upgrade -y       # Strictly upgrades the current packages
sudo apt-get dist-upgrade -y  # Installs updates (new ones)
sudo apt-get autoremove -y    # removes packages that were automatically installed to satisfy dependencies for some package and that are nomore needed.
sudo apt-get autoclean -y     # clears out the local repository of retrieved package files. The difference is that it only removes package files that can no longer be downloaded, and are largely useless. This allows a cache to be maintained over a long period without it growing out of control.
sudo reboot                   # if needed
    -y, --yes, --assume-yes
               Automatic yes to prompts;
```

## [Configuring Static IP Addresses On Ubuntu Servers](https://websiteforstudents.com/configure-static-ip-addresses-on-ubuntu-18-04-beta/)
* The file is in [YAML format](https://yaml.org/start.html): Use spaces, no tabs !
```
netplan ip leases enp0s3
```
Gave me:

- ADDRESS=10.0.2.15
- NETMASK=255.255.255.0
- ROUTER=10.0.**2**.2
- SERVER_ADDRESS=10.0.2.2
- NEXT_SERVER=10.0.2.4
- T1=43200
- T2=75600
- LIFETIME=86400
- DNS=8.8.8.8 8.8.4.4

According to [this guide](https://dnsmadeeasy.com/support/subnet/) section **/30 -- 64 Subnets -- 2 Hosts/Subnet**

In my router the gateway address is 10.0.**2**.2 so **2** is in 0 in the guide, so my ip in netmask /30 can only be 1 or 2. As 2 already taken for gateway then I can assign only 1 - 10.0.2.**1**/30.
Also 3 will be my broadcast address.
So I should change this file

    sudo nano /etc/netplan/50-cloud-init.yaml

Into following:

    network:
        ethernets:
            enp0s3:
                addresses: [10.0.2.1/30] #static IP and netmask in /30
                gateway4: 10.0.2.2
                nameservers:
                  addresses: [8.8.8.8,8.8.4.4]
                dhcp4: no
        version: 2

then:

    sudo netplan apply
    #or
    sudo netplan --debug apply

To check do:

    sudo apt-get update   
  if it works than my static ip is assigned correctly

To make it permanent, follow the steps described in 50-cloud-init. Write a file:

    /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg

With the following:

    network: {config: disabled}
