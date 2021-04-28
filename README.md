# UbuntuDesktopFocalM7

Trying to run Ubuntu Desktop 19.04 on the HTC One M7

## Status

Not yet working. Using the PostmarketOS boot.img for a debug-ramdisk in order to use telnet. Almost had some success, but the device doesn't seem to have a framebuffer device. If it does, then I cannot see the device. The other thing used was debootstrap to create a armhf rootfs for Ubuntu Focal.

## Commands to make the rootfs and possibly install (I will leave my debootstrapped rootfs in the releases tab, but it might not work unless the fbdev works)

sudo -i # type in your root password after

apt install -y debootstrap qemu-user-static

cd /mnt && mkdir ubuntu

debootstrap  --arch=armhf --foreign focal /mnt/ubuntu # wait some time for the packages to download and set up

### These next steps are recommended to be done on the device within recovery, but I don't really like having to tar, move, extract, modify, then tar, move, and extract a rootfs from a device multiple times as it can be slow and since we are in recovery, it will not have internet access, so no downloading and installing necessesities. This will be done on a PC using qemu-arm-static and chroot

cp /usr/bin/qemu-arm-static /mnt/ubuntu/usr/bin/

chroot /mnt/ubuntu /usr/bin/qemu-arm-static /bin/sh -i

/debootstrap/debootstrap --second-stage # wait awhile for it to set up and install the base system

passwd # type in preferred password

passwd root # type in preferred password again

apt install -y nano

### To avoid some issues with telnet/ssh, we can change the hostname to any preferred name

nano /etc/hostname # change name in here to something that is not your PC's name

apt install locales

dpkg-reconfigure locales # choose region, in my case, I chose en_US.UTF-8 (choose this option for both prompts)

### Configuring ethernet with DHCP

auto lo eth0

allow-hotplug eth0

iface lo inet loopback

iface eth0 inet dhcp

### Fixing sources.list

nano sources.list 

> deb http://ports.ubuntu.com/ focal main universe
> 
> deb-src http://ports.ubuntu.com/ focal main universe
> 
> deb http://ports.ubuntu.com/ focal-security main universe
> 
> deb-src http://ports.ubuntu.com/ focal-security main universe
> 
> deb http://ports.ubuntu.com/ focal-updates main universe
> 
> deb-src http://ports.ubuntu.com/ focal-updates main universe

### Press Ctrl + X, then Y to save, then X to exit.

## Install a desktop environment. XFCE4 was not working for me, so try something else. For the sake of this small guide, I will put xfce4 because that is what I was trying.

apt install xfce4 # or gdm3, xdm, etc. Whatever works for you.

## I skipped some of these next steps because I am wanting to install full Ubuntu without Android installed.

### To get Openssh-server, run

apt install -y openssh-server

### To login to ssh as root

nano /etc/ssh/sshd_config

### Uncomment this line:

#PermitRootLogin prohibit-password

### Uncomment the PermitRootLogin line, and set it to yes.

PermitRootLogin yes

### Press Ctrl+X, Y, then enter. Then add your PC's SSH Keys to /root/.ssh/authorized_keys.

## Clean up.

apt clean

rm /usr/bin/qemu-arm-static

exit

## Now we are done with setting up our rootfs. Let's .tar.bz2 the rootfs, then push it to the device to test if it has worked.

tar -cjpf ~/ubuntu_rootfs.tar.bz2 # Wait a moment as it is compressing everything. 

## This will produce a archive of around ~475MB located in the folder that appears as soon as you open File Explorer.

### Now let's push and extract.

adb push ~/ubuntu_rootfs.tar.bz2 /data

adb shell

cd data && tar xvf ubuntu_rootfs.tar.bz2 . # Wait for some more as it is a big rootfs.

### Once extracted, make sure you have pmbootstrap to get the telnet boot.img for the M7. Installing pmbootstrap is as easy as:

### Make sure you have Python 3.6+, openssl, pip3 and git.

git clone https://gitlab.com/postmarketOS/pmbootstrap.git

cd pmbootstrap && python3 setup.py install

### Making the telnet boot.img is simple.

pmbootstrap init # choose everything as default

pmbootstrap initfs hook_add debug-shell

pmbootstrap export

### Done! Now copy it to the home folder.

cp /tmp/postmarketOS-export/boot.img-htc-m7 ~/boot.img

### And flash..

fastboot flash boot ~/boot.img

fastboot reboot

## From here on out you essentially have Ubuntu on your phone, whether or not it works is a matter of if the display manager is compatible and if the kernel is working (I guess.) Let's telnet and mount some partitions.

telnet 172.16.42.1

mount /dev/mmcblk0p39 /root

mount -o bind /dev /root/dev # this fails for me, idk why tho.

mount -o bind /dev/pts /root/dev/pts

mount -o bind /sys /root/sys

mount -o bind /proc /root/proc

### After this we can chroot and start the display manager. startx is for xfce4 as far as I know, I don't know the others, hence why I cannot try to start xdm, gdm3, or gnome on my device.

chroot /root /bin/bash -i

startx

### DONE! If it worked, you should see somthing on screen (I think?)

## If it worked, give yourself a pat on the back. You accomplished something I never though I could do, but I will do it one day because when I start something, I usually continue unless I don't think I can figure it out anymore. Please let me know if you guys got it working, and let me know how please. Thanks for trying, and if it didn't work, keep trying with different things. It might work eventually.

## Thanks to Aljoshua Hell on Telegram for helping me with my questions. Although I did not get it working in the end (yet), I will try to continue to make it work. And thanks to the PostmarketOS team for providing the boot.img and debug ramdisk for telnet.

## Device Image

<img src="https://m.media-amazon.com/images/I/51XwWuTTQZL._AC_.jpg" width="500" height="421" />
