# Litex Linux on Zynq with Symbiflow toolchain

This guide presents how to build a Linux capable SoC for Xilinx Zynq FPGA using open source toolchain.
The guide discusses all the steps from getting the sources, building them and booting the system.
The guide targets [Zybo Z7 7010 platform](https://store.digilentinc.com/zybo-z7-zynq-7000-arm-fpga-soc-development-board/)

## Prerequisites

### Getting the SymbiFlow sources

```sh
cd ~
git clone https://github.com/SymbiFlow/symbiflow-linux-guide
cd symbiflow-linux-guide
git submodule update --init --recursive
export GUIDEDIR=$(pwd)
```

### RISC-V toolchain

To get the RISC-V toolchain run the following commands:
```sh
mkdir ~/toolchains && cd ~/toolchains
wget https://static.dev.sifive.com/dev-tools/riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-centos6.tar.gz
tar -xvf riscv64-unknown-elf-gcc-8.3.0-2019.08.0-x86_64-linux-centos6.tar.gz
export PATH=$(pwd)/riscv64-unknown-elf-gcc-20171231-x86_64-linux-centos6/bin:$PATH
```

### ARM toolchain:

To get the ARM toolchain run the following commands:
```sh
```sh
mkdir -p ~/toolchains && cd ~/toolchains
wget https://developer.arm.com/-/media/Files/downloads/gnu-rm/9-2019q4/gcc-arm-none-eabi-9-2019-q4-major-x86_64-linux.tar.bz2
tar -xvf gcc-arm-none-eabi-9-2019-q4-major-x86_64-linux.tar.bz2
export PATH=$(pwd)/gcc-arm-none-eabi-9-2019-q4-major/bin:$PATH
```

### Zynq U-Boot bootloader

To get the source code and build the U-Boot bootloader run the following commands:
```sh
cd $GUIDEDIR
git clone https://github.com/Xilinx/u-boot-xlnx.git
cd u-boot-xlnx
export CROSS_COMPILE=arm-none-eabi-
export ARCH=arm
make zynq_zybo_defconfig
make -j$(nproc)
```

## Building the bitstream

Since Symbiflow toolchain is highly experimental and under heavy development, the binary package is not available.
The following guide will build the whole toolchain and then use it for building the Litex Linux SoC project.
This process may take a significant amount of time and memory.

To build the SymbiFlow toolchain and Litex Linux SoC run the following commands:

```sh
cd $GUIDEDIR/symbiflow-arch-defs
make env -j$(nproc)
make all_conda -j$(nproc)
cd build/xc7/tests/ps7/linux_litex
make litex_linux_zybo_bin -j$(nproc)
```

The bitstream routes the Litex serial console to the JA header of the board.
Litex' TX pin is routed to the JA1 pin (N15 pin of the FPGA chip).
Litex' RX pin is routed to the JA2 pin (L14 pin of the FPGA chip).

## Building Linux binaries

To boot Linux on the Litex system the following components are required:

* Linux kernel image
* Devicetree image
* Ramdisk image
* SBI emulator binary

Linux kernel and ramdisk images can be built using the Buildroot project.
To build the kernel and ramdisk images run the following commands:
```sh
cd $GUIDEDIR/linux-on-litex-vexriscv
git clone http://github.com/buildroot/buildroot
cd buildroot
make BR2_EXTERNAL=../linux-on-litex-vexriscv/buildroot/ litex_vexriscv_defconfig
make
```

## Booting the system
Copy all the required files to SD card:
```sh
cp $GUIDEDIR
cp $GUIDEDIR
cp $GUIDEDIR
cp $GUIDEDIR
cp $GUIDEDIR
## Building the bitstream
```
### Prepare SD-Card

Follow the [official SD card preparation guide](https://xilinx-wiki.atlassian.net/wiki/spaces/A/pages/18842385/How+to+format+SD+card+for+SD+boot) to create and format bootable Zynq-7000 SD card.

```

Insert SD card in the board and power it up.
Stop U-Boot from autoboot by pressing any key during countdown.
Run the following commands in U-Boot's terminal:

Load FPGA bistream:
```sh
load mmc 0 0x1000000 top.bit
fpga loadb 0 0x1000000 ${filesize}
```

Load Linux images:
```sh
load mmc 0 0x10000000 Image
load mmc 0 0x10800000 rootfs.cpio
load mmc 0 0x11000000 rv32.dtb
```

Release LiteX system from reset:
```sh
mw.l 0x40000000 1
```

LiteX system should start booting Linux.
The output is visible on Litex' serial console.
