## Summary

Initial enablement for the Wi-Fi Samsung Galaxy Tab S6 Lite (2022), model
SM-P613 / `gta4xlve`, based on the existing SM7125 Samsung device support.

This is intentionally a draft. It preserves the tested bring-up order and
asks for review direction before individual changes are proposed for merge.

## Series

1. add the base `gta4xlve` DTS;
2. clock quaternary and quinary MI2S links;
3. set MI2S codec frame layout;
4. add the ISL98608 display-bias regulator;
5. add Himax HX83102E SPI touchscreen support;
6. add BOE TV104WUM panel support and DCS backlight;
7. fix QCA/WCN3991 reinitialisation and extended-advertising handling;
8. enable panel, touch, audio and Wi-Fi in the device DTS.

## Tested on hardware

- direct Samsung Android-bootimg boot;
- display, backlight and touchscreen;
- Freedreno FD618 GPU;
- speaker audio through QUINARY_MI2S and TAS256x amplifiers;
- Wi-Fi; and
- Bluetooth controller plus A2DP playback.

## Known limits and requested review

The device still uses Samsung/Qualcomm vendor firmware and this branch is based
on the project's 6.14.7 SM7125 tree. It is not an assertion that the complete
series is ready for Linux mainline. The generic drivers and Bluetooth changes
may need splitting, rebasing and subsystem-specific review; the DTS should
follow its accepted dependencies.

Camera, wired headset/microphone paths, suspend/resume and cellular support
were not validated by this series. No proprietary firmware or user data is
included.
