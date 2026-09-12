# Samsung Galaxy Tab S6 Lite (2022) Linux port archive

This is the reproducible source portion of a postmarketOS/Mobian port for the
Wi-Fi Samsung Galaxy Tab S6 Lite (2022), model SM-P613, codename `gta4xlve`.
It deliberately excludes partition images, Android firmware, proprietary DSP
blobs, network connection files, Bluetooth pairing keys and user data.

## Contents

- `pmaports/` contains the original `device-samsung-gta4xlve` and
  `firmware-samsung-gta4xlve` package recipes. The firmware recipe needs a
  locally extracted `gta4xlve-firmware.tar.gz`; do not commit or redistribute
  that vendor archive. See [the firmware reproduction notes](docs/firmware.md).
- `config/mobian/Samsung/gta4xlve/` contains the ALSA UCM profile that makes
  the internal audio card usable.
- `config/mobian/wireplumber/60-alsa-s16.conf` works around silent speakers
  when the sink negotiates `s24-32le`.
- `config/mobian/modprobe.d/hci_uart.conf` sets `ibs_host_sleep=0`, preventing
  the board Bluetooth controller from cycling on and off.
- `docs/performance-and-storage.md` records the verified UFS migration and
  Wine/Box64/WineASIO investigation. It is operational evidence, not a
  distribution-supported FL Studio setup.
- `docs/gta4xlve-kernel-series.mbox` is an archival export of the eight commits
  on `gta4xlve-dev`, based on `sm7125-6.14.7-a52q-a72q`. It is a review aid;
  it must be rebased and split before submission to Linux upstream.
- `docs/gta4xlve-dev.bundle` preserves the same branch as Git objects. In a
  clone containing base commit `11590c16c`, restore it with
  `git fetch docs/gta4xlve-dev.bundle gta4xlve-dev:gta4xlve-dev`.

## Hardware status verified in this port

The downstream Linux 6.14.7-sm7125 booted from Samsung's boot partition with
the `qcom/sm7125-samsung-gta4xlve` DTB. Display, touchscreen, GPU (Freedreno
FD618), Wi-Fi, Bluetooth A2DP, speakers and UFS-backed rootfs were used.

The platform still depends on Samsung/Qualcomm downstream firmware and a
downstream kernel. It is therefore a community port, not a mainline-support
claim. Camera, suspend reliability, modem and other untested hardware must be
recorded as untested rather than assumed working.

## Upstreaming order

1. Submit `device-samsung-gta4xlve` to `postmarketOS/pmaports` with a device
   wiki page and a precise test matrix.
2. Discuss the firmware recipe with pmaports maintainers before submitting it:
   the recipe is useful, but the proprietary archive must stay out of Git.
3. Turn the Mobian configuration into a separate device-overlay or packaging
   proposal after asking in `#mobian-ports:matrix.org`; Mobian only accepts
   mainline-based kernels for supported devices.
4. Send kernel fixes only to the tree that needs them, with a minimal build
   reproduction. Do not mix generated build files into a source patch.

The complete working kernel history is in
`/home/mikroffarad/src/linux-sm7125`, branch `gta4xlve-dev`, with upstream
remote `https://github.com/sm7125-mainline/linux.git`. It contains eight
commits from `9d5ca0b30` through `79ba9b92c`.

## Security and privacy review before publication

Run `git status --ignored` and inspect the staged diff. Never publish boot or
userdata images, proprietary `*.mbn`/`*.jsn` blobs, WLAN passwords, Bluetooth
pairing databases, SSH keys, Wine prefixes, or installed applications.
