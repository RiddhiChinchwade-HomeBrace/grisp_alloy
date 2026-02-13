# GRiSP2 System Notes

This file keeps target-specific, implementation-coupled notes for GRiSP2.

Build/deploy and operations workflows live in wiki:

- [Build and Deploy GRiSP2](https://github.com/grisp/grisp_alloy/wiki/Build-and-Deploy-GRiSP2)
- [GRiSP2 Debugging](https://github.com/grisp/grisp_alloy/wiki/GRiSP2-Debugging)

## Debug boot fallback

Fallback to barebox console during boot by pressing any key when seeing:

```text
Hit m for menu or any to stop autoboot:    X
```

Then set debug bootargs and boot manually:

```sh
global linux.bootargs.extra="loglevel=8 ignore_loglevel initcall_debug panic=-1 -v --run-on-exit /bin/sh --hang-on-exit"
boot
```

## Core dump collection

```sh
dmesg -n 8
echo 0 > /proc/sys/kernel/printk_ratelimit
ulimit -c unlimited
echo '/tmp/core.%e.%p' > /proc/sys/kernel/core_pattern

# trigger crash, then persist core
mount -o remount,rw /mnt/boot
cp /tmp/core.* /mnt/boot
umount /mnt/boot
```
