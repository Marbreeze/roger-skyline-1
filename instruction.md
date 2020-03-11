# **************************************************************************** #
#                                                                              #
#                                                         :::      ::::::::    #
#    instruction.md                                     :+:      :+:    :+:    #
#                                                     +:+ +:+         +:+      #
#    By: mstratu <marvin@42.fr>                     +#+  +:+       +#+         #
#                                                 +#+#+#+#+#+   +#+            #
#    Created: 2019/12/17 21:22:00 by mstratu           #+#    #+#              #
#    Updated: 2019/12/17 21:23:59 by mstratu          ###   ########.fr        #
#                                                                              #
# **************************************************************************** #


ROGER_SKYLINE1

#V1.Followed Slash16 on Facebook, Twitter and Linkedin.

#instalation
#system
	Debian10

#hardware
	Disk size 8GB
	One 4.3 partition

#network specification
	static IP address 198.168.99.1/24
	ssh listens on port 220
	apache listens to port: 80(HTTP)

#user management
	root - root user
	marina - user, added to root and sudo

#how to add user
	adduser<marina>

#make it sudo & root
	adduser <username>sudo
	adduser <username>root

# if you want to modify other users: 
#check nano /etc/passwd & change user UID & GID to 0 

## other
I used Virtual Box and downloaded the iso image of the latest [Debian10] <https://cdimage.debian.org/debian-cd/current/amd64/iso-cd>
During the installation process, made disc partition 4.3 GB mounted on / root and 4.3GB of swap area

<https://debian-handbook.info/browse/stable/sect.installation-steps.html>  
OR
<http://www.brianlinkletter.com/installing-debian-linux-in-a-virtualbox-virtual-machine/>

# V2. Network and security part

#install Dependecies 

To have all the currencies updated run:
*
apt -y update
apt -y upgrade
*
#note do it separetelly
apt-get install -y sudo net-tools iptables-persistent fail2ban sendmail apache2 cron vim ufw nginx

# configure SUDO
#Login as root user
*
su
nano /etc/ssh/sshd_config
*
# add new lines OR change lines (uncomment them to make active) into: 

Port 220
PasswordAuthentification yes
PermitRootLogin no           #if you want to have permision set it to yes and pub-auth no!
PubkeyAuthentication yes
*

# setup a static IP

#- to find ip (https://vitux.com/find-debian-ip-network-address)

*
ip a
or
ip addr | grep 'inet'
*

#- to enable [network_adapter] in VM (https://cs4118.github.io/dev-guides/host-only-network.html).

*
nano /etc/network/interfaces
*

https://www.youtube.com/watch?v=j1TzvIXRNqs

```
source /etc/network/interfaces.d/*
auto lo
iface lo inet loopback
# allow-hotplug enp0s3 
# iface enp0s3 inet dhcp
auto enp0s8 
iface enp0s8 inet static
    address 192.168.56.1/24
    #gateway 192.168.56.102
    #network 192.168.56.100 
    netmask 255.255.255.0
	```
#the mandatory variables are: address and netmask! 

#Restart the network service to make changes effective
```
sudo service networking restart

ip addr
```
#We can now login users with ssh

```
ssh <user>@debian -p 222 


ssh <user>@192.168.56.1/24 -p 220
```
## Setup public keys acces to ssh
#Generate the rsa key
```
ssh-keygen
```
#This command will generate 2 files id_rsa and id_rsa.pub
#id_rsa: Our private key, should be keep safely, it can be crypted with a password.
#id_rsa.pub Our private key, you have to transfer this one to the server.

To copy id_rsa
```
ssh-copy-id -i id_rsa.pub <user>@debian/IP -p 222
OR
mkdir .ssh
~/.ssh/id_rsa.pub > .ssh/authorized_keys
```
Edit the sshd_config file /etc/ssh/sshd.config to remove root login permit, password authentification
```
sudo vim /etc/ssh/sshd.conf
#Edit line like: PermitRootLogin no
#Edit line like PasswordAuthentication no
```
Don't forget to delete de # at the beginning of each line. Now we can run ssh without password.
```
sudo service ssh restart
```
Try again:
```
#ssh <user>@debian/IP -p 220
ssh<user>@debian -p 220 
```


- to fix the problem with pub key [access] if appeared (<https://www.digitalocean.com/community/questions/error-permission-denied-publickey-when-i-try-to-ssh>)

    ## Setup Firewall
```
sudo nano /etc/network/if-pre-up.d/iptables

//enter this:
#! / Bin / bash
iptables-restore </etc/iptables.test.rules
iptables -F iptables -X iptables -t nat -F iptables -t nat -X iptables -t mangle -F iptables -t mangle -X
iptables -P INPUT DROP
iptables -P OUTPUT DROP
iptables -P FORWARD DROP
iiptables -A INPUT -m conntrack -ctstate ESTABLISHED, RELATED -j ACCEPT
iptables -A INPUT -p tcp -i enp0s8 -dport 22 -j ACCEPT
iptables -A INPUT -p tcp -i enp0s8 -dport 80 -j ACCEPT
iptables -A INPUT -p tcp -i enp0s8 -dport 443 -j ACCEPT
iptables -A OUTPUT -m conntrack! --ctstate INVALID -j ACCEPT
iptables -I INPUT -i lo -j ACCEPT
iptables -A INPUT -j LOG
iptables -A FORWARD -j LOG
iptables -I INPUT -p tcp -dport 80 -m connlimit -connlimit-above 10 -connlimit-mask 20 -j DROP

#port scan
iptables -N port-scanning
iptables -A port-scanning -p tcp --tcp-flags SYN,ACK,FIN,RST RST -m limit --limit 60/s --limit-burst 2 -j RETURN
iptables -A port-scanning -j DROP

sudo chmod +x /etc/network/if-pre-up.d/iptables


## Mail configurations

echo "Mail configurations"
# Allow SMTP
echo "   -- Allow SMTP"
iptables -t filter -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -t filter -A OUTPUT -p tcp --dport 22 -j ACCEPT
# Allow POP3
echo "   -- Allow POP3"
iptables -t filter -A INPUT -p tcp --dport 110 -j ACCEPT
iptables -t filter -A OUTPUT -p tcp --dport 110 -j ACCEPT
# Allow IMAP
echo "   -- Allow IMAP"
iptables -t filter -A INPUT -p tcp --dport 143 -j ACCEPT
iptables -t filter -A OUTPUT -p tcp --dport 143 -j ACCEPT
# Allow POP3S
echo "   -- Allow POP3S"
iptables -t filter -A INPUT -p tcp --dport 995 -j ACCEPT
iptables -t filter -A OUTPUT -p tcp --dport 995 -j ACCEPT



```
## ---not  mandatory--DDoS protections
#
echo "🛡  DDoS protetion"

# Synflood protection
echo "   -- Synflood protection"
/sbin/iptables -A INPUT -p tcp --syn -m limit --limit 2/s --limit-burst 30 -j ACCEPT
# Pingflood protection
echo "   -- Pingflood protection"
/sbin/iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 1/s -j ACCEPT
# Port scanning protection
echo "   -- Port scanning protection"
/sbin/iptables -A INPUT -p tcp --tcp-flags ALL NONE -m limit --limit 1/h -
## Save the tables configuration
#
iptables-save > /etc/iptables/rules.v4
```

##  can skip --Alternative firewall with ufw

ufw enable
ufw default deny incoming
ufw default allow outgoing
ufw allow 222
ufw allow 443
ufw allow 80
ufw allow 'Nginx Full'
ufw reload
#-------------

## !!!!!!!!!!!!!Setup logs

Creates log and send info if files were changed and protects from ddos
```
sudo touch /var/log/apache2/server.log
```
```
sudo nano /etc/fail2ban/jail.local
```
[DEFAULT] destemail = USER@student.42.us.org sender = root@debian

#just to make sure do it again but now in crontab:
```
sudo nano/etc/cron.d/packages.sh
```
```
sudo nano /etc/cron.d/survey.sh
```
```
if [[ $(($(date +%s) - $(date +%s -r /etc/crontab))) -lt 86400 ]]
then
	echo "Crontab file has been modified" | sudo /usr/sbin/sendmail root
fi
```
crontab -e

```
0 4 * * 1 /etc/cron.d/packages.sh
@reboot /etc/cron.d/packages.sh
0 0 * * * /etc/cron.d/survey.sh
```

## Update crontab

Create a script:
```
sudo vim /root/scripts/update_script.sh
```
With the following lines:

```
#!/bin/bash
apt update -y >> /var/log/update_script.log
apt upgrade -y >> /var/log/update_script.log
```

Give it some permissions:
```
sudo chmod 755 /root/scripts/update_script.sh
```
And make root be the owner for automated execution:
```
sudo chown root /root/scripts/update_script.sh
```
To automate execution, we must edit the crontab file:
```
sudo crontab -e
```
To which we add the lines:
```
0 4 * * wed root /root/scripts/update_script.sh
@reboot root /root/scripts/update_script.sh
```

##  Making a script to warn of all crontab edits

Create a script :
```
sudo vim /root/scripts/crontab_canary.sh
```
With the following lines :
```
#!/bin/bash
DIFF=$(diff /etc/crontab.bak /etc/crontab)

```
```
cat /etc/crontab > /etc/crontab.bak
```
```
if [ "$DIFF" != ""] then
    echo "crontab check: changed, notifying admin." | sudo /usr/sbin/sendmail root
else
    echo "crontab check: unchanged."
fi
```
Give it the appropriate execution permessions:
```
sudo chmod 755 /root/scripts/script_crontab.sh
```
Make it owned by root :
```
sudo chown root /root/scripts/script_crontab.sh
```
And edit crontab by adding the following line:
```
0 0 * * * root /root/scripts/crontab_canary.sh
```
## Port scan protection
portsentry, test from host with
```
nmap [vm-IP]
and
nmap -Pn [vm-IP]
```
After each test (they are long), check the file /etc/hosts.deny for the host IP (it should have been added by portsentry), remove it, and reboot, or else you won't be able to ssh into the vm from the host.
## Stopping useless services
```
sudo service --status-all
```
#finish
