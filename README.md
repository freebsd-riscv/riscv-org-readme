# FreeBSD/RISC-V
This is a port of the [FreeBSD Operating System](http://www.freebsd.org) to the RISC-V instruction set architecture.

[![Build Status](https://ci.freebsd.org/buildStatus/icon?job=FreeBSD-head-riscv64-build)](https://ci.freebsd.org/job/FreeBSD-head-riscv64-build/)


## Quick Start

**You can use pre-built images, otherwise proceed to the next step.**

### Set up your path

```
mkdir $HOME/riscv
cd $HOME/riscv
```

### Install pre-built [OpenSBI](https://github.com/riscv/opensbi/) bootloaders

```
sudo pkg install opensbi
````

### Install `zstd` utility to decompress FreeBSD/RISC-V image

This step can be skipped on FreeBSD >= 12.

```
sudo pkg install zstd
```

### Install QEMU emulator

```
sudo pkg install qemu-devel
```

### Download FreeBSD/RISC-V snapshot and kernel

```
fetch https://artifact.ci.freebsd.org/snapshot/head/latest/riscv/riscv64/disk-test.img.zst
zstd -d disk-test.img.zst -o riscv.img
fetch https://artifact.ci.freebsd.org/snapshot/head/latest/riscv/riscv64/kernel.txz 
tar Jxvf kernel.txz --strip-components 3 boot/kernel/kernel
```

### Configure tap(4) 

Load (tun)tap module(s) if not already loaded

If FreeBSD >= 13:

```
sudo kldload if_tuntap
```

Else:

```
sudo kldload if_tun if_tap
```

Configure the network with tap(4):

```
ifconfig tap0 create up
ifconfig bridge0 create up
ifconfig bridge0 addm em0 addm tap0 # em0 might depends of your ethernet devices
```

### Run FreeBSD/RISC-V in QEMU

```
sudo qemu-system-riscv64 -machine virt -m 2048M -smp 2 -nographic -kernel $HOME/riscv/kernel -bios /usr/local/share/opensbi/lp64/generic/firmware/fw_jump.elf -drive file=$HOME/riscv/riscv.img,format=raw,id=hd0 -device virtio-blk-device,drive=hd0 -netdev tap,ifname=tap0,script=no,id=net0 -device virtio-net-device,netdev=net0
# login as root without password
```

## Complete build from scratch
Set the following environment variables:

with (t)csh shell:

```
setenv MAKEOBJDIRPREFIX ${HOME}/riscv/obj/
setenv DESTDIR ${HOME}/riscv/riscv-world
```

with bash/zsh/etc. shell:

```
export MAKEOBJDIRPREFIX=${HOME}/riscv/obj/
export DESTDIR=${HOME}/riscv/riscv-world
```

### Build FreeBSD/RISC-V

Get source from svn:
```
svnlite co https://svn.freebsd.org/base/head ${HOME}/riscv/freebsd-riscv
```

Build FreeBSD/RISC-V
```
cd ${HOME}/riscv/freebsd-riscv
make -j`sysctl -n kern.smp.cpus` TARGET_ARCH=riscv64 buildworld
make -j`sysctl -n kern.smp.cpus` TARGET_ARCH=riscv64 buildkernel
```

Install FreeBSD/RISC-V into $DESTDIR
```
make TARGET_ARCH=riscv64 -DNO_ROOT -DWITHOUT_TESTS DESTDIR=$DESTDIR installworld
make TARGET_ARCH=riscv64 -DNO_ROOT -DWITHOUT_TESTS DESTDIR=$DESTDIR distribution 
make TARGET_ARCH=riscv64 -DNO_ROOT -DWITHOUT_TESTS DESTDIR=$DESTDIR installkernel
```

### Create the disk image
```
cd $DESTDIR
sed -E 's/time=[0-9\.]+$//' METALOG > METALOG.new
mv METALOG.new METALOG
echo 'hostname="qemu"' > etc/rc.conf
echo "/dev/vtbd0        /       ufs     rw      1       1" > etc/fstab
echo "./etc/fstab type=file uname=root gname=wheel mode=0644" >> METALOG
echo "./etc/rc.conf type=file uname=root gname=wheel mode=0644" >> METALOG
makefs -Z -D -f 1000000 -o version=2 -s 10g $HOME/riscv/riscv.img METALOG
```

### Install pre-built [OpenSBI](https://github.com/riscv/opensbi/) bootloaders

```
sudo pkg install opensbi
````

### Install QEMU emulator

```
sudo pkg install qemu-devel
```

### Run FreeBSD/RISC-V in QEMU

```
sudo qemu-system-riscv64 -machine virt -m 2048M -smp 2 -nographic -kernel $HOME/riscv/kernel -bios /usr/local/share/opensbi/lp64/generic/firmware/fw_jump.elf -drive file=$HOME/riscv/riscv.img,format=raw,id=hd0 -device virtio-blk-device,drive=hd0 -netdev tap,ifname=tap0,script=no,id=net0 -device virtio-net-device,netdev=net0
# login as root without password
```

Additional information is available on [FreeBSD Wiki](https://wiki.freebsd.org/riscv).
