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
auto enp0s3
iface enp00s3 inet static
    address 10.11.249.86/30
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
```
$ sudo vim /etc/default/portsentry
```

### 7. Stopping the unnecessary services

### 8. A script to update packages

### 9. A script to monitor changes in /etc/crontab file to notify root via local email service

## Web part

## Deploymenyt part








