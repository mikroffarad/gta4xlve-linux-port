# Contribution plan

## postmarketOS

Create a device wiki page for `Samsung Galaxy Tab S6 Lite (2022)
(samsung-gta4xlve)`. State the exact SM-P613 Wi-Fi variant, downstream kernel
version, installation method, partition written, recovery path and a hardware
matrix. Mark features that were not exercised as untested.

The first code MR should add only `device-samsung-gta4xlve`. Include the UCM,
deviceinfo, initramfs module list, DTB requirement and the reproducible test
log. The firmware package needs separate maintainer review because it packages
vendor files; publish extraction instructions and checksums, never blobs.

## Mobian

Mobian has a contribution page and welcomes documentation and kernel work, but
its supported-device policy requires a current mainline-based kernel. This
device's current boot path is downstream, so do not present it as an official
Mobian port. Publish the UCM/WirePlumber/Bluetooth notes as an experimental
porting page or issue first, then ask maintainers where a device overlay should
live.

## Kernel

The actual port history is `/home/mikroffarad/src/linux-sm7125:gta4xlve-dev`:
eight commits on tag `sm7125-6.14.7-a52q-a72q`. The archival mbox is included
here. It contains, in dependency order, the device DTS, quaternary/quinary
MI2S clock and frame-layout fixes, the ISL98608 regulator, Himax SPI touchscreen
support, BOE TV104WUM panel support, QCA Bluetooth fixes and device enablement.

Do not send the eight-patch series unchanged to Linux mainline: it is based on
a downstream-oriented SM7125 tree. First rebase each generic change onto the
subsystem maintainer's current tree and split unrelated Bluetooth fixes. The
device DTS comes last, after bindings and generic drivers are accepted.

`/home/mikroffarad/pmos-export/kernel_lineage` is a separate LineageOS worktree.
Its three tracked modifications are local build fixes only: two trace-header
include paths and a TAS256x include path. Rebuild from a clean tree and submit
each only if it remains necessary; include the compiler error and no generated
artifacts.

## Suggested issue/MR evidence

- `uname -a`, exact kernel commit and DTB name.
- `deviceinfo` and partition/flash command, plus a tested recovery command.
- test matrix for display/touch/GPU/Wi-Fi/Bluetooth/audio/charging/suspend,
  naming test method and result.
- known limitations: proprietary firmware, downstream kernel and unsupported
  features.
- no private identifiers, serial numbers, MAC addresses, credentials or blobs.
