# GRiSP Alloy (Linux RT / Buildroot)

Build tool for Linux-based embedded systems.

This repository uses `git lfs` to manage large files and `git-crypt` to
manage PKI secrets. Please install `git lfs` and `git-crypt` before cloning
this repository or checking this branch out.

If a system contains sensitive encrypted files, unlock PKI secrets before
building a production firmware image:

```sh
git-crypt unlock .pki-key
```

## Overview

`grisp_alloy` builds target firmware through a four-stage pipeline:

1. Build toolchain
2. Build SDK (Buildroot rootfs + host tools)
3. Build project artefact(s)
4. Build firmware/update/image artefacts

Main outputs in `artefacts/`:

- toolchain archive
- SDK archive
- project artefacts (`*.tgz`)
- firmware (`*.fw`)
- optional raw disk image(s) (`*.img`)
- optional update package (`*.tar`)

Supported targets in this repository:

- `grisp2`
- `kontron-albl-imx8mm`

## Getting Started

### Prerequisites (Linux)

Install build dependencies:

```sh
sudo apt install build-essential libncurses5-dev \
    pv git bzr cvs mercurial subversion libc6:i386 unzip bc \
    bison flex gperf libncurses5-dev texinfo help2man \
    libssl-dev gawk libtool-bin automake lzip python3 mtools \
    u-boot-tools git-lfs keyutils qemu-user qemu-user-static \
    gcc-x86-64-linux-gnu binutils-x86-64-linux-gnu \
    binutils-x86-64-linux-gnu-dbg cpio
```

### Prerequisites (macOS)

macOS is supported via Vagrant VM execution:

```sh
brew install vagrant qemu
```

The scripts start the VM automatically when needed. Use `-P` to reprovision.

### First Successful Build

```sh
./build-toolchain.sh <TARGET>
./build-sdk.sh <TARGET>
./build-project.sh <TARGET> <PROJECT_DIR>
./build-firmware.sh <TARGET> <ARTEFACT_PREFIX_OR_PATH>
```

Example:

```sh
./build-toolchain.sh grisp2
./build-sdk.sh grisp2
./build-project.sh grisp2 samples/hello_grisp
./build-firmware.sh grisp2 hello_grisp
```

Target-specific end-to-end guides:

- [Build and Deploy GRiSP2](https://github.com/grisp/grisp_alloy/wiki/Build-and-Deploy-GRiSP2)
- [Build and Deploy Kontron AL/BL iMX8M Mini](https://github.com/grisp/grisp_alloy/wiki/Build-and-Deploy-Kontron-ALBL-iMX8MM)

## Build Pipeline

### 1. Build Toolchain

```sh
./build-toolchain.sh [-h] [-d] [-c] [-V] [-P] [-K] <TARGET>
```

### 2. Build SDK

```sh
./build-sdk.sh [OPTIONS] <TARGET>
```

Common options:

- `-d | --debug`: print debug commands
- `-c | --clean`: clean and rebuild from scratch
- `-r | --rebuild`: rerun Buildroot stages for small changes
- `-p | --clean-package <PREFIX>`: rebuild specific Buildroot packages
- `-V | --force-vagrant`: run in Vagrant even on Linux
- `-P | --provision`: reprovision Vagrant VM
- `-K | --keep-vagrant`: keep VM running when script exits

### 3. Build Project Artefact

```sh
./build-project.sh [OPTIONS] <TARGET> <PROJECT_DIR>
```

This packages a project tarball containing:

- `ALLOY-PROJECT` manifest
- `ALLOY-FS-PRIORITIES` (optional)
- `release/` (OTP release)

### 4. Build Firmware

```sh
./build-firmware.sh [OPTIONS] <TARGET> (ARTEFACT_PREFIX | ARTEFACT_PATH [--name NAME])...
```

Useful options:

- `-i | --generate-images`: also generate raw image files
- `-u | --generate-update`: also generate software update package
- `-o | --overlay <DIR>`: merge overlay into rootfs before packaging
- `-S | --security-pack <DIR>`: use security pack
- `-U | --sign-update`: sign update package (requires `--security-pack`)
- `-s | --serial <SERIAL>`: device serial (default `00000000`)
- `-p | --profile <PROFILE>`: security profile (default `default`)
- `-n | --name <NAME>` / `-v | --version <VER>`: override firmware metadata

## Disk Images & Deployment

### Generate firmware + disk images

```sh
./build-firmware.sh -i <TARGET> <ARTEFACT>
```

### Write firmware to device with `fwup`

```sh
fwup -a -d <DEVICE> -i artefacts/<FIRMWARE>.fw -t complete
```

On macOS use `/dev/rdiskX` device names.

### Convert `.fw` to `.img`

```sh
rm -f out.img
touch out.img
fwup -a -d out.img -t complete -i artefacts/<FIRMWARE>.fw
```

### Inspect image partitions

```sh
fdisk -l artefacts/<FIRMWARE>.img
```

## Troubleshooting

Common failures:

- `SDK ... not found in artefacts`: run `./build-sdk.sh <TARGET>` first.
- `No artefact matching prefix ...`: build project first or pass full `.tgz` path.
- `Multiple artefacts found for prefix ...`: use a more specific prefix or explicit tarball path.
- `Missing release erts directory`: ensure the release includes ERTS (`include_erts`).
- `--sign-update/-U requires --security-pack/-S`: pass `-S <SECURITY_PACK_DIR>`.
- Vagrant environment drift after VM/config changes: rerun with `-P`.

VMware provider error (`vagrant-vmware-utility`) can be fixed by restarting the service:

```sh
sudo launchctl stop com.vagrant.vagrant-vmware-utility
sudo launchctl start com.vagrant.vagrant-vmware-utility
```

## Advanced Use Cases

- [Run Multiple VMs with grisp_supervisor](https://github.com/grisp/grisp_alloy/wiki/Run-Multiple-VMs-with-grisp_supervisor)
- [Customize core isolation for critical RT applications](https://github.com/grisp/grisp_alloy/wiki/Customize-core-isolation-for-critical-RT-applications)

## Platform Source-of-Truth Notes

Keep these in-repo READMEs aligned with implementation:

- `system_grisp2/README.md`
- `system_kontron-albl-imx8mm/README.md`
