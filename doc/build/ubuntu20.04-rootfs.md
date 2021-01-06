## Build Ubuntu 20.04 Desktop RootFS

### Setup parameters 

```console
shell$ apt-get install qemu-user-static debootstrap binfmt-support
shell$ export targetdir=ubuntu20.04-rootfs
shell$ export distro=focal
```

### Prepare Debian Packages

#### Download ZynqMP-FPGA-Linux

```console
shell$ git clone --depth 1 -b v2020.2.1-rc1 git://github.com/ikwzm/ZynqMP-FPGA-Linux
shell$ cd ZynqMP-FPGA-Linux
shell$ git lfs pull
shell$ cd ..
```

### Build the root file system in $targetdir(=ubuntu20.04-rootfs)

```console
shell$ mkdir                                               $PWD/$targetdir
shell$ sudo debootstrap --arch=arm64 --foreign $distro     $PWD/$targetdir
shell$ sudo cp /usr/bin/qemu-aarch64-static                $PWD/$targetdir/usr/bin
shell$ sudo cp /etc/resolv.conf                            $PWD/$targetdir/etc
shell$ sudo cp scripts/build-ubuntu20.04-server-rootfs.sh  $PWD/$targetdir
shell$ sudo cp scripts/build-ubuntu20.04-desktop-rootfs.sh $PWD/$targetdir
shell$ sudo cp ZynqMP-FPGA-Linux/linux-*.deb               $PWD/$targetdir
shell$ sudo cp files/*.deb                                 $PWD/$targetdir
shell$ sudo cp files/xorg.conf                             $PWD/$targetdir
shell$ sudo mount -vt proc proc                            $PWD/$targetdir/proc
shell$ sudo mount -vt devpts devpts -o gid=5,mode=620      $PWD/$targetdir/dev/pts
````

### Build ubuntu20.04-rootfs with QEMU

#### Change Root to ubuntu20.04

```console
shell$ sudo chroot $PWD/$targetdir
```

There are two ways

1. run build-ubuntu20.04-server-rootfs.sh then run build-ubuntu20.04-desktop-rootfs.sh (easy)
2. run this chapter step-by-step (annoying)

#### Setup APT

````console
ubuntu20.04-rootfs# distro=focal
ubuntu20.04-rootfs# export LANG=C
ubuntu20.04-rootfs# /debootstrap/debootstrap --second-stage
````

```console
ubuntu20.04-rootfs# cat <<EOT > /etc/apt/sources.list
deb http://ports.ubuntu.com/ubuntu-ports focal main restricted universe multiverse
deb-src http://ports.ubuntu.com/ubuntu-ports focal main restricted universe multiverse
deb http://ports.ubuntu.com/ubuntu-ports focal-updates main restricted universe multiverse
deb-src http://ports.ubuntu.com/ubuntu-ports focal-updates main restricted universe multiverse
deb http://ports.ubuntu.com/ubuntu-ports focal-security main restricted universe multiverse
deb-src http://ports.ubuntu.com/ubuntu-ports focal-security main restricted universe multiverse
EOT
```

```console
ubuntu20.04-rootfs# cat <<EOT > /etc/apt/apt.conf.d/71-no-recommends
APT::Install-Recommends "0";
APT::Install-Suggests   "0";
EOT
```

```console
ubuntu20.04-rootfs# apt-get -y update
ubuntu20.04-rootfs# apt-get -y upgrade
```

#### Install Core applications

```console
ubuntu20.04-rootfs# apt-get install -y locales dialog
ubuntu20.04-rootfs# dpkg-reconfigure locales
ubuntu20.04-rootfs# apt-get install -y net-tools openssh-server ntpdate resolvconf sudo less hwinfo ntp tcsh zsh file
```

#### Setup hostname

```console
ubuntu20.04-rootfs# echo ubuntu-fpga > /etc/hostname
```

#### Setup root password

```console
ubuntu20.04-rootfs# passwd
```

This time, we set the "admin" at the root' password.

To be able to login as root from Zynq serial port.

```console
ubuntu20.04-rootfs# cat <<EOT >> /etc/securetty
# Seral Port for Xilinx Zynq
ttyPS0
EOT
```

#### Add a new guest user

```console
ubuntu20.04-rootfs# adduser fpga
```

This time, we set the "fpga" at the fpga'password.

```console
ubuntu20.04-rootfs# echo "fpga ALL=(ALL:ALL) ALL" > /etc/sudoers.d/fpga
```

#### Setup sshd config

```console
ubuntu20.04-rootfs# sed -i -e 's/#PasswordAuthentication/PasswordAuthentication/g' /etc/ssh/sshd_config
```

#### Setup Time Zone

```console
ubuntu20.04-rootfs# dpkg-reconfigure tzdata
```

or if noninteractive set to Asia/Tokyo

```console
ubuntu20.04-rootfs# echo "Asia/Tokyo" > /etc/timezone
ubuntu20.04-rootfs# dpkg-reconfigure -f noninteractive tzdata
```


#### Setup fstab

```console
ubuntu20.04-rootfs# cat <<EOT > /etc/fstab
none            /config         configfs        defaults        0       0
/dev/mmcblk0p1  /mnt/boot       auto            defaults        0       0
EOT
````

#### Setup /lib/firmware

```console
ubuntu20.04-rootfs# mkdir /lib/firmware
ubuntu20.04-rootfs# mkdir /lib/firmware/ti-connectivity
ubuntu20.04-rootfs# mkdir /lib/firmware/mchp
```

#### Install Development applications

```console
ubuntu20.04-rootfs# apt-get install -y build-essential
ubuntu20.04-rootfs# apt-get install -y pkg-config
ubuntu20.04-rootfs# apt-get install -y git
ubuntu20.04-rootfs# apt-get install -y kmod
ubuntu20.04-rootfs# apt-get install -y flex bison
ubuntu20.04-rootfs# apt-get install -y u-boot-tools device-tree-compiler
ubuntu20.04-rootfs# apt-get install -y libssl-dev
ubuntu20.04-rootfs# apt-get install -y socat
ubuntu20.04-rootfs# apt-get install -y ruby rake ruby-msgpack ruby-serialport
ubuntu20.04-rootfs# apt-get install -y python3 python3-dev python3-setuptools python3-wheel python3-pip python3-numpy
ubuntu20.04-rootfs# pip3 install msgpack-rpc-python
```

#### Install Wireless tools and firmware

```console
ubuntu20.04-rootfs# apt-get install -y wireless-tools
ubuntu20.04-rootfs# apt-get install -y wpasupplicant
```

```console
ubuntu20.04-rootfs# git clone git://git.ti.com/wilink8-wlan/wl18xx_fw.git
ubuntu20.04-rootfs# cp wl18xx_fw/wl18xx-fw-4.bin /lib/firmware/ti-connectivity
ubuntu20.04-rootfs# rm -rf wl18xx_fw/
```

```console
ubuntu20.04-rootfs# git clone git://git.ti.com/wilink8-bt/ti-bt-firmware
ubuntu20.04-rootfs# cp ti-bt-firmware/TIInit_11.8.32.bts /lib/firmware/ti-connectivity
ubuntu20.04-rootfs# rm -rf ti-bt-firmware
```

```console
ubuntu20.04-rootfs# git clone git://github.com/linux4wilc/firmware  linux4wilc-firmware  
ubuntu20.04-rootfs# cp linux4wilc-firmware/*.bin /lib/firmware/mchp
ubuntu20.04-rootfs# rm -rf linux4wilc-firmware  
```

#### Install Other applications

```console
ubuntu20.04-rootfs# apt-get install -y samba
ubuntu20.04-rootfs# apt-get install -y avahi-daemon
```

#### Install haveged for Linux Kernel 4.19

```console
ubuntu20.04-rootfs# apt-get install -y haveged
```

#### Install network-manager for Ubuntu20.04

```console
ubuntu20.04-rootfs# apt-get install -y network-manager
```

#### Move Debian Package to /home/fpga/debian

```console
ubuntu20.04-rootfs# mkdir              home/fpga/debian
ubuntu20.04-rootfs# mv *.deb           home/fpga/debian
ubuntu20.04-rootfs# mv xorg.conf       home/fpga/debian
ubuntu20.04-rootfs# chown fpga.fpga -R home/fpga/debian
```

#### Install Linux Image and Header Debian Packages

```console
ubuntu20.04-rootfs# dpkg -i home/fpga/debian/linux-image-5.4.0-xlnx-v2020.2-zynqmp-fpga_5.4.0-xlnx-v2020.2-zynqmp-fpga-1_arm64.deb
ubuntu20.04-rootfs# dpkg -i home/fpga/debian/linux-headers-5.4.0-xlnx-v2020.2-zynqmp-fpga_5.4.0-xlnx-v2020.2-zynqmp-fpga-1_arm64.deb
```

#### Create Debian Package List

```console
ubuntu20.04-rootfs# dpkg -l > dpkg-server-list.txt
```

#### Install Xorg HWE (Option)

```console
ubuntu20.04-rootfs# apt-get install -y xserver-xorg-core-hwe-20.04 xserver-xorg-input-all-hwe-20.04 xserver-xorg-legacy-hwe-20.04
ubuntu20.04-rootfs# apt-get install -y xorg
```

#### Install Ubuntu Desktop

```console
ubuntu20.04-rootfs# apt-get install -y ubuntu-desktop
```

#### Install ZynqMP-FPGA-Xserver

```console
ubuntu20.04-rootfs# dpkg -i home/fpga/debian/xserver-xorg-video-armsoc-xilinx_1.4-ubuntu20-2_arm64.deb
ubuntu20.04-rootfs# cp      home/fpga/debian/xorg.conf /etc/X11
```

#### Change Display Manager gdm -> lightdm

```console
ubuntu20.04-rootfs# apt install -y libpam-gnome-keyring libpam-kwallet5
ubuntu20.04-rootfs# apt install -y lightdm lightdm-settings slick-greeter
```

#### Disable Sleep/Suspend Mode

```console
ubuntu20.04-rootfs# systemctl mask sleep.target suspend.target hybrid-sleep.target
```

#### Work around Upower Service 

```console
ubuntu20.04-rootfs# sed -i -e 's/PrivateUsers=yes/#PrivateUsers=yes/g'             /usr/lib/systemd/system/upower.service
ubuntu20.04-rootfs# sed -i -e 's/RestrictNamespaces=yes/#RestrictNamespaces=yes/g' /usr/lib/systemd/system/upower.service
```

#### Work around Gtk application crashes

 * https://gitlab.gnome.org/GNOME/gdk-pixbuf/-/issues/159

```console
ubuntu20.04-rootfs# update-mime-database /usr/share/mime
ubuntu20.04-rootfs# /usr/lib/aarch64-linux-gnu/gdk-pixbuf-2.0/gdk-pixbuf-query-loaders --update-cache
```

#### Clean Cache

```console
ubuntu20.04-rootfs# apt-get clean
```

#### Create Debian Package List

```console
ubuntu20.04-rootfs# dpkg -l > dpkg-desktop-list.txt
```

#### Finish

```console
ubuntu20.04-rootfs# exit
shell$ sudo rm -f  $PWD/$targetdir/usr/bin/qemu-aarch64-static
shell$ sudo rm -f  $PWD/$targetdir/build-ubuntu20.04-server-rootfs.sh
shell$ sudo rm -f  $PWD/$targetdir/build-ubuntu20.04-desktop-rootfs.sh
shell$ sudo mv     $PWD/$targetdir/dpkg-desktop-list.txt files/ubuntu20.04-dpkg-list.txt
shell$ sudo umount $PWD/$targetdir/proc
shell$ sudo umount $PWD/$targetdir/dev/pts
```

### Build ubuntu20.04-desktop-rootfs.tgz

```console
shell$ cd $PWD/$targetdir
shell$ sudo tar cfz ../ubuntu20.04-desktop-rootfs.tgz *
```
