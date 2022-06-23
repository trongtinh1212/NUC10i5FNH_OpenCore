# Actively Used Files

## Current NUC BIOS Revision

FNCML357.0056.2022.0223.1614

## Fix CFG Lock

setup_var_cv CpuSetup 0x3E 0x01 0x00 (Above BIOS Version, re-apply this everytime you reset bios settings, if not, it won't boot)

## Generating Personalised SMBIOS

It is important to generate a personalised SMBIOS using `Macmini8,1` as target model. To complete the respective OpenCore configuration section for SMBIOS (namely `MLB`, `SystemSerialNumber` and `SystemUUID` keys) it is advised to use [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) scripts and add the generated values in the respective places in `config.plist` file.

As for the _unique_ number needed in the **ROM** value of the `PlatformInfo` section, the recommended method is to take the 12 digits from the **en0** network controler (without the colons) and convert them to [Base64](https://cryptii.com/pipes/hex-to-base64) for use as `<data>` under `<key>ROM</key>` in the OpenCore configuration file. The value `<data>ESIzRFVm</data>` is generic; read more over at [Dortania](https://dortania.github.io/OpenCore-Post-Install/universal/iservices.html#fixing-rom).

To confirm that the injected value works persistently across reboots, one can either run in Terminal [iMessageDebug](https://mac.softpedia.com/get/System-Utilities/iMessageDebug.shtml) or the command:<br/>
`nvram -x 4D1EDE05-38C7-4A6A-9CC6-4BCCA8B38C14:ROM` and verify the output.

## OpenCore Version Installed

These files have been running without issues with the official OpenCore releases on [GitHub](https://github.com/acidanthera/OpenCorePkg/releases).

The original configuration, especially setting the "Quirks" to the correct values for this _specific_ NUC chipset and platform, was done by following the [Dortania Guide to OpenCore](https://dortania.github.io/OpenCore-Install-Guide/config.plist/comet-lake.html) and has remained pretty much the same with the last OpenCore iterations.


## OpenCore Main Parameters

1. The following **important** DSDT patches are enabled:

Rename `_STA` to `XSTA` for device `(H_EC)` allowing to disable this native device and use **SSDT-EC-USBX.aml** that injects a fake `(EC)` device that is absolutely required for a successful boot into macOS.

2. The following custom SSDTs are included, defined and enabled:

* SSDT-APPLE.aml
* SSDT-AWAC.aml
* SSDT-DTGP.aml
* SSDT-EC-USBX.aml
* SSDT-HPTE.aml
* SSDT-NAMES.aml
* SSDT-PLUG.aml
* SSDT-PMCR.aml
* SSDT-SBUS.aml
* SSDT-TITAN.aml
* SSDT-XOSI.aml → disabled

The ACPI code and justification for each custom SSDT is described in more detail in the [SSDTs](../SSDTs) section.

3. The following kexts are included, defined and required:

* [AirportItlwm.kext](https://github.com/OpenIntelWireless/itlwm/releases)
* [AppleALC.kext](https://github.com/acidanthera/AppleALC/releases)
* [IntelBluetoothFirmware.kext](https://github.com/OpenIntelWireless/IntelBluetoothFirmware/releases)
* [IntelBluetoothInjector.kext](https://github.com/OpenIntelWireless/IntelBluetoothFirmware/releases) → disabled
* [BlueToolFixup.kext](https://github.com/acidanthera/BrcmPatchRAM/releases) :warning:
* [IntelMausi.kext](https://github.com/acidanthera/IntelMausi/releases)
* [Lilu.kext](https://github.com/acidanthera/Lilu/releases)
* [NVMeFix.kext](https://github.com/acidanthera/NVMeFix/releases) → disabled
* [SMCProcessor.kext](https://github.com/acidanthera/VirtualSMC/releases)
* [SMCSuperIO.kext](https://github.com/acidanthera/VirtualSMC/releases)
* [VirtualSMC.kext](https://github.com/acidanthera/VirtualSMC/releases)
* [WhateverGreen.kext](https://github.com/acidanthera/WhateverGreen/releases)
* [SATA-Unsupported](https://github.com/khronokernel/Legacy-Kexts/blob/master/Injectors/Zip/SATA-unsupported.kext.zip) - Adds support for a large variety of SATA controllers, mainly relevant for laptops which have issues seeing the SATA drive in macOS. We recommend testing without this first.
* [CtlnaAHCIPort.kext](https://github.com/dortania/OpenCore-Install-Guide/blob/master/extra-files/CtlnaAHCIPort.kext.zip) - macOS Big Sur Note: SATA-Unsupported but for Big Sur and ablove, will need to be used instead due to numerous controllers being dropped from the binary itself
* USBPorts.kext

## Sleep/Wake Parameters

The following parameters can be set via Terminal, according to the Dortania guide to [Fixing Sleep](https://dortania.github.io/OpenCore-Post-Install/universal/sleep.html#preparations):

```
sudo pmset autopoweroff 0; \
sudo pmset powernap 0; \
sudo pmset standby 0; \
sudo pmset proximitywake 0; \
sudo pmset tcpkeepalive 0
```

These `pmset` parameters above achieve the following:

* Disable `Auto Power-Off` → prevents this form of hibernation;
* Disable `Power Nap` → prevents periodically waking the computer for network and updates(but not the display);
* Disable `Standby` → minimises the time period between sleep and going into hibernation;
* Disable `Proximity Wake` → does not allow waking from an iPhone or an Apple Watch when they come near;
* Disable `TCP Keep Alive` → prevents the mechanism that wakes the computer up every 2 hours.
