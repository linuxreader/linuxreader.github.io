## NFS DIY Labs

### Lab: Configure NFS Share and Automount with Direct Map

- As user1 with sudo on server30, share directory /sharenfs (create it) in read/write mode using NFS. 
```bash
[root@server30 /]# mkdir /sharenfs
[root@server30 /]# chmod 777 /sharenfs
[root@server30 /]# vim /etc/exports

# Add -> /sharenfs server40(rw)

[root@server30 /]# dnf -y install nfs-utils
[root@server30 /]# firewall-cmd --permanent --add-service nfs
[root@server30 /]# firewall-cmd --reload
success

[root@server30 /]# systemctl --now enable nfs-server


[root@server30 /]# exportfs -av
exporting server40.example.com:/sharenfs
```

- On server40 as user1 with sudo, install the AutoFS software and start the service. 
```bash
[root@server40 nfshome]# dnf -y install autofs
```
- Configure the master and a direct map to automount the share on /mntauto (create it). 
```bash
[root@server40 ~]# vim /etc/auto.master
/- /etc/auto.master.d/auto.dir

[root@server40 ~]# vim /etc/auto.master.d/auto.dir
/mntauto server30:/sharenfs

[root@server40 /]# mkdir /mntauto

[root@server40 ~]# systemctl enable --now autofs

```


- Run ls on /mntauto to trigger the mount. 
```bash
[root@server40 /]# mount | grep mntauto
/etc/auto.master.d/auto.dir on /mntauto type autofs (rw,relatime,fd=10,pgrp=6211,timeout=300,minproto=5,maxproto=5,direct,pipe_ino=40247)
server30:/sharenfs on /mntauto type nfs4 (rw,relatime,vers=4.2,rsize=262144,wsize=262144,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,clientaddr=192.168.0.40,local_lock=none,addr=192.168.0.30)
```

- Use `df -h` to confirm. 
```
[root@server40 /]# df -h | grep mntauto
server30:/sharenfs      17G  2.2G   15G  13% /mntauto
```

###  Lab: Automount NFS Share with Indirect Map

- As user1 with sudo on server40, configure the master and an indirect map to automount the share under /autoindir (create it). 
```bash
[root@server40 /]# mkdir /autoindir

[root@server40 etc]# vim /etc/auto.master
/autoindir /etc/auto.misc

[root@server40 etc]# vim /etc/auto.misc
sharenfs server30:/common

[root@server40 etc]# systemctl restart autofs

```

- Run ls on /autoindir/sharenfs to trigger the mount. 
```bash
[root@server40 etc]# ls /autoindir/sharenfs
test.text
```
- Use `df -h` to confirm.
```bash
[root@server40 etc]# df -h
Filesystem             Size  Used Avail Use% Mounted on
devtmpfs               4.0M     0  4.0M   0% /dev
tmpfs                  888M     0  888M   0% /dev/shm
tmpfs                  356M  5.1M  351M   2% /run
/dev/mapper/rhel-root   17G  2.2G   15G  13% /
/dev/sda1              960M  344M  617M  36% /boot
tmpfs                  178M     0  178M   0% /run/user/0
server30:/common        17G  2.2G   15G  13% /autoindir/sharenfs
```