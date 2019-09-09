# RAID building
RAID-10 auto building using Vagrant and Centos 7 box in Vagrantfile

## How to move legacy system without raid to RAID-1

First of all add additional disk to the system with the same size as the original.
```
$ lsblk
    # NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
    # sda      8:0    0   10G  0 disk 
    # └─sda1   8:1    0   10G  0 part /
    # sdb      8:16   0   10G  0 disk 
    # sr0     11:0    1 1024M  0 rom  

# install everything which we will need
yum install mdadm rsync

# first we need to prepare /dev/sdb, to copy everything from /dev/sda and only after that we will able to add /dev/sda to our RAID

# so let's start from partition table copy to new disk
sfdisk -d /dev/sda | sfdisk /dev/sdb

# now it should look identical
fdisk -l 

# create raid on new disk, at the moment only with ONE disk
mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 missing /dev/sdb1

# check
cat /proc/mdstat
mdadm -D /dev/md0

# create FS
mkfs.ext4 /dev/md0

# mount raid device
mount /dev/md0 /mnt

# copy everything; but check before
cd /mnt
rsync -avxu -n --delete / /mnt/
rsync -avxu --delete / /mnt/

# mount everything what we need and make chroot
mount --rbind /dev/ /mnt/dev && mount --rbind /proc /mnt/proc && mount --rbind /sys /mnt/sys && mount --rbind /run /mnt/run
chroot /mnt

# get uuid of raid device and replace it in /etc/fstab
blkid | grep md0
    # /dev/md0: UUID="344079c8-8e32-4473-9327-2fa6ec993058" TYPE="ext4" 
vi /etc/fstab
    UUID=344079c8-8e32-4473-9327-2fa6ec993058 /                       ext4    defaults        1 1

# create mdadm config 
mdadm --detail --scan > /etc/mdadm.conf 

# recreate initramfs
ll /boot
mv /boot/initramfs-3.10.0-957.el7.x86_64.img /boot/initramfs-3.10.0-957.el7.x86_64.img.bak
dracut /boot/initramfs-$(uname -r).img $(uname -r)

# add to kernel rd.auto=1 and disable selinux to prevent problems on login stage after reboot
vi /etc/default/grub
    GRUB_CMDLINE_LINUX="rhgb quiet rd.auto=1 selinux=0"

# recreate grub config and add it to /dev/sdb
grub2-mkconfig -o /boot/grub2/grub.cfg && grub2-install /dev/sdb

# check
cat /boot/grub2/grub.cfg

# just for visibility
touch /thisIsTheRaidDrive

# now we can restart
shutdown -r now

# choose second disk in bios boot menu; in Virtualbox press Fn+F12

# check that now we are using /dev/md0 instead /dev/sda
df -h
    # Файловая система Размер Использовано  Дост Использовано% Cмонтировано в
    # /dev/md0           9,8G         1,3G  8,0G           14% /
    # devtmpfs           990M            0  990M            0% /dev
    # tmpfs             1000M            0 1000M            0% /dev/shm
    # tmpfs             1000M         8,6M  991M            1% /run
    # tmpfs             1000M            0 1000M            0% /sys/fs/cgroup
    # tmpfs              200M            0  200M            0% /run/user/0

# add 'Linux raid autodetect' to /dev/sda
fdisk /dev/sda
    t
    fd
    w

# add /dev/sda to our raid
mdadm --manage /dev/md0 --add /dev/sda1

# wait recovery process
watch -n 1 "cat /proc/mdstat"

# check
mdadm -D /dev/md0
lsblk
    # NAME    MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
    # sda       8:0    0   10G  0 disk  
    # └─sda1    8:1    0   10G  0 part  
    #   └─md0   9:0    0   10G  0 raid1 /
    # sdb       8:16   0   10G  0 disk  
    # └─sdb1    8:17   0   10G  0 part  
    #   └─md0   9:0    0   10G  0 raid1 /
    # sr0      11:0    1 1024M  0 rom 
```






