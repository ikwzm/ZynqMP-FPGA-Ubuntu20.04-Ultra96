ZynqMP-FPGA-Ubuntu20.04-Ultra96
====================================================================================

Overview
------------------------------------------------------------------------------------

### Introduction

This Repository provides a Linux Boot Image(U-boot, Kernel, Ubuntu 20.04 Desktop) for Ultra96/Ultra96-V2.

### Note

This Repository is currentrly under development on the 'develop' branch.

### Features

* Hardware
  + Ultra96    : Xilinx Zynq UltraScale+ MPSoC development board based on the Linaro 96Boards specification. 
  + Ultra96-V2 : updates and refreshes the Ultra96 product that was released in 2018.
* Boot Loader
  + FSBL(First Stage Boot Loader for ZynqMP)
  + PMU Firmware(Platform Management Unit Firmware)
  + BL31(ARM Trusted Firmware Boot Loader stage 3-1)
  + U-Boot xilinx-v2019.2 (customized)
* Linux Kernel Version v5.4.0
  + [linux-xlnx](https://github.com/Xilinx/linux-xlnx) tag=xilinx-v2020.2
  + Enable Device Tree Overlay with Configuration File System
  + Enable FPGA Manager
  + Enable FPGA Bridge
  + Enable FPGA Reagion
  + Enable ATWILC3000 Linux Driver for Ultra96-V2
  + Enable CIFS (Common Internet File System)
  + Enable Lima (Open-source reverse-engineered driver for Mali-4xx GPUs)
  + Enable Xilinx APF Accelerator driver
  + Enable Xilinx APF DMA engines support
* Ubuntu20.04(focal) Desktop Root File System
  + Installed build-essential
  + Installed ubuntu-desktop
  + Installed lightdm
