# Installation

## Requirements

- An OpenCore setup.
- [Lilu](https://github.com/acidanthera/Lilu)
- [BlueToolFixup](https://github.com/acidanthera/BrcmPatchRAM) — required on
  macOS 12 (Monterey) and later to enable third-party Bluetooth.
- A supported controller — see [Supported Devices](Supported-Devices.md).

No injector kext is needed.

## Building

```sh
make            # -> build/RealtekBluetoothFirmware.kext
make test       # host-side unit test of the firmware parser
make fw         # regenerate include/FwBinary.cpp from fw/*.bin
make clean
```

The result is a standard kext bundle ready to be code-signed and injected.

## Installing with OpenCore

1. Copy `RealtekBluetoothFirmware.kext` and `BlueToolFixup.kext` into
   `EFI/OC/Kexts`.
2. Add both to `config.plist` under `Kernel > Add` (after `Lilu.kext`).
3. Set the required NVRAM variables under both `NVRAM > Add > 7C436110-AB2A-4BBB-A880-FE41995C9F82`
   and `NVRAM > Delete > 7C436110-AB2A-4BBB-A880-FE41995C9F82` (with empty/zero Data values):
   - `bluetoothExternalDongleFailed` = `<00>` (1 byte)
   - `bluetoothInternalControllerInfo` = `<0000000000000000000000000000>` (14 zero bytes —
     required by `bluetoothd` on newer macOS builds such as macOS 26 Tahoe; without it
     `bluetoothd` may fall back to `BCM_4350C2` or fail controller initialization)
4. On macOS 14 (Sonoma) and newer (Sequoia / Tahoe), add `-btlfxboardid` to `boot-args` if your
   SMBIOS does not natively map an internal USB Bluetooth `board-id` (for example `MacBookPro16,2`);
   starting in macOS 14, `BlueToolFixup` only applies the `board-id` patch when `-btlfxboardid`
   is explicitly passed.

## Verifying

After reboot, Bluetooth should appear in System Settings. To confirm the
firmware upload ran:

```sh
log show --last boot --predicate 'eventMessage CONTAINS "RealtekFirmware"'
```

For example, on an RTL8822CE you should see the matched chip, `rom_version 3`, the parsed
payload size, the per-fragment download, and `firmware download complete`.

## Troubleshooting

- **No Bluetooth (`BCM_4350C2` fallback or `bluetoothd` restart loop) after a clean upload**
  (the log shows `firmware download complete` / `FirmwareLoaded = Yes` but Bluetooth never
  initializes): check that `bluetoothExternalDongleFailed` (`00`) and the 14-byte
  `bluetoothInternalControllerInfo` are set in OpenCore NVRAM (`Add` and `Delete`), add
  `-btlfxboardid` to `boot-args` on macOS 14+ when needed for your SMBIOS, and verify
  that `Lilu`/`BlueToolFixup` match your macOS build.
- **Device not matched at all** (no `RealtekFirmware` log lines): your USB
  product ID isn't in `Info.plist` — see [Supported Devices](Supported-Devices.md).
