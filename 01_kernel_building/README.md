# Kernel building
Build your own kernel on Centos 7

## Getting Started

```
$ cat /etc/redhat-release
```
> CentOS Linux release 7.6.1810 (Core)
```
$ uname -r
```
> 3.10.0-957.5.1.el7.x86_64
```
$ sudo yum makecache
$ sudo yum install ncurses-devel make gcc bc openssl-devel elfutils-libelf-devel rpm-build wget bison flex ncurses-devel bc perl
$ wget https://cdn.kernel.org/pub/linux/kernel/v3.x/linux-3.16.65.tar.xz
$ tar xf linux-3.16.65.tar.xz
$ cd linux-3.16.65
$ ls /boot/config*
```
> /boot/config-3.10.0-957.5.1.el7.x86_64
```
$ sudo cp -v /boot/config-3.10.0-957.5.1.el7.x86_64 .config
$ make menuconfig
$ diff /boot/config-3.10.0-957.5.1.el7.x86_64 .config | less
```

### The way #1

```
$ time make -j `nproc` && make modules_install && make install
$ grep "menuentry '" /etc/grub2.cfg
```
The IDs are assigned in order the menu entries appear in the /etc/grub2.cfg file starting with 0
```
$ grub2-set-default 0
$ cat /boot/grub2/grubenv |grep saved
```
> saved_entry=0
```
$ sudo shutdown -r now
$ uname -r
```
> 3.16.65

### The way #2

```
$ make rpm-pkg
$ ls ~/rpmbuild/RPMS/x86_64
```
> kernel-3.16.65-1.x86_64.rpm  kernel-devel-3.16.65-1.x86_64.rpm  kernel-headers-3.16.65-1.x86_64.rpm
```
$ sudo rpm -Uvh ~/rpmbuild/RPMS/x86_64/*.rpm
$ sudo shutdown -r now
$ uname -r
```
> 3.16.65

## Vagrant with compiled kernel

```
git clone https://github.com/Isakofff/otus_linux_admin.git
cd otus_linux_admin
vagrant up
vagrant ssh
uname -r
```
> 3.16.65


