Installing Arch Linux in a JiffyBox
===========================

## Set up JiffyBox

- Create a new JiffyBox (asuming Cloud Level 1 with 50GB but this is irrelevant)
- Select Ubuntu 12.04 (64 bit)
- Change the size of Ubuntus harddisk to 10 GB
- Add the following Harddisks that will be used by Arch Linux:
	- arch-boot 200MB ext4 (will be mounted as /boot) 
	- arch-hd: remaining space (24,83 GB) ext4 (will contain /home and /) 
	- note: the formatting with ext4 is not visible from inside Arch Linux (arch-hd needs to be partitioned later)
- Add the created harddisks as third (/dev/xvdc) and fourth (/dev/xvdd) drives to the Standard profile
- Create a new profile with name aProfile
- Select pvgrub64 as Kernel (its the last entry of the drop down list)
- Add the created harddisks as first (/dev/xvda) and second (/dev/xvdb) drives to the new profile aProfile
- Set the root disk to the second drive (/dev/xvdb)

## Installing Arch Linux from inside Ubuntu

- Start JiffyBox with Standard profile
- ssh login with root and defined password
- [opt] Perform an update of Ubuntu via `apt-get update; apt-get upgrade;`
- [opt] Install convenient environment `apt-get install zsh tmux;tmux`

### Partitioning and formating

- Do NOT format arch-boot (/dev/xvda)! The bootloaeder expects an unformatted drive to load the grub menu config
- partition and format arch-hd (/dev/xvdd):
	- use cfdisk or something similar
	- /dev/xvdd1 root partition with 15GB ext4
	- /dev/xvdd2 home partition with remaining space ext4
- mount partitions to install Arch Linux onto them:
	- `mount /dev/xvdd1 /mnt`
	- `mkdir /mnt/home; mount /dev/xvdd2 /mnt/home/`
	- `mkdir /mnt.archboot; mount /dev/xvdc /mnt.archboot`
	- `mkdir /mnt.archboot/boot`
	- `mkdir /mnt/boot/; mount -o bind /mnt.archboot/boot /mnt/boot `
	(This is needed, because the pvgrub bootloader expects to find /boot/grub/menu.lst on the unpartitioned device with the subfolder boot, which would not be there if it would be just mounted in /mnt/boot)
- Do additional needed mounts:
	- `mount -t proc none /mnt/proc`
	- `mount -t sysfs none /mnt/sys`
	- `mount -o bind /dev /mnt/dev`
	- `mount -o bind /dev/pts /mnt/dev/pts`

### Bootstraping Arch Linux
- Grab [Archbootstrap](https://wiki.archlinux.org/index.php/Archbootstrap):
	- `cd /tmp`
	- `wget http://tokland.googlecode.com/svn/trunk/archlinux/arch-bootstrap.sh`
	- `chmod +x arch-bootstrap.sh`
- Install Arch Linux in /mnt via `./arch-bootstrap.sh -a x86_64 -r "ftp://ftp.hosteurope.de/mirror/ftp.archlinux.org" /mnt/`

### Configure Arch Linux in chroot 
- Chroot into the Arch dir: `chroot /mnt bash`
- Init pacman keyring via `pacman-key --init` (**takes some time**)
- `pacman-key --populate archlinux` (sign the keys by answering yes)
- Install base and base-devel packages via `pacman -S base base-devel` (selecting all packages of each group)
- Install ssh via `pacman -S openssh`
- Enable ssh via `systemctl enable sshd.service`
- Configure network (`pacman -S netcfg; systemctl enable netcfg` and so on)
- `mkdir mnt.archboot` (in chroot arch linux)
- Create /etc/fstab for Arch Linux:
		
		/dev/xvdb1 / ext4 defaults 0 0
		/dev/xvda /mnt.archboot ext4 defaults 0 0
		/mnt.archboot/boot /boot none bind defaults 0 0`
		/dev/xvdb2 /home ext4 defaults 0 0
		proc /proc proc defaults 0 0
		dev /dev tmpfs rw 0 0
- Modify /etc/mkinitcpio.conf:
	- set `COMPRESSION="cat"` in `/etc/mkinitcpio.conf`
	- set `MODULES="xen-blkfront xen-fbfront xen-netfront xen-kbdfront"` in /etc/mkinitcpio.conf
- Modify /etc/initab
	- comment in `h0:2345:respawn:/sbin/agetty -8 -s 38400 hvc0 linux`
- Create new linux image via `mkninitcpio -p linux`
- Create menu.lst (grub legacy boot menu) under `/boot/grub/menu.lst`:
		
		timeout 1
		default 0

		title Arch Linux
		root (hd0)
		kernel /boot/vmlinuz-linux root=/dev/xvdb1 ro
		initrd /boot/initramfs-linux.img

- Change root password via `passwd`
- [opt] Customize Arch Linux:
	- change hostname in /etc/hostname to *** (where ***.servers.jiffybox.net)
	- `ln -s /usr/share/zoneinfo/Europe/Berlin /etc/localtime`
	- `echo "LANG=de_DE.UTF-8" > /etc/locale.conf`
	- `echo KEYMAP=de > /etc/vconsole.conf`
	- uncomment de_DE and en_US in /etc/locale.gen

## Use Arch Linux Profile

- Shutdown the box (dont just remove power)
- Change profile to aProfile
- Start the JiffyBox with Arch Linux installed




