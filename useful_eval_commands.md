## check that Traefik, Docker or Vagrant are not installed 
```
sudo apt list –installed  | grep ‘docker\|vagrant\|traefik’
```
## check disk size and partition
- sudo parted
- lsblk
- sudo fdisk -l

## Are packages up to date?
```
sudo apt-get upgrade
```

## Which packages are installed?
```
sudo apt list --installed | less
```
## Create a user, who is part of the sudo group, with SSH key to be able to connect to the VM
```
sudo adduser
```
Check the new_user:
```
awk -F: '$3 >= 1000 && $1 != "nobody" {print $1}' /etc/passwd
```
Give sudo privilages:
```
su -
```
```
usermmod -aG sudo new_user
```
Login with the newuser, create ~/.ssh directory, copy the authorized key there:
```
mkdir ~/.ssh
cp /home/bkandemi/.ssh/authorized_keys /home/new_user/.ssh/
sudo chmod 644 /home/new_user/.ssh/authorized_keys
```
Try to connect via SSH from client side:
```
ssh -p <port_nb> new_user@IP
```
## check ssh configurations
```
sudo cat /etc/ssh/sshd_config
```

## root should not connect via ssh
```
ssh -p 49786 root@10.11.239.86
```
See -> root@10.11.239.86: Permission denied (publickey).
## List all firewall rules
```
sudo ufw status verbose
```

## Slowloris attack
```
cd /Users/bkandemi/bkandemi_workspace/slowloris
python3 slowloris.py 10.11.239.86
```
check the jail for http-get-dos
```
sudo fail2ban-client status http-get-dos
```
## List all open ports
```
sudo lsof -Pni
```
or 
```
netstat -tunlp
```
## Check active services
```
sudo systemctl list-unit-files --type service --state=enabled
```
## Check update script and crontab
```
cat /usr/local/bin/auto_update.sh
sudo crontab -e
```
## Check monitor_crontab script
```
cat /usr/local/bin/monitor_crontab.sh
sudo crontab -e
```
```
sudo vim /etc/crontab
```
run the monitor_crontab.sh script and check mail
```
sudo mail
```
## check that there is only one active configuration on the web server and. not the default one. in addtion, it should not "Listen" on the localhost of the VM.

```
netstat --listening
or 
sudo lsof -Pni
```
##





