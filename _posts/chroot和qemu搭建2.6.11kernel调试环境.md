titile: chroot和qemu搭建2.6.11kernel调试环境
categories: linux
tags: [linux]
---
通过chroot搭建kernel编译环境，通过qemu调试kernel

### 系统环境
ubuntu 16.04 x86_64

### 编译环境
2.6.11的内核没法在16.04的系统上编译通过，通过chroot和debian sarge搭建编译环境

	sudo apt-get install debootstrap schroot

编辑/etc/schroot/chroot.d/sarge_i386.conf文件，替换USERID为实际的用户名。

	[sarge_i386]
	description=Debian 3.1 (Sarge) for i386
	directory=/srv/chroot/sarge_i386
	personality=linux32
	root-users=USERID
	type=directory
	users=USERID

运行

	sudo mkdir -p /srv/chroot/sarge_i386
	sudo debootstrap --variant=buildd --arch i386 sarge /srv/chroot/sarge_i386 http://archive.debian.org/debian/

然后进入chroot安装编译工具

	schroot -c sarge_i386 -u root
	apt-get install sudo build-essential gdb vim bzip2 libncurses-dev
	exit

以后通过下面命令进入chroot环境编译内核

	schroot -c sarge_i386

### 运行环境
通过qemu搭建运行环境

生成文件系统

	qemu-img create -f raw root.img 500M
	mkfs.ext2 -F root.img
	sudo mount -o loop root.img /mnt
	sudo debootstrap --arch i386 sarge /mnt http://archive.debian.org/debian
	sudo cp /etc/resolv.conf /mnt/etc
	sudo cp /etc/hosts /mnt/etc
	
编辑文件/mnt/etc/network/interfaces

	auto lo
	iface lo inet loopback

编译文件/mnt/etc/fstab

	proc /proc proc defaults 0 0
	/dev/hda / ext2 defaults,errors=remount-ro 0 1

unmount

	sudo umount /mnt

### 调试
	qemu-system-i386 -s -S -kernel linux-2.6.11/arch/i386/boot/bzImage -hda root.img -append "root=/dev/hda"

打开gdb

	gdb vmlinux
	target remote localhost:1234

### 遇到的问题
1. make menuconfig没报错，不显示配置界面
在chroot环境中export TERM=xterm
2. 	qemu运行时报tty1 no such file
mount的时候sudo mknod /mnt/dev/tty1 c 4 1