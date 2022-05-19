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

