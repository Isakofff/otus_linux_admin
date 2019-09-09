# RAID building
RAID-10 auto building using Vagrant and Centos 7 box in Vagrantfile

## How to move legacy system without raid to RAID-1

First of all add additional disk to the system with the same size as the original.
```
$ lsblk
```
> NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
> sda      8:0    0   10G  0 disk 
> └─sda1   8:1    0   10G  0 part /
> sdb      8:16   0   10G  0 disk 
> sr0     11:0    1 1024M  0 rom  





