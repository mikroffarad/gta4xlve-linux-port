# Recreating the local firmware input

The firmware used while developing this port came from an unpacked official
Samsung firmware release for the Wi-Fi SM-P613.  It is intentionally absent
from this repository: it contains Samsung/Qualcomm proprietary binaries and
must not be committed, uploaded to a Git forge, or redistributed here.

To reproduce the input, obtain an official firmware package for **SM-P613**
and the desired region through Samsung's own update tooling or download
channel.  Record the PDA/build identifier, CSC/region, download URL or tool
used, download date, and SHA-256 of the original archive in your own build
notes.  A firmware build can change the blobs even when the device model is
the same, so this information is necessary for a useful bug report.

Unpack the package locally and retain the contents supplied by Samsung's
`firmware_mnt` image.  On this device those files are needed by TrustZone;
copying superficially similar files from Android's `vendor/firmware` tree did
not provide an equivalent bootable result.  Build a local
`gta4xlve-firmware.tar.gz` that has these paths at its archive root:

```
usr/lib/firmware/qca/crbtfw32.tlv
usr/lib/firmware/qca/crnv32.bin
usr/lib/firmware/qca/crnv32u.bin
usr/lib/firmware/qcom/sm7125/gta4xlve/a615_zap.mbn
usr/lib/firmware/qcom/sm7125/gta4xlve/adsp.mbn
usr/lib/firmware/qcom/sm7125/gta4xlve/cdsp.mbn
usr/lib/firmware/qcom/sm7125/gta4xlve/ipa_fws.mbn
usr/lib/firmware/qcom/sm7125/gta4xlve/modem.mbn
usr/lib/firmware/qcom/sm7125/gta4xlve/venus.mbn
usr/lib/firmware/qcom/sm7125/gta4xlve/wlanmdsp.mbn
```

The exact installed-file list is the tracked
[`pmaports/firmware-samsung-gta4xlve/firmware.files`](../pmaports/firmware-samsung-gta4xlve/firmware.files).
Place the resulting archive next to that package's `APKBUILD`, update its
`sha512sums` entry with `abuild checksum`, and build it locally.  Do not add
the archive to a commit.

The archived recipe's checksum describes the private development copy only;
it is not a checksum for a currently downloadable Samsung release.  A newer
official package is expected to have a different checksum.  Preserve your
own manifest with the original-package hash and the hashes of the extracted
files so the result can be reproduced without publishing the blobs.
