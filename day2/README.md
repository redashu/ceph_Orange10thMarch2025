# ceph_Orange10thMarch2025

### Existing storage in  real world 

<img src="st1.png">

###  Few problems with existing storage tech 

<img src="st2.png">

## Introduction to RADOS 

<img src="st3.png">

### Ceph is using LibRADOS to interact with RADOS platform 
## libsRADO can be used by Developers like python , golang 

<img src="st4.png">


### Validation of Installation with components

<img src="st5.png">


### Info about cluster 

```
ceph -s
  cluster:
    id:     73126190-66a0-425d-bc7b-971b13e67210
    health: HEALTH_WARN
            mon is allowing insecure global_id reclaim
            1 monitors have not enabled msgr2
 
  services:
    mon: 1 daemons, quorum ashu-mon (age 7m)
    mgr: no daemons active
    osd: 0 osds: 0 up, 0 in
 
  data:
    pools:   0 pools, 0 pgs
    objects: 0 objects, 0 B
    usage:   0 B used, 0 B / 0 B avail
    pgs:     
 

```

### In OSD nodes using disk to be used in Cluster 

### creating basic primary partitions in Disk 

```
[root@ashu-node1 ~]# lsblk 
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINTS
xvda    202:0    0   50G  0 disk 
├─xvda1 202:1    0    1M  0 part 
└─xvda2 202:2    0   50G  0 part /
xvdb    202:16   0  100G  0 disk 
[root@ashu-node1 ~]# fdisk   /dev/xvdb

Welcome to fdisk (util-linux 2.37.4).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): 

Using default response p.
Partition number (1-4, default 1): 
First sector (2048-209715199, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-209715199, default 209715199): +50G

Created a new partition 1 of type 'Linux' and of size 50 GiB.

Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.

```

### On each OSD prepare a disk using LVM 

```
ceph-volume  lvm prepare  --data  /dev/xvdb1 
Running command: /usr/bin/ceph-authtool --gen-print-key
Running command: /usr/bin/ceph --cluster ceph --name client.bootstrap-osd --keyring /var/lib/ceph/bootstrap-osd/ceph.keyring -i - osd new 22e145cd-c5d2-44bd-892e-f4bcf148b1aa
Running command: vgcreate --force --yes ceph-0dd2afb0-26a1-49a9-8e24-2d22be59184f /dev/xvdb1
 stdout: Physical volume "/dev/xvdb1" successfully created.
  Creating devices file /etc/lvm/devices/system.devices
 stdout: Volume group "ceph-0dd2afb0-26a1-49a9-8e24-2d22be59184f" successfully created
Running command: lvcreate --yes -l 12799 -n osd-block-


====>
ceph-volume  lvm create    --data  /dev/xvdb1 
```

### listing lvm 

```
ceph-volume  lvm list


====== osd.0 =======

  [block]       /dev/ceph-0dd2afb0-26a1-49a9-8e24-2d22be59184f/osd-block-22e145cd-c5d2-44bd-892e-f4bcf148b1aa

      block device              /dev/ceph-0dd2afb0-26a1-49a9-8e24-2d22be59184f/osd-block-22e145cd-c5d2-44bd-892e-f4bcf148b1aa
      block uuid                e4Gb3F-Uw75-OSqr-ty5N-xSne-h70P-AzJenR
      cephx lockbox secret      
      cluster fsid              73126190-66a0-425d-bc7b-971b13e67210
      cluster name              ceph
      crush device class        
      encrypted                 0
      osd fsid                  22e145cd-c5d2-44bd-892e-f4bcf148b1aa
      osd id                    0
      osdspec affinity          
      type                      block
      vdo                       0
      devices                   /dev/xvdb1

```

### Monitor server in CEPH cluster usage 

<img src="mon11.png">

### More info about Monitor map and client 

<img src="mon22.png">

## ON OSD 

### steps
i) we have disk  (use or do partitions)
ii) creating lvm using ceph-volume 
iii)  activating lvm (if required) systemctl status ceph-osd@1

### checking OSD details 

```
[root@ashu-node2 ~]# ceph osd stat
2 osds: 2 up (since 54s), 2 in (since 54s); epoch: e12
[root@ashu-node2 ~]# ceph osd tree
ID  CLASS  WEIGHT   TYPE NAME            STATUS  REWEIGHT  PRI-AFF
-1         0.09760  root default                                  
-3         0.04880      host ashu-node1                           
 1    ssd  0.04880          osd.1            up   1.00000  1.00000
-5         0.04880      host ashu-node2                           
 0    ssd  0.04880          osd.0            up   1.00000  1.00000

 ```

 ### Creating pool and checking replica

 ```
 [root@ashu-mon ~]# ceph osd  lspools 
[root@ashu-mon ~]# 
[root@ashu-mon ~]# 
[root@ashu-mon ~]# ceph osd  pool create ashu_pool1  128 
pool 'ashu_pool1' created
[root@ashu-mon ~]# ceph osd  lspools 
1 ashu_pool1
[root@ashu-mon ~]# ceph osd  pool get  ashu_pool1 size 
size: 3
[root@ashu-mon ~]# 


```
### Initial as this pool to block device pool

```
rbd  pool init ashu_pool1

```

### checking placement group in a pool

```
[root@ashu-mon ~]# ceph osd lspools
1 ashu_pool1
[root@ashu-mon ~]# 
[root@ashu-mon ~]# ceph osd pool  get ashu_pool1  pg_num 
pg_num: 128
[root@ashu-mon ~]# 

```

### Creating block device parts inside Pool 

```
[root@ashu-mon ~]# ceph osd pool  get ashu_pool1  pg_num 
pg_num: 128
[root@ashu-mon ~]# ceph osd lspools 
1 ashu_pool1
[root@ashu-mon ~]# rbd  create  --size 5G  ashu_pool1/ashu_part1 
[root@ashu-mon ~]# rbd  ls  ashu_pool1
ashu_part1
[root@ashu-mon ~]# 
[root@ashu-mon ~]# rbd  create  --size 15G  ashu_pool1/ashu_part2 
[root@ashu-mon ~]# rbd  ls  ashu_pool1
ashu_part1
ashu_part2
[root@ashu-mon ~]# rbd  create  --size 150G  ashu_pool1/ashu_os_disk 
[root@ashu-mon ~]# 
[root@ashu-mon ~]# rbd  ls  ashu_pool1
ashu_os_disk
ashu_part1
ashu_part2
[root@ashu-mon ~]# RDB ceph -- using LVM thin provisioning to over commit the size 

===> info 

[root@ashu-mon ~]# rbd  info   ashu_pool1/ashu_part1
rbd image 'ashu_part1':
	size 5 GiB in 1280 objects
	order 22 (4 MiB objects)
	snapshot_count: 0
	id: 3758bc338315
	block_name_prefix: rbd_data.3758bc338315
	format: 2
	features: layering, exclusive-lock, object-map, fast-diff, deep-flatten
	op_features: 
	flags: 
	create_timestamp: Tue Mar 11 10:07:44 2025
	access_timestamp: Tue Mar 11 10:07:44 2025
	modify_timestamp: Tue Mar 11 10:07:44 2025

```

### Getting ready with CEPH client 

```
dnf -y install centos-release-ceph-reef epel-release 
dnf install ceph-common  -y 

## Download Monitor and client key details 

[root@ip-172-31-0-33 ~]# cd /etc/ceph/
[root@ip-172-31-0-33 ceph]# ls
rbdmap
[root@ip-172-31-0-33 ceph]# scp  172.31.4.54:/etc/ceph/ceph.conf  /etc/ceph/
The authenticity of host '172.31.4.54 (172.31.4.54)' can't be established.
ED25519 key fingerprint is SHA256:JdD6/h2vnpW3Wzhlg6/WMaGRzJsiZOVITSuX0wPZxkA.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.31.4.54' (ED25519) to the list of known hosts.
root@172.31.4.54's password: 
ceph.conf                                                                                                  100%  646   335.6KB/s   00:00    
[root@ip-172-31-0-33 ceph]# scp  172.31.4.54:/etc/ceph/ceph.client.admin.keyring  /etc/ceph/
root@172.31.4.54's password: 
ceph.client.admin.keyring                                                                                  100%  151    58.1KB/s   00:00    
[root@ip-172-31-0-33 ceph]# ls
ceph.client.admin.keyring  ceph.conf  rbdmap
[root@ip-172-31-0-33 ceph]# ls -lh 
total 12K
-rw-------. 1 root root 151 Mar 11 11:26 ceph.client.admin.keyring
-rw-r--r--. 1 root root 646 Mar 11 11:26 ceph.conf
-rw-r--r--. 1 root root  92 Sep 23 17:01 rbdmap
[root@ip-172-31-0-33 ceph]#  

====>
  10   chown ceph:ceph  ceph.* 
   11  ls -l 
   12  ceph -s
   13  ceph osd lspools 
   14  rbd ls ashu_pool1
   15  rbd info  ashu_pool1/ashu_part1
```
