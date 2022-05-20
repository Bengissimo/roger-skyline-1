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
groups
```
### 2. configure a static IP with a Netmask in \30
To be able to use the network, change the VM adapter setting to Bridged Adapter, en0: Ethernet.
Next, [configure a static IP.](https://wiki.debian.org/NetworkConfiguration#Configuring_the_interface_manually)
```
sudo vim /etc/network/interfaces
```
```
The primary network interface
auto enp0s3
```
```
sudo vim /etc/network/interfaces.d/enp0s3
```
```
iface enp00s3 inet static
    address 10.11.239.86
    netmask 255.255.255.252
    gateway 10.11.254.254
```

Here I picked a random IP which does not belogn any computer in the cluster, and not used by anyone else (to check this I just used ping). With a Netmask \30 configuration the number of usable IP is two. I picked a network address 10.11.239.84 and set my IP to 10.11.239.86.

Restart networking service to update network interface and check the current state and verify internet connection:
```
sudo service networking restart
ip a
ping google.com
```

### 3. SSH configuration
Changing the default port 22:
```
sudo vim /etc/ssh/sshd_config
```
Uncomment #Port 22 and change 22 to another number. Prefer something between [49152 and 65535](http://www.steves-internet-guide.com/tcpip-ports-sockets/)

Configure publickey authentication:
Copyy the public key into ~/.ssh/authorized_keys file:
```
cat ~/.ssh/id_rsa.pub | ssh remote_username@server_ip_address "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```
And update the sshd_config file accordingly:
- PermitRootLogin no
- PasswordAuthentication no
Restart ssh service and try connecting via ssh to see the new settings work properly.

### 4. Firewall configuration
I installed [Uncomplicated Firewall](https://wiki.debian.org/Uncomplicated%20Firewall%20%28ufw%29#Port_Ranges)
```
sudo apt install ufw
sudo ufw enable
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw status verbose
sudo ufw allow <portnumber>/tcp
sudo ufw allow 443
sudo ufw allow 80/tcp
sudo ufw enable
sudo ufw status verbose
```

### 5. Fail2ban configuration to protect against denial of servicce attacks
First I installed [nginx].(https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-debian-10)

I chose to use [Fail2ban](https://www.digitalocean.com/community/tutorials/how-fail2ban-works-to-protect-services-on-a-linux-server) service. Fail2ban basically monitors the logs of common services and spot patterns in failed attemps using specific filters. If there is a match, an action is executed for that service. For example blocks the IP if it exceeds the treshhold of max retry. Here is how I configured Fail2ban for:

```
sudo apt-get update
sudo apt-get install fail2ban
```
To copy the contents of jail.conf with all contents, including the ones commented out:
```
awk '{ printf "# "; print; }' /etc/fail2ban/jail.conf | sudo tee /etc/fail2ban/jail.local
```
I did not change the default settings but changed [sshd] setting, did not change filter for sshd and added [http-get-dos]:
```
sudo vim /etc/fail2ban/jail.local

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

sudo vim /etc/fail2ban/filter.d/http-get-dos.conf

"
[Definition]

failregex = ^<HOST> -.*"(GET|POST).*

ignoreregex =
"
```
Activate jails and check info:
```
sudo service fail2ban restart
sudo fail2ban-client status
```
To test if Fail2ban configuration:
I used [slowloris](https://github.com/gkbrk/slowloris) for DoS attack and tried to connect via ssh from another computer lacking the ssh key.
In both of the cases attacking IPs were blocked successfully.
to confirm banned IPs, check this out:
```
sudo fail2ban-client status http-get-dos
sudo fail2ban-client status sshd
```

To unban IP:
```
sudo fail2ban-client set sshd unbanip <IP>
```
### 6. Protection against port scans
I installed [portsentry](https://en-wiki.ikoula.com/en/To_protect_against_the_scan_of_ports_with_portsentry)

```
sudo apt-get update && apt-get install portsentry
```
To use the portsentyr in advanced mode, following rules were changed:
```
sudo vim /etc/default/portsentry
```
```
TCP_MODE="atcp"
UDP_MODE="audp"
```
To ban the IP addresses attempting port scan:
```
sudo vim /etc/portsentry/portsentry.conf
```
```
BLOCK_UDP="1"
BLOCK_TCP="1"
```
Uncomment the following line and comment on all the other options starting with "KILL_ROUTE".
```
# iptables support for Linux
KILL_ROUTE="/sbin/iptables -I INPUT -s $TARGET$ -j DROP"
```
Comment out the KILL_HOSTS_DENY. If this option is active, the first nmap attack is banned. But the consequent attacks were not banned even though I unbanned the IP using iptables command and remove the IP from /etc/hosts.deny file. Somehow this IP is remembered.
```
#KILL_HOSTS_DENY="ALL: $TARGET$"
#KILL_HOSTS_DENY="ALL: $TARGET$ : DENY"
```
Restart the portsentry:
```
sudo service portsentry restart
```
To test if portsentry protects against port scans, I used Nmap.
```
nmap <IP>
```
```
nmap -sT <IP>
```
To check if the IP using Nmap is blocked:
```
sudo tail -f /var/log/syslog
```
To unban the IP, this iptables command can be used. Also remove the banned IP from /etc/hosts.deny file:
```
iptables -D INPUT -s 178.170.xxx.xxx -j DROP
sudo vim /etc/hosts.deny
```
### 7. Stopping the unnecessary services

check the enabled services with the following command:
```
sudo systemctl list-unit-files --type service --state=enabled
```
I had the following list:
```
UNIT FILE              STATE   VENDOR PRESET
apparmor.service       enabled enabled
console-setup.service  enabled enabled
cron.service           enabled enabled
e2scrub_reap.service   enabled enabled
fail2ban.service       enabled enabled
getty@.service         enabled enabled
keyboard-setup.service enabled enabled
networking.service     enabled enabled
nginx.service          enabled enabled
ntp.service            enabled enabled
postfix.service        enabled enabled
rsyslog.service        enabled enabled
ssh.service            enabled enabled
systemd-pstore.service enabled enabled
ufw.service            enabled enabled

15 unit files listed.
```
I disabled console-setup, keyboard-setup services and ntp.service:
```
sudo systemctl disable console-setup.service
sudo systemctl disable keyboard-setup.service
sudo systemctl disable ntp.service
```
### 8. A script to update packages
```
cd /usr/local/bin/
sudo touch auto_update.sh
sudo chmod 0755 auto_update.sh
vim auto_update.sh
```
```
#!/bin/bash

sudo apt-get update -y >> /var/log/update_script.log
sudo apt-get upgrade -y >> /var/log/update_script.log
```
Add to crontab:
```
sudo crontab -e
```
```
0 4 * * 0 /usr/local/bin/auto_update.sh
@reboot /usr/local/bin/auto_update.sh
```
### 9. A script to monitor changes in /etc/crontab file to notify root via local email service
To use the local mail system, I installed mailutils and postfix:
```
sudo apt install mailutils postfix
```
For postfix settings, I chose local only and set system mail name to debian.lan. Then edited /etc/aliases:
```
root: root@debian.lan
```
to enable alias change:
```
sudo newaliases
```
To test if the local mail service is working properly:
```
echo "sth to try" | mail -s "this is a mail trial" root
sudo mail
```
If you see '"/var/mail/root": 1 message 1 new', all good. 

monitor_crontab.sh script:
```
#!/bin/bash

DIFF=$(diff /etc/crontab.back /etc/crontab)
cat /etc/crontab > /etc/crontab.back
if [ "$DIFF" != "" ]; then
	echo "change detected in crontab, root will be notified" | mail -s "crontab modified" root
fi
```
Crontab schedule at midnight to check if the /etc/crontab changed:
```
sudo crontab -e
add the following line
0 0 * * * /usr/local/bin/monitor_crontab.sh
```

## Web part
I did a pretty basic login page using html, php and css.
I installed nginx (it has been already installed to test Fail2ban) and php-fpm.
```
sudo apt install php-fpm
```
First I modified the default conf file for the settings asked for this project.
```
sudo vim /etc/nginx/sites-available/default
```
Uncomment arrow pointed lines seen below:
```
	# pass PHP scripts to FastCGI server
	#
->	#location ~ \.php$ {
->	#	include snippets/fastcgi-php.conf;
	#
	#	# With php-fpm (or other unix sockets):
->	#	fastcgi_pass unix:/run/php/php7.4-fpm.sock;
	#	# With php-cgi (or other tcp sockets):
	#	fastcgi_pass 127.0.0.1:9000;
->	#}

```
Later I just renamed the default file:
```
cd /etc/nginx/sites-enabled/
sudo mv default my_login_page
```
restart ```sudo systemctl restart nginx.service```
[Resource](https://www.php.net/manual/en/tutorial.forms.php)

I have three files for the login page at /var/www/html/
- index.html
- action.php
- styles.css

For self signed ssl certificate I followed [these instructions](https://www.digitalocean.com/community/tutorials/how-to-create-a-self-signed-ssl-certificate-for-nginx-in-ubuntu-16-04) and modified accrodingly the /etc/nginx/sites-enabled/my_login_page

## Deploymenyt part
I wrote a script for deployment which basicly move files through ssh and update the old ones if there is a change.
