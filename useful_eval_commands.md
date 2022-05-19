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
```
this is what I see now:
```
bkandemi@debian:~$ netstat --listening
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 0.0.0.0:http            0.0.0.0:*               LISTEN
tcp        0      0 localhost:smtp          0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:49786           0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:https           0.0.0.0:*               LISTEN
tcp6       0      0 [::]:http               [::]:*                  LISTEN
tcp6       0      0 localhost:smtp          [::]:*                  LISTEN
tcp6       0      0 [::]:49786              [::]:*                  LISTEN
tcp6       0      0 [::]:https              [::]:*                  LISTEN
raw        0      0 0.0.0.0:tcp             0.0.0.0:*               7
raw        0      0 0.0.0.0:udp             0.0.0.0:*               7
Active UNIX domain sockets (only servers)
Proto RefCnt Flags       Type       State         I-Node   Path
unix  2      [ ACC ]     STREAM     LISTENING     15186    public/qmgr
unix  2      [ ACC ]     STREAM     LISTENING     15190    private/tlsmgr
unix  2      [ ACC ]     STREAM     LISTENING     15983    /run/user/1000/systemd/private
unix  2      [ ACC ]     STREAM     LISTENING     10599    /run/systemd/private
unix  2      [ ACC ]     STREAM     LISTENING     10601    /run/systemd/userdb/io.systemd.DynamicUser
unix  2      [ ACC ]     STREAM     LISTENING     10602    /run/systemd/io.system.ManagedOOM
unix  2      [ ACC ]     STREAM     LISTENING     10612    /run/systemd/fsck.progress
unix  2      [ ACC ]     STREAM     LISTENING     10620    /run/systemd/journal/stdout
unix  2      [ ACC ]     SEQPACKET  LISTENING     10622    /run/udev/control
unix  2      [ ACC ]     STREAM     LISTENING     11810    /run/php/php7.4-fpm.sock
unix  2      [ ACC ]     STREAM     LISTENING     14534    /var/run/fail2ban/fail2ban.sock
unix  2      [ ACC ]     STREAM     LISTENING     15211    private/proxymap
unix  2      [ ACC ]     STREAM     LISTENING     10758    /run/systemd/journal/io.systemd.journal
unix  2      [ ACC ]     STREAM     LISTENING     15202    private/trace
unix  2      [ ACC ]     STREAM     LISTENING     15205    private/verify
unix  2      [ ACC ]     STREAM     LISTENING     15208    public/flush
unix  2      [ ACC ]     STREAM     LISTENING     15214    private/proxywrite
unix  2      [ ACC ]     STREAM     LISTENING     15217    private/smtp
unix  2      [ ACC ]     STREAM     LISTENING     15220    private/relay
unix  2      [ ACC ]     STREAM     LISTENING     15193    private/rewrite
unix  2      [ ACC ]     STREAM     LISTENING     15223    public/showq
unix  2      [ ACC ]     STREAM     LISTENING     15226    private/error
unix  2      [ ACC ]     STREAM     LISTENING     15229    private/retry
unix  2      [ ACC ]     STREAM     LISTENING     15232    private/discard
unix  2      [ ACC ]     STREAM     LISTENING     15235    private/local
unix  2      [ ACC ]     STREAM     LISTENING     15238    private/virtual
unix  2      [ ACC ]     STREAM     LISTENING     15241    private/lmtp
unix  2      [ ACC ]     STREAM     LISTENING     15244    private/anvil
unix  2      [ ACC ]     STREAM     LISTENING     15179    public/pickup
unix  2      [ ACC ]     STREAM     LISTENING     15247    private/scache
unix  2      [ ACC ]     STREAM     LISTENING     15253    private/maildrop
unix  2      [ ACC ]     STREAM     LISTENING     15256    private/uucp
unix  2      [ ACC ]     STREAM     LISTENING     15259    private/ifmail
unix  2      [ ACC ]     STREAM     LISTENING     15262    private/bsmtp
unix  2      [ ACC ]     STREAM     LISTENING     15265    private/scalemail-backend
unix  2      [ ACC ]     STREAM     LISTENING     15268    private/mailman
unix  2      [ ACC ]     STREAM     LISTENING     15196    private/bounce
unix  2      [ ACC ]     STREAM     LISTENING     15183    public/cleanup
unix  2      [ ACC ]     STREAM     LISTENING     15199    private/defer
unix  2      [ ACC ]     STREAM     LISTENING     11424    /run/dbus/system_bus_socket
```




