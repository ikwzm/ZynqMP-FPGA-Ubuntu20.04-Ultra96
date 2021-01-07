## Install Ubuntu 20.04(Desktop) to Ultra96-V2

### Downlowd from github

**Note: Downloading the entire repository is time consuming, so download only the branch you need.**

```console
shell$ git clone --depth=1 --branch v2020.2-desktop-rc1 git://github.com/ikwzm/ZynqMP-FPGA-Ubuntu20.04-Ultra96
shell$ cd ZynqMP-FPGA-Ubuntu20.04-Ultra96
shell$ git lfs pull
```

### File Description

 * target/Ultra96-V2
   + boot/
     - boot.bin                                                    : Stage 1 Boot Loader
     - uEnv.txt                                                    : U-Boot environment variables for linux boot
     - image-5.4.0-xlnx-v2020.2-zynqmp-fpga                        : Linux Kernel Image       (use Git LFS)
     - devicetree-5.4.0-xlnx-v2020.2-zynqmp-fpga-ultra96v2.dtb     : Linux Device Tree Blob   
     - devicetree-5.4.0-xlnx-v2020.2-zynqmp-fpga-ultra96v2.dts     : Linux Device Tree Source
 * ubuntu20.04-desktop-rootfs.tgz                                  : Ubuntu 20.04 Desktop Root File System (use Git LFS)
 
### Format SD-Card

[./doc/install/format-disk.md](format-disk.md)

### Write to SD-Card

#### Mount SD-Card

```console
shell# mount /dev/sdc1 /mnt/usb1
shell# mount /dev/sdc2 /mnt/usb2
```
#### Make Boot Partition

```console
shell# cp target/Ultra96-V2/boot/*               /mnt/usb1
```

#### Make RootFS Partition

```console
shell# tar xfz ubuntu20.04-desktop-rootfs.tgz -C /mnt/usb2
```

#### Unmount SD-Card

```console
shell# umount /mnt/usb1
shell# umount /mnt/usb2
```
