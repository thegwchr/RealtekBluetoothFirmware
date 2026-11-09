# Supported Devices

On Realtek combo cards the Wi-Fi may be PCIe while the **Bluetooth radio is an
internal USB device**. This kext matches that USB Bluetooth device and uploads
its patch firmware.

Both Realtek firmware formats are supported — **v2 (`RTBTCore`)** and **v1
(`Realtech`)**:

| Chip            | lmp_subver | hci_rev | hci_ver | Firmware              | Format |
|-----------------|-----------|---------|---------|-----------------------|--------|
| RTL8822C/CE     | 0x8822    | 0x000c  | 0x0a    | `rtl8822cu_fw.bin`    | v2 |
| RTL8851B        | 0x8851    | 0x000b  | 0x0c    | `rtl8851bu_fw.bin`    | v2 |
| RTL8852B/BE     | 0x8852    | 0x000b  | 0x0b    | `rtl8852bu_fw.bin`    | v2 |
| RTL8852C        | 0x8852    | 0x000c  | 0x0c    | `rtl8852cu_fw_v2.bin` | v2 |
| RTL8852BT/BE-VT | 0x8852    | 0x0087  | 0x0c    | `rtl8852btu_fw.bin`   | v2 |
| RTL8922A        | 0x8922    | 0x000a  | 0x0c    | `rtl8922au_fw.bin`    | v2 |
| RTL8723B        | 0x8723    | 0x000b  | 0x06    | `rtl8723b_fw.bin`     | v1 |
| RTL8821A        | 0x8821    | 0x000a  | 0x06    | `rtl8821a_fw.bin`     | v1 |
| RTL8821C        | 0x8821    | 0x000c  | 0x08    | `rtl8821c_fw.bin` + config | v1 |
| RTL8761A        | 0x8761    | 0x000a  | 0x06    | `rtl8761a_fw.bin`     | v1 |
| RTL8761B/BUV    | 0x8761    | 0x000b  | 0x0a    | `rtl8761bu_fw.bin` + config | v1 |
| RTL8761C        | 0x8761    | 0x000e  | 0x00    | `rtl8761cu_fw.bin` + config | v1 |
| RTL8822B/BE     | 0x8822    | 0x000b  | 0x07    | `rtl8822b_fw.bin` + config | v1 |
| RTL8852A/AE     | 0x8852    | 0x000a  | 0x0b    | `rtl8852au_fw.bin`    | v1 |

The USB IDs matched (vendor `0x0BDA` plus the various module vendors) are taken
from the Linux `btusb.c` device table — every Realtek BT USB ID is listed in
`Info.plist`. The actual chip is identified at runtime from its HCI version, so
a board only needs its USB ID present to be picked up; an ID whose chip isn't in
the table above is simply released untouched.

> Not yet handled: **RTL8723A** (older non-epatch firmware) and **RTL8723D**
> (its mandatory config blob is absent from linux-firmware).

## Finding your device ID

In macOS, open **System Information → USB** and look for a Realtek "Bluetooth
Radio" entry (vendor `0x0BDA`), or on Linux/Windows run `lsusb` / check Device
Manager. The 4-digit product ID after `0bda:` is what must appear in
`Info.plist`.

If your device reports a different product ID than those listed
above, add it as a new personality in `Info.plist` (the `idProduct` value is in
decimal).

## Adding another Realtek chip

The firmware parser is generic across the Realtek family. To add support for a
new controller:

1. Add its firmware `.bin` (from
   [linux-firmware/rtl_bt](https://gitlab.com/kernel-firmware/linux-firmware/-/tree/main/rtl_bt))
   to `fw/` and run `make fw`.
2. Add a row to `kIcTable` in `BtRtl.cpp` with its `lmp_subver` / `hci_rev` /
   `hci_ver` and firmware name.
3. Add an `idProduct` personality to `Info.plist`.
