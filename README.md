# FreeBSD/RISC-V
This is a port of [FreeBSD Operating System](http://www.freebsd.org) to RISC-V instruction set architecture.

[![Build Status](https://ci.freebsd.org/buildStatus/icon?job=FreeBSD-head-riscv64-build)](https://ci.freebsd.org/job/FreeBSD-head-riscv64-build/)


## Quick way

### You can use pre-built images, otherwise proceed to next step.
```
mkdir $HOME/riscv
cd $HOME/riscv
# Install pre-built [OpenSBI](https://github.com/riscv/opensbi/) bootloaders
sudo pkg install opensbi
# Install zstd utility to decompress FreeBSD riscv image
sudo pkg install zstd
# Download FreeBSD riscv Snapshot
fetch https://artifact.ci.freebsd.org/snapshot/head/latest/riscv/riscv64/disk-test.img.zst
zstd -d disk-test.img.zst -o riscv.img
fetch https://artifact.ci.freebsd.org/snapshot/head/latest/riscv/riscv64/kernel.txz 
tar Jxvf kernel.txz --strip-components 3 boot/kernel/kernel
sudo kldload if_tun if_tap
sudo qemu-system-riscv64 -machine virt -m 2048M -smp 2 -nographic -kernel $HOME/riscv/kernel -bios /usr/local/share/opensbi/lp64/generic/firmware/fw_jump.elf -drive file=$HOME/riscv/riscv.img,format=raw,id=hd0 -device virtio-blk-device,drive=hd0 -netdev tap,ifname=tap0,script=no,id=net0 -device virtio-net-device,netdev=net0
# login as root without password
```

## Complete build from scratch
Set the following environment variables:
```
$ setenv MAKEOBJDIRPREFIX /home/${USER}/obj/
$ setenv WITHOUT_FORMAT_EXTENSIONS yes
$ setenv DESTDIR /home/${USER}/riscv-world
```

### Prepare your environment
On FreeBSD 11.0 machine install the required packages:
```
$ sudo pkg install riscv64-xtoolchain-gcc riscv-isa-sim
```

### Build FreeBSD world
```
$ svnlite co http://svn.freebsd.org/base/head freebsd-riscv
$ cd freebsd-riscv
$ make -j4 CROSS_TOOLCHAIN=riscv64-gcc TARGET_ARCH=riscv64 buildworld
```

### Build 32mb rootfs image
```
$ cd freebsd-riscv
$ make TARGET_ARCH=riscv64 -DNO_ROOT -DWITHOUT_TESTS DESTDIR=$DESTDIR installworld
$ make TARGET_ARCH=riscv64 -DNO_ROOT -DWITHOUT_TESTS DESTDIR=$DESTDIR distribution
$ fetch https://raw.githubusercontent.com/bukinr/riscv-tools/master/image/basic.files
$ tools/tools/makeroot/makeroot.sh -s 32m -f basic.files riscv.img $DESTDIR
```

### Prepare your kernel config
Modify sys/riscv/conf/GENERIC. Uncomment the following lines and specify the path to your riscv.img:
```
options 	MD_ROOT
options 	MD_ROOT_SIZE=32768	# 32MB ram disk
makeoptions	MFS_IMAGE=/path/to/riscv.img
options 	ROOTDEVNAME=\"ufs:/dev/md0\"
```

### Build FreeBSD kernel
```
$ cd freebsd-riscv
$ make -j4 CROSS_TOOLCHAIN=riscv64-gcc TARGET_ARCH=riscv64 buildkernel
```

### Build BBL
```
$ git clone https://github.com/freebsd-riscv/riscv-pk
$ cd riscv-pk
$ mkdir build && cd build
$ setenv OBJCOPY riscv64-freebsd-objcopy
$ setenv READELF riscv64-freebsd-readelf
$ setenv RANLIB riscv64-freebsd-ranlib
$ setenv CFLAGS "-nostdlib"
$ ../configure --host=riscv64-unknown-freebsd11.0 --with-payload=path_to_freebsd_kernel
$ gmake bbl
$ unsetenv OBJCOPY
$ unsetenv READELF
$ unsetenv RANLIB
$ unsetenv CFLAGS
```

### Run Spike simulator
```
$ spike /path/to/bbl
```

Additional information is available on [FreeBSD Wiki](http://wiki.freebsd.org/riscv).
