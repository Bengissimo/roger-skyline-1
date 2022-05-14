# roger-skyline-1
Second system and network administration project of Hive Helsinki. With this project I configured a web server on a virtual machine from scratch, made it secure against possible attacks, created a login page and figured out a way for deployment automation.  

## VM set up
I set up a virtual machine with Debian Linux operating system with a disk size of 8 GB and a 4.2 GB partition.

## Network and security
### 1. Create a non-root user with sudo priviliges
``` 
$ su - 
# apt update -y && apt upgrade -y
# apt install sudo vim -y
# usermod -aG sudo <username>
# exit 
```

To check if the user in sudo group now:
```
$ groups
```
### 2. configure a static IP with a Netmask in \30
To be able to use the network, change the VM adapter setting to Bridged Adapter, en0: Ethernet.
Next, [configure a static IP.](https://wiki.debian.org/NetworkConfiguration#Configuring_the_interface_manually)
```
$ sudo vim /etc/network/interfaces
```
```
# The primary network interface
auto enp0s3
```
```
$ sudo vim /etc/network/interfaces.d/enp0s3
```
```
iface enp00s3 inet static
    address 10.11.249.86
    netmask 255.255.255.252
    gateway 10.11.254.254
```

Here I picked a random IP which does not belogn any computer in the cluster, and not used by anyone else (to check this I just used ping). With a Netmask \30 configuration the number of usable IP is two. I picked a network address 10.11.249.84 and set my IP to 10.11.200.86.

Restart networking service to update network interface and check the current state and verify internet connection:
```
$ sudo service networking restart
$ ip a
$ ping google.com
```

### 3. SSH configuration
Changing the default port 22:
```
$ sudo vim /etc/ssh/sshd_config
```
Uncomment #Port 22 and change 22 to another number. Prefer something between [49152 and 65535](http://www.steves-internet-guide.com/tcpip-ports-sockets/)

Configure publickey authentication:
Copyy the public key into ~/.ssh/authorized_keys file:
```
$ cat ~/.ssh/id_rsa.pub | ssh remote_username@server_ip_address "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```
And update the sshd_config file accordingly:
- PermitRootLogin no
- PasswordAuthentication no
Restart ssh service and try connecting via ssh to see the new settings work properly.

### 4. Firewall configuration
I installed [Uncomplicated Firewall](https://wiki.debian.org/Uncomplicated%20Firewall%20%28ufw%29#Port_Ranges)
```
$ sudo apt install ufw
$ sudo ufw enable
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw status verbose
$ sudo ufw allow <portnumber>/tcp
$ sudo ufw allow 443
$ sudo ufw allow 80/tcp
$ sudo ufw enable
$ sudo ufw status verbose
```

### 5. Fail2ban configuration to protect against denial of servicce attacks
I chose to use [Fail2ban](https://www.digitalocean.com/community/tutorials/how-fail2ban-works-to-protect-services-on-a-linux-server) service. Fail2ban basically monitors the logs of common services and spot patterns in failed attemps using specific filters. If there is a match, an action is executed for that service. For example blocks the IP if it exceeds the treshhold of max retry. Here is how I configured Fail2ban for:

```
$ sudo apt-get update
$ sudo apt-get install fail2ban
```
To copy the contents of jail.conf with all contents, including the ones commented out:
```
$ awk '{ printf "# "; print; }' /etc/fail2ban/jail.conf | sudo tee /etc/fail2ban/jail.local
```
I did not change the default settings but changed [sshd] setting, did not change filter for sshd and added [http-get-dos]:
```
$ sudo vim /etc/fail2ban/jail.local

"
[sshd]
mode   = aggressive
enabled = true
port    = 49786
maxretry = 3
findtime = 300
bantime = 600
logpath = %(sshd_log)s
backend = %(sshd_backend)s

[http-get-dos]
enabled = true
port = http,https
filter = http-get-dos
logpath = /var/log/nginx/access.log
maxretry = 300
findtime = 300
bantime = 600
action = iptables[name=HTTP, port=http, protocol=tcp]
"

$ sudo vim /etc/fail2ban/filter.d/http-get-dos.conf

"
[Definition]

failregex = ^<HOST> -.*"(GET|POST).*

ignoreregex =
"
```
Activate jails and check info:
```
$ sudo service fail2ban restart
$ sudo fail2ban-client status
```
To test if Fail2ban configuration:
I used [slowloris](https://github.com/gkbrk/slowloris) for DoS attack and tried to connect via ssh from another computer lacking the ssh key.
In both of the cases attacking IPs were blocked successfully.

### 6. Protection against port scans
I installed [portsentry](https://en-wiki.ikoula.com/en/To_protect_against_the_scan_of_ports_with_portsentry)

```
$ sudo apt-get update && apt-get install portsentry
```
To use the portsentyr in advanced mode, following rules were changed:
```
$ sudo vim /etc/default/portsentry
```
```
TCP_MODE="atcp"
UDP_MODE="audp"
```
To ban the IP addresses attempting port scan:
```
$ sudo vim /etc/portsentry/portsentry.conf
```
```
BLOCK_UDP="1"
BLOCK_TCP="1"
```
```
# iptables support for Linux
KILL_ROUTE="/sbin/iptables -I INPUT -s $TARGET$ -j DROP"
```
To unban the IP, this command can be used and it is a good practice to check /etc/hosts.deny file to see/remove banned IPs:
```
iptables -D INPUT -s 178.170.xxx.xxx -j DROP
```
To test if portsentry protects against port scans, I used Nmap:
```
nmap <IP>
```
```
nmap -sT <IP>
```
### 7. Stopping the unnecessary services


### 8. A script to update packages
```
$ cd /usr/local/bin/
$ touch auto_update.sh
$ chmod 0755 auto_update.sh
$ vim auto_update.sh
```
```
#!/bin/bash

sudo apt-get update -y >> /var/log/update_script.log
sudo apt-get upgrade -y >> /var/log/update_script.log
```
Add to crontab:
```
$ sudo crontab -e
```
```
0 4 * * 0 /usr/local/bin/auto_update.sh
@reboot /usr/local/bin/auto_update.sh
```
### 9. A script to monitor changes in /etc/crontab file to notify root via local email service
monitor_crontab.sh script:
```
#!/bin/bash

DIFF=$(diff /etc/crontab.back /etc/crontab)
cat /etc/crontab > /etc/crontab.back
if [ "$DIFF" != "" ]; then
	echo "change detected in crontab, root will be notified" | mail -s "crontab modified" root
fi
```
To use the local mail system, I installed mailutils and postfix:
```
$ apt install mailutils postfix
```
For postfix settings, I chose local only and set system mail name to debian.lan. Then edited /etc/aliases:
```
root: root@debian.lan
```
to enable alias change:
```
$ sudo newaliases
```
## Web part
I did a pretty basic login page using html, php and css.
[Resource](https://www.php.net/manual/en/tutorial.forms.php)
I have three files for the login page
- index.html
- action.php
- styles.css

For self signed ssl certificate I followed [these instructions](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-in-ubuntu-16-04)

## Deploymenyt part
I wrote a script for deployment which basicly move files through ssh nad update the old ones if there is a change.








