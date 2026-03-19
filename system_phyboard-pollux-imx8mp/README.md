# Kontron AL/BL iMX8M Mini System Notes

This file keeps target-specific source-of-truth details that must stay aligned
with packaging/runtime implementation.

Build/deploy and operations workflows live in wiki:

- [Build and Deploy Kontron AL/BL iMX8M Mini](https://github.com/grisp/grisp_alloy/wiki/Build-and-Deploy-Kontron-ALBL-iMX8MM)
- [Kontron Operations and Updates](https://github.com/grisp/grisp_alloy/wiki/Kontron-Operations-and-Updates)

## Firmware layout and storage mapping

The firmware layout is defined in [`fwup.conf`](fwup.conf) and mirrored in
[`crucible.sh`](crucible.sh) (`GSU_PARTITIONS`). Keep these two in sync.

Partition scheme (GPT):

- p0 reserved, 128 MiB
- p1 boot, FAT32, 64 MiB (`fitImage_A`, `fitImage_B`)
- p2 rootfs A, squashfs, 256 MiB
- p3 rootfs B, squashfs, 256 MiB
- p4 data, f2fs, remaining space

## Bootloader and U-Boot environment

- U-Boot is written at 33 KiB offset (`UBOOT_OFFSET = 66` blocks).
- Redundant U-Boot environments are placed after bootloader area.
- If offsets change, update:
  - `uboot/uboot.config`
  - `rootfs_overlay/etc/fw_env.config`
  - updater HAL mapping in `grisp_updater_kalblimx8mm`

## Runtime device mapping

`erlinit` provides stable aliases:

- `/dev/rootdisk0`
- `/dev/rootdisk0p1` reserved
- `/dev/rootdisk0p2` boot (`/boot`)
- `/dev/rootdisk0p3` system A
- `/dev/rootdisk0p4` system B
- `/dev/rootdisk0p5` data (`/data`)

Use these aliases to stay independent from current SD/eMMC boot origin.

## Data partition initialization

`/data` is prepared at boot by:

- `../system_common/board/grisp-common/rootfs_overlay/sbin/early-init.sh`
