### software define storage 

### basic info 

<img src="info1.png">



### storage access options 

<img src="st1.png">

### chef cluster setup with container support 

<img src="st2.png">

### simple ceph architecture is like 

<img src="st3.png">

### setup 3 node ceph cluster 

### login to node1 -- to configure this as Monitor / manager 

```
 C:\Users\hp>
PS C:\Users\hp>
PS C:\Users\hp> ssh  -i .\Downloads\orangekey.cer   ec2-user@18.188.68.153
Last login: Mon Mar 10 09:05:11 2025 from 106.219.69.80
[ec2-user@ip-172-31-4-54 ~]$
[ec2-user@ip-172-31-4-54 ~]$
[ec2-user@ip-172-31-4-54 ~]$
[ec2-user@ip-172-31-4-54 ~]$

```

### changing hostname of machines 

```
ec2-user@ip-172-31-4-54 ~]$ sudo -i
[root@ip-172-31-4-54 ~]# 
[root@ip-172-31-4-54 ~]# whoami
root
[root@ip-172-31-4-54 ~]# hostnamectl set-hostname  ashu-mon 
[root@ip-172-31-4-54 ~]# 
[root@ip-172-31-4-54 ~]# exit
logout
[ec2-user@ip-172-31-4-54 ~]$ sudo -i
[root@ashu-mon ~]# hostname
ashu-mon
[root@ashu-mon ~]# 

```

### history 

```
  9  yum  install net-tools -y 
   10  vim /etc/hosts 
   11  dnf install vim -y
   12  dnf install vim net-tools  -y
   13  ifconfig 
   14  vim /etc/hosts
   15  cat  /etc/hosts
   16  history 
[root@ashu-mon ~]# cat  /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6


172.31.4.54   ashu-mon 
[root@ashu-mon ~]# 
[root@ashu-mon ~]# vim /etc/hosts
[root@ashu-mon ~]# 
[root@ashu-mon ~]# cat  /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6


172.31.4.54   ashu-mon 
172.31.10.59   ashu-node1
172.31.9.143   ashu-node2

```

### changing selinux security to permissive mode 

```
[root@ashu-mon sshd_config.d]# cat  /etc/selinux/config 

# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
# See also:
# https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/9/html/using_selinux/changing-selinux-states-and-modes_using-selinux#changing-selinux-modes-at-boot-time_changing-selinux-states-and-modes
#
# NOTE: Up to RHEL 8 release included, SELINUX=disabled would also
# fully disable SELinux during boot. If you need a system with SELinux
# fully disabled instead of SELinux running with no policy loaded, you
# need to pass selinux=0 to the kernel command line. You can use grubby
# to persistently set the bootloader to boot with selinux=0:
#
#    grubby --update-kernel ALL --args selinux=0
#
# To revert back to SELinux enabled:
#
#    grubby --update-kernel ALL --remove-args selinux
#
SELINUX=permissive

```

### above file changes need a restart so we are doing it 

```
[root@ashu-mon sshd_config.d]# getenforce 
Enforcing
[root@ashu-mon sshd_config.d]# setenforce  0
[root@ashu-mon sshd_config.d]# 
[root@ashu-mon sshd_config.d]# getenforce 
Permissive

```
