# 42 - Roger Skyline 1.5 - Mandatory Part Tutorial
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

![image](https://github.com/dishults/roger-skyline-1.5/blob/master/image_preview.png?raw=true)

> If you leave the **default Attached to NAT** you will be able to connect to your machine from mac’s terminal but only through loopback interface (127.0.0.1) not your static ip! And also for that to work you’d need to click **Advanced - Port Forwarding** and add in **Host Port** and in **Guest Port** your ```ssh port```.
---
### Continue on your machine.
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

To make it permanent, follow the steps described in *50-cloud-init*. Write a file:

    /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg

With the following:

    network: {config: disabled}


## [Change default SSH port](https://www.tecmint.com/change-ssh-port-in-linux/)
    sudo nano /etc/ssh/sshd_config

Find the line `#Port 22` and add random port, preferably higher than **1024** (the superior limit of standard well-known ports). The maximum port that can be setup for for SSH is **65535**.
Configure your firewall to the port you’ve added :

    sudo ufw allow 42424
    sudo service sshd restart


## [Enable publickeys and disable password and direct root access](https://www.cyberciti.biz/tips/linux-unix-bsd-openssh-server-best-practices.html)
    sudo nano /etc/ssh/sshd_config

Remove `#` and change lines to:
```bash
Port 42424
HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key
HostKey /etc/ssh/ssh_host_ed25519_key
PermitRootLogin no
PubkeyAuthentication yes
PasswordAuthentication no
ChallengeResponseAuthentication no   # to further desable passwords
UsePAM no                            # to complitely desable passwords
AuthenticationMethods publickey      # add this line right after UsePAM
```
Restart

    sudo service sshd restart

### To connect to my server from mac’s terminal
Generate keys for host (mac):

    ssh-keygen

In order to add the key to guest machine (server) you would need to enable password authentication just this once.

    sudo nano /etc/ssh/sshd_config

Then:

    PasswordAuthentication yes

And silence this line:

    #AuthenticationMethods publickey

Restart:

    sudo service sshd restart

Add host key to guest machine (server).
```bash
#ssh-copy-id username@ipaddress -p [ssh port]
ssh-copy-id dshults@192.168.86.2 -p 42424
#if the key is in different location than .ssh/ or has different name than id_rsa.pub then use -i option, ex:
ssh-copy-id -i ~/.ssh/id_rsa_new.pub dshults@192.168.86.2 -p 42424
```
Disable back the password authentication:

    PasswordAuthentication no
    AuthenticationMethods publickey

Now you can connect to your machine from your mac’s terminal:

    ssh -p 42424 dshults@192.168.86.2

---
To check that I can be root (belong to sudo group)

    id dshults

If not - add myself to sudo

    sudo adduser dshults sudo 

---

## [Setting Up a Basic Firewall](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu-18-04)

If haven’t added the ssh port - do it now

    sudo ufw allow 42424

><details><summary>allow</summary>or <code>limit​</code> to set DOS protection</details>

Enable

    sudo ufw enable

Check the status
```bash
sudo ufw status

#or for more info
sudo ufw status verbose
```

## Set DOS protection on open ports

To check what ports are open (listening):
```bash
sudo lsof -i -P -n | grep LISTEN
#or
sudo netstat -tulpn | grep LISTEN
```
> anything with 127.0.0.* is from a loopback interface, meaning there is no outside threat for those ports.
><details><summary>loopback interface</summary>an internal interface that exists so that the machine can communicate with itself</details>

Then:

    sudo ufw limit 42424/tcp
><details><summary>42424</summary>my ssh custom port</details>
---
Delete any duplicates or unnecessary ports using

    sudo ufw status numbered

Then, by selecting the number

    sudo ufw delete 1

---

## Set scan protection on open ports

Enable logging

    sudo ufw logging medium
><details><summary>logging</summary>Will log everything to <code>/var/log​</code> general logs <code>syslog.log​</code> and <code>kern.log​</code> and to ufw specific <code>ufw.log</code></details>
Install Psad:

    sudo apt-get install psad

[Configure it](https://blog.rapid7.com/2017/06/24/how-to-install-and-use-psad-ids-on-ubuntu-linux/):

    nano /etc/psad/psad.conf

Change to:
```bash
EMAIL_ADDRESSES root@localhost;
	
##Your system hostname 
HOSTNAME ubuntu;
	
##Specify the home and external networks. 
HOME_NET any;     #or your ip
EXTERNAL_NET any;

##By default, psad search for logs in /var/log/messages so change it to
IPT_SYSLOG_FILE /var/log/ufw.log;  #or /var/log/syslog for general log

ALERTING_METHODS        noemail;
ENABLE_AUTO_IDS         Y;
AUTO_IDS_DANGER_LEVEL   1;
IGNORE_PORTS            NONE;
```
Add loopback interface to ignore it to /etc/psad/auto_dl.

    127.0.0.0/8      0

Save and close the file when you are finished. Then update the signatures so that it can correctly recognize known attack types.
```bash
psad --sig-update
# or 
psad -R #restart
```
Restart

    sudo service psad restart

Check the current status

    psad -S

---
### Test it
Install nmap (for tests)

    sudo apt-get istall nmap

Then scan your port multiple times until you can't (normally after 4th):

    sudo nmap 192.168.86.2

Then do the check

    psad -S

To flush (reset) blocked ips:

    psad -F

To restart:

    psad -R


## Stop unnecessary services

List all

    sudo service --status-all | grep +

Disable using:

    sudo systemctl stop [service]
    sudo systemctl disable [service]


## Update_script

Configure the timezone:

    sudo dpkg-reconfigure tzdata

Create update script in `/home/username/`
```bash
nano update_script.sh
chmod +x update_script.sh # or chmod 755
```

With the following:
```bash
#!/bin/bash

ulog=/var/log/update_script.log
date > $ulog
echo -e "\n1/5 - Fetching the list of available updates" >> $ulog
apt-get update >> $ulog
echo -e "\n2/5 - Strictly upgrading the current packages" >> $ulog
apt-get upgrade -y >> $ulog
echo -e "\n3/5 - Installing updates (new ones)" >> $ulog
apt-get dist-upgrade -y >> $ulog
echo -e "\n4/5 - Removing packages that were automatically installed to satisfy dependencies for some packages and that are no more needed" >> $ulog
apt-get autoremove -y >> $ulog
echo -e "\n5/5 - Clearing out the local repository of retrieved package files. Cache cleaning" >> $ulog
apt-get autoclean -y >> $ulog
```
### Set up root cron jobs ([CronHowTo](https://help.ubuntu.com/community/CronHowto))

    sudo crontab -e
>* sudo crontab -l to list
>* sudo crontab -r to remove

Very important ! In the top of the cron file add:

    PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
><details><summary>PATH</summary>Copy here whatever is in ​​<code>sudo visudo​ - Defaults secure_path​</code> ​​because cron doesn’t  know the path especially for root cron jobs</details>
Then add the jobs:

    0 4 * * 1 /home/dshults/update_script.sh
    @reboot /home/dshults/update_script.sh

---
### To check if the @reboot cron jobs are finished or still running
```bash
sudo service cron status      #and look under CGroup
```
---
## Monitor changes to /etc/crontab and email script
### Install [mail](http://manpages.ubuntu.com/manpages/bionic/man1/bsd-mailx.1.html) that sends emails to root

    sudo apt install mailutils

Configure as local mail. In case if need to reconfigure, run:

    sudo dpkg-reconfigure postfix

### Send test mail

    echo "Test" | mail -s "Test " root

### Check mail:

    sudo mail
>with <code>sudo</code> to check mail for root. Without sudo​ it checks mail for current user

Delete all msgs for root (or replace * with a number to del specific msg):

  `d *` in program or in terminal `echo 'd *' | sudo mail -N`

to quit `q` or to exit (which will not delete msgs) `x`

---
### Create a script

    mkdir notify_script
    cd notify_script
    nano notify_script.sh
    chmod +x notify_script.sh

script:
```bash
#!/bin/bash

original=`stat /etc/crontab -c %Z`
tmp=1550061682
if [[ $original != $tmp ]];
then
	echo "This file has been modified" | mail -s "/etc/crontab" root
	sed -i "/^tmp=/s/.*/tmp=$original/" /home/dshults/notify_script.sh
fi
```
><details><summary>sed</summary>double quotes "" so that sed can interpolate (insert) values of $original and not the word itself</details>
### Add to root crontab

    sudo crontab -e
    @midnight /home/dshults/notify_script.sh
><details><summary>@midnight</summary>or <code>@daily​</code> or <code>0 0 * * *</code></details>
---

# For corrections

Do the checksum

    shasum < Ubuntu.vdi | > submitted.txt

So that the checksum doesn’t change for multiple corrections do:

1. Create Snapshot for each correction
2. Then you can safely launch your machine.
3. After the correction is over click *Restore Snapshot* then *Delete Snapshot*
4. Then create new Snapshot for new correction
## To connect to the machine from any computer on the network
- First in your VirtualBox - Ubuntu - Settings - Network - Bridged Adapter - Attached to - set the interface through which you’re connected to the Internet at the moment (normally Ethernet)
- Then turn on auto dhcp - get the results of `netplan ip leases enp0s3` and adjust your static IP accordingly. Turn back off auto dhcp.

You're all set and ready to go ! Good luck and have fun :)