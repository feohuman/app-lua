# Lua on Unikraft

This application starts a Lua interpreter with Unikraft.
Follow the instructions below to set up, configure, build and run Lua.

To get started immediately, you can use Unikraft's companion command-line companion tool, [`kraft`](https://github.com/unikraft/kraftkit).
Start by running the interactive installer:

```console
curl --proto '=https' --tlsv1.2 -sSf https://get.kraftkit.sh | sudo sh
```

Once installed, clone [this repository](https://github.com/unikraft/app-lua) and run `kraft build`:

```console
git clone https://github.com/unikraft/app-lua lua
cd lua/
kraft build
```

This will guide you through an interactive build process where you can select one of the available targets (architecture/platform combinations).
You will get a list of options out of which to select one:

```text
[?] select target:
  ▸ lua-fc-arm64-initrd (fc/arm64)
    lua-fc-x86_64-initrd (fc/x86_64)
    lua-qemu-arm64-9pfs (qemu/arm64)
    lua-qemu-arm64-initrd (qemu/arm64)
    lua-qemu-x86_64-9pfs (qemu/x86_64)
    lua-qemu-x86_64-initrd (qemu/x86_64)
```

Otherwise, we recommend building for `qemu/x86_64` with an `initrd` support filesystem like so:

```console
kraft build --target lua-qemu-x86_64-initrd -j $(nproc)
```

Once built, you can instantiate the unikernel via:

```console
kraft run --initrd rootfs/ /helloworld.lua
```

## Work with the Basic Build & Run Toolchain (Advanced)

You can set up, configure, build and run the application from grounds up, without using the companion tool `kraft`.

### Quick Setup (aka TLDR)

For a quick setup, run the commands below.
Note that you still need to install the [requirements](#requirements).

For building and running everything for `x86_64`, follow the steps below:

```console
git clone https://github.com/unikraft/app-lua lua
cd lua/
./scripts/setup.sh
wget https://raw.githubusercontent.com/unikraft/app-testing/staging/scripts/generate.py -O scripts/generate.py
chmod a+x scripts/generate.py
./scripts/generate.py
./scripts/build/make-qemu-x86_64.sh
./scripts/run/qemu-x86_64-lua.sh
```

This will configure, build and run the Lua application, resulting in a `Hello world!` message being printed, along with the Unikraft banner.

The same can be done for `AArch64`, by running the commands below:

```console
git clone https://github.com/unikraft/app-lua lua
cd lua/
./scripts/setup.sh
wget https://raw.githubusercontent.com/unikraft/app-testing/staging/scripts/generate.py -O scripts/generate.py
chmod a+x scripts/generate.py
./scripts/generate.py
./scripts/build/make-qemu-arm64.sh
./scripts/run/qemu-arm64-lua.sh
```

Similar to the `x86_64` build, this will result in a `Hello world!` message being printed.
Information about every step is detailed below.

### Requirements

In order to set up, configure, build and run Lua on Unikraft, the following packages are required:

* `build-essential` / `base-devel` / `@development-tools` (the meta-package that includes `make`, `gcc` and other development-related packages)
* `sudo`
* `flex`
* `bison`
* `git`
* `wget`
* `uuid-runtime`
* `qemu-system-x86`
* `qemu-system-arm`
* `qemu-kvm`
* `sgabios`
* `gcc-aarch64-linux-gnu`

GCC >= 8 is required to build Lua on Unikraft.

On Ubuntu/Debian or other `apt`-based distributions, run the following command to install the requirements:

```console
sudo apt install -y --no-install-recommends \
  build-essential \
  sudo \
  gcc-aarch64-linux-gnu \
  libncurses-dev \
  libyaml-dev \
  flex \
  bison \
  git \
  wget \
  uuid-runtime \
  qemu-kvm \
  qemu-system-x86 \
  qemu-system-arm \
  sgabios
```

### Set Up

The following repositories are required for Lua:

* The application repository (this repository): [`app-lua`](https://github.com/unikraft/app-lua)
* The Unikraft core repository: [`unikraft`](https://github.com/unikraft/unikraft)
* Library repositories:
  * The Lua "library" repository: [`lib-lua`](https://github.com/unikraft/lib-lua)
  * The standard C library: [`lib-musl`](https://github.com/unikraft/lib-musl)

Follow the steps below for the setup:

  1. First clone the [`app-lua` repository](https://github.com/unikraft/app-lua) in the `lua/` directory:

     ```console
     git clone https://github.com/unikraft/app-lua lua
     ```

     Enter the `lua/` directory:

     ```console
     cd lua/

     ls -aF
     ```

     You will see the contents of the repository:

     ```text
     Makefile  Makefile.uk  README.md  defconfigs/  kraft.cloud.yaml  kraft.yaml  rootfs/  workdir/
     ```

  1. While inside the `lua/` directory, clone all required repositories by using the `setup.sh` script:

     ```console
     ./scripts/setup.sh
     ```

  1. Use the `tree` command to inspect the contents of the `workdir/` directory:

     ```console
     tree -F -L 2 workdir/
     ```

     The layout of the `workdir/` directory should look something like this:

     ```text
     workdir/
     |-- libs/
     |   |-- lua/
     |   `-- musl/
     `-- unikraft/
         |-- arch/
         |-- Config.uk
         |-- CONTRIBUTING.md
         |-- COPYING.md
         |-- include/
         |-- lib/
         |-- Makefile
         |-- Makefile.uk
         |-- plat/
         |-- README.md
         |-- support/
         `-- version.mk

     10 directories, 7 files
     ```

## Scripted Building and Running

To make it easier to build, run and test different configurations, the repository provides a set of scripts that do everything required.
These are scripts used for building different configurations of the `lua` application and for running these with all the requirements behind the scenes: setting up archives etc.

First of all, grab the [`generate.py` script](https://github.com/unikraft/app-testing/blob/staging/scripts/generate.py) and place it in the `scripts/` directory by running:

```console
wget https://raw.githubusercontent.com/unikraft/app-testing/staging/scripts/generate.py -O scripts/generate.py
chmod a+x scripts/generate.py
```

```console
./scripts/generate.py
```

The scripts (as shell scripts) are now generated in `scripts/build/` and `scripts/run/`:

```text
scripts/
|-- build/
|   |-- kraft-fc-arm64-initrd.sh*
|   |-- kraft-fc-x86_64-initrd.sh*
|   |-- kraft-qemu-arm64-9pfs.sh*
|   |-- kraft-qemu-arm64-initrd.sh*
|   |-- kraft-qemu-x86_64-9pfs.sh*
|   |-- kraft-qemu-x86_64-initrd.sh*
|   |-- make-fc-x86_64-initrd.sh*
|   |-- make-qemu-arm64-9pfs.sh*
|   |-- make-qemu-arm64-initrd.sh*
|   |-- make-qemu-x86_64-9pfs.sh*
|   `-- make-qemu-x86_64-initrd.sh*
|-- generate.py*
|-- run/
|   |-- fc-x86_64-initrd.json
|   |-- fc-x86_64-initrd.sh*
|   |-- kraft-fc-arm64-initrd.sh*
|   |-- kraft-fc-x86_64-initrd.sh*
|   |-- kraft-qemu-arm64-9pfs.sh*
|   |-- kraft-qemu-arm64-initrd.sh*
|   |-- kraft-qemu-x86_64-9pfs.sh*
|   |-- kraft-qemu-x86_64-initrd.sh*
|   |-- qemu-arm64-9pfs.sh*
|   |-- qemu-arm64-initrd.sh*
|   |-- qemu-x86_64-9pfs.sh*
|   `-- qemu-x86_64-initrd.sh*
|-- run.yaml
`-- setup.sh*
```

They are shell scripts, so you can use an editor or a text viewer to check their contents:

```console
cat scripts/run/kraft-fc-x86_64-initrd.sh
```

Now, invoke each script to build and run the application.
A sample build and run set of commands is:

```console
./scripts/build/make-qemu-x86_64-9pfs.sh
./scripts/run/qemu-x86_64-9pfs.sh
```

Another one is:

```console
./scripts/build/make-qemu-arm64-9pfs.sh
./scripts/run/qemu-arm64-9pfs.sh
```

Note that Firecracker only works with initrd (not 9pfs).

## Detailed Steps

### Configure

Configuring, building and running a Unikraft application depends on our choice of platform and architecture.
Currently, supported platforms are QEMU (KVM), Firecracker (KVM), Xen and linuxu.
QEMU (KVM) is known to be working, so we focus on that.

Supported architectures are x86_64 and AArch64.

Builds can use a 9pfs-based filesystem or an initial ramdisk (`initrd`)-based filesystem.

Use the corresponding the configuration files (`defconfigs/*`), according to your choice of platform, architecture and filesystem.

#### QEMU x86_64

Use the `defconfigs/qemu-x86_64-9pfs` configuration file together with `make defconfig` to create the configuration file:

```console
UK_DEFCONFIG=$(pwd)/defconfigs/qemu-x86_64-9pfs make defconfig
```

This results in the creation of the `.config` file:

```console
ls .config
.config
```

The `.config` file will be used in the build step.

#### QEMU AArch64

Use the `defconfigs/qemu-arm64-9pfs` configuration file together with `make defconfig` to create the configuration file:

```console
UK_DEFCONFIG=$(pwd)/defconfigs/qemu-arm64-9pfs make defconfig
```

Similar to the x86_64 configuration, this results in the creation of the `.config` file that will be used in the build step.

### Build

Building uses as input the `.config` file from above, and results in a unikernel image as output.
The unikernel output image, together with intermediary build files, are stored in the `workdir/build/` directory.

#### Clean Up

Before starting a build on a different platform or architecture, you must clean up the build output.
This may also be required in case of a new configuration.

Cleaning up is done with 3 possible commands:

* `make clean`: cleans all actual build output files (binary files, including the unikernel image)
* `make properclean`: removes the entire `build/` directory
* `make distclean`: removes the entire `build/` directory **and** the `.config` file

Typically, you would use `make properclean` to remove all build artifacts, but keep the configuration file.

#### QEMU x86_64

Building for QEMU x86_64 assumes you did the QEMU x86_64 configuration step above.
Build the Unikraft Lua image for QEMU AArch64 by using the command below:

```console
make -j $(nproc)
```

You will see a list of all the files generated by the build system:

```text
[...]
  LD      lua_qemu-x86_64.dbg
/usr/bin/ld: warning: /home/unikraft/lua/build/libkvmplat.o: requires executable stack (because the .note.GNU-stack section is executable)
  UKBI    lua_qemu-x86_64.dbg.bootinfo
  SCSTRIP lua_qemu-x86_64
  GZ      lua_qemu-x86_64.gz
rm /home/unikraft/lua/build/liblua/origin/lua-5.4.4/src/lua.hpp
make[1]: Leaving directory '/home/unikraft/lua/workdir/unikraft'
```

At the end of the build command, the `lua_qemu-x86_64` unikernel image is generated.
This image is to be used in the run step.

#### QEMU AArch64

If you had configured and build a unikernel image for another platform or architecture (such as x86_64) before, then:

1. Do a cleanup step with `make properclean`.

1. Configure for QEMU AAarch64, as shown above.

1. Follow the instructions below to build for QEMU AArch64.

Building for QEMU AArch64 assumes you did the QEMU AArch64 configuration step above.
Build the Unikraft lua image for QEMU AArch64 by using the same command as for x86_64:

```console
make -j $(nproc)
```

Same as in the x86_64 setup, you will see a list of all the files generated by the build system:

```text
[...]
  LD      lua_qemu-arm64.dbg
/usr/lib/gcc-cross/aarch64-linux-gnu/12/../../../../aarch64-linux-gnu/bin/ld: warning: -z relro ignored
  UKBI    lua_qemu-arm64.dbg.bootinfo
  SCSTRIP lua_qemu-arm64
  GZ      lua_qemu-arm64.gz
rm /home/unikraft/lua/build/liblua/origin/lua-5.4.4/src/lua.hpp
make[1]: Leaving directory '/home/unikraft/lua/workdir/unikraft'
```

Similarly to x86_64, at the end of the build command, the `lua_qemu-arm64` unikernel image is generated.
This image is to be used in the run step.

### Run

Run the resulting image using `qemu-system`.

#### QEMU x86_64

To run the QEMU x86_64 build, use `qemu-system-x86_64`:

```console
qemu-system-x86_64 -fsdev local,id=myid,path=$(pwd)/rootfs,security_model=none -device virtio-9p-pci,fsdev=myid,mount_tag=rootfs,disable-modern=on,disable-legacy=off -kernel workdir/build/lua_qemu-x86_64 -nographic -append "-- /helloworld.lua"
```

You will be met by the Unikraft banner, along with the `Hello, world!` message:

```text
Booting from ROM..Powered by
o.   .o       _ _               __ _
Oo   Oo  ___ (_) | __ __  __ _ ' _) :_
oO   oO ' _ `| | |/ /  _)' _` | |_|  _)
oOo oOO| | | | |   (| | | (_) |  _) :_
 OoOoO ._, ._:_:_,\_._,  .__,_:_, \___)
                  Atlas 0.13.1~f7511c8b
hello world from initrd
```

#### QEMU AArch64

To run the AArch64 build, use `qemu-system-aarch64`:

```console
qemu-system-aarch64 -fsdev local,id=myid,path=$(pwd)/rootfs,security_model=none -device virtio-9p-pci,fsdev=myid,mount_tag=rootfs,disable-modern=on,disable-legacy=off -kernel workdir/build/lua_qemu-arm64 -nographic -append "-- /helloworld.lua" -machine virt -cpu max
```

Same as running on x86_64, the application will start:

```text
Booting from ROM..Powered by
o.   .o       _ _               __ _
Oo   Oo  ___ (_) | __ __  __ _ ' _) :_
oO   oO ' _ `| | |/ /  _)' _` | |_|  _)
oOo oOO| | | | |   (| | | (_) |  _) :_
 OoOoO ._, ._:_:_,\_._,  .__,_:_, \___)
                  Atlas 0.13.1~f7511c8b
hello world from initrd
```

### Building and Running with initrd

The examples above use 9pfs as the filesystem interface.
In order two use initrd, you need to first create a CPIO archive that will be passed as the initial ramdisk:

```console
cd rootfs && find -depth -print | tac | bsdcpio -o --format newc > ../rootfs.cpio && cd ..
```

Clean up previous configuration, use the initrd configuration and build the unikernel by using the commands:

```console
make distclean
UK_DEFCONFIG=$(pwd)/defconfigs/qemu-x86_64-initrd make defconfig
make -j $(nproc)
```

Then, run the resulting image with:

```console
qemu-system-x86_64 -kernel workdir/build/lua_qemu-x86_64 -nographic -initrd rootfs.cpio -append "-- /helloworld.lua"
```

The commands for AArch64 are similar:

```console
make distclean
UK_DEFCONFIG=$(pwd)/defconfigs/qemu-arm64-initrd make defconfig
make -j $(nproc)
qemu-system-aarch64 -kernel workdir/build/lua_qemu-arm64 -nographic -initrd rootfs.cpio -append "-- /helloworld.lua" -machine virt -cpu max
```

### Building and Running with Firecracker

[Firecracker](https://firecracker-microvm.github.io/) is a lightweight VMM (*virtual machine manager*) that can be used as more efficient alternative to QEMU.

Configure and build commands are similar to a QEMU-based build with an initrd-based filesystem:

```console
make distclean
UK_DEFCONFIG=$(pwd)/defconfigs/fc-x86_64-initrd make defconfig
make -j $(nproc)
```

For running, a CPIO archive of the filesystem is required to be passed as the initial ramdisk:

```console
cd rootfs && find -depth -print | tac | bsdcpio -o --format newc > ../rootfs.cpio && cd ..
```

To use Firecraker, you need to download a [Firecracker release](https://github.com/firecracker-microvm/firecracker/releases).
You can use the commands below to make the `firecracker-x86_64` executable from release v1.4.0 available globally in the command line:

```console
cd /tmp 
wget https://github.com/firecracker-microvm/firecracker/releases/download/v1.4.0/firecracker-v1.4.0-x86_64.tgz
tar xzf firecracker-v1.4.0-x86_64.tgz 
sudo cp release-v1.4.0-x86_64/firecracker-v1.4.0-x86_64 /usr/local/bin/firecracker-x86_64
```

To run a unikernel image, you need to configure a JSON file.
This is the `lua-fc-x86_64-initrd.json` file.
Pass this file to the `firecracker-x86_64` command to run the Unikernel instance:

```console
rm /tmp/firecracker.socket
firecracker-x86_64 --api-sock /tmp/firecracker.socket --config-file scripts/run/fc-x86_64-initrd-lua.json
```

Same as running with QEMU, the application will start:

```text
Powered by
o.   .o       _ _               __ _
Oo   Oo  ___ (_) | __ __  __ _ ' _) :_
oO   oO ' _ `| | |/ /  _)' _` | |_|  _)
oOo oOO| | | | |   (| | | (_) |  _) :_
 OoOoO ._, ._:_:_,\_._,  .__,_:_, \___)
                  Atlas 0.13.1~f7511c8b
hello world from initrd
```
