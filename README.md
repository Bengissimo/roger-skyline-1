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
    address 10.11.200.86/30
    gateway 10.11.254.254
```
Here I picked a random IP which does not belogn any computer in the cluster, and not used by anyone else (to check this I just used ping). With a Netmask \30 configuration the number of usable IP is two. I picked a network address 10.11.200.84 and set my IP to 10.11.200.86.

Restart networking service to update network interface and check the current state and verify internet connection:
```
$ sudo service networking restart
$ ip a
$ ping google.com
```

### 3. SSH configuration

### 4. Firewall configuration

### 5. Fail2ban configuration

### 6. Protection against port scans

### 7. Stopping the unnecessary services

### 8. A script to update packages

### 9. A script to monitor changes in /etc/crontab file to notify root via local email service

## Web part

## Deploymenyt part








