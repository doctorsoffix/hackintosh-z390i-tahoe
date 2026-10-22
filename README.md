# Hackintosh – ASUS ROG Strix Z390-I · i7-9700K · RX 6700 XT · macOS Tahoe 26

OpenCore-EFI für macOS **Tahoe 26.7.1** (25G241) auf einem Mini-ITX-System mit Coffee Lake und Navi 22.

> **Wichtig:** Seriennummer, MLB, SystemUUID und ROM in der `config.plist` sind **Platzhalter**.
> Vor dem ersten Start eigene Werte erzeugen (siehe [SMBIOS](#smbios)).

## Hardware

| Komponente | Modell |
|---|---|
| CPU | Intel Core i7-9700K (Coffee Lake) |
| Mainboard | ASUS ROG Strix Z390-I Gaming |
| GPU | Gigabyte Aorus Radeon RX 6700 XT (Navi 22), iGPU UHD 630 headless |
| WLAN / Bluetooth | Intel Wireless-AC 9560 (CNVi, onboard) |
| LAN | Intel I219-V (onboard) |
| Audio | Realtek ALCS1220A (onboard) |
| SMBIOS | iMac20,2 |

## Was funktioniert

| Funktion | Status | Lösung |
|---|---|---|
| Grafik RX 6700 XT, 2 Monitore (HDMI + DP) | ✅ | NootRX |
| iGPU (Quick Sync, Videodekodierung) | ✅ | headless, `AAPL,ig-platform-id 0300913E` |
| WLAN | ✅ | AirportItlwm (Verbinden über Systemeinstellungen) |
| Bluetooth | ✅ | IntelBluetoothFirmware/IntelBTPatcher **2.5.1 (lshbluesky-Fork)** + NVRAM-Einträge |
| Onboard-Audio | ✅ | AppleALC + **SimpleHDA** (Layout 1) |
| LAN | ✅ | IntelMausi |
| USB (alle Ports inkl. beider Front-Ports, USB-C) | ✅ | USBToolBox + UTBMap (15 Ports) |
| iCloud / iMessage | ✅ | gültige eigene SMBIOS-Werte nötig |
| AirDrop / Hotspot / Continuity Camera | ❌ | unter Tahoe mit Intel-WLAN derzeit nicht möglich |

## Versionen

| Komponente | Version |
|---|---|
| OpenCore | 1.0.8 |
| Lilu | 1.7.2 |
| VirtualSMC / SMCProcessor / SMCSuperIO | 1.3.9 |
| WhateverGreen | 1.7.2 (deaktiviert, NootRX übernimmt) |
| AppleALC | 1.9.8 |
| SimpleHDA | 1.0.0 (**nicht enthalten**, siehe unten) |
| NootRX | 1.0.0 |
| AirportItlwm | 2.4.0 (Tahoe-Build) |
| IntelBluetoothFirmware / IntelBTPatcher | 2.5.1 ([lshbluesky](https://github.com/lshbluesky/IntelBluetoothFirmware)) |
| BlueToolFixup | 2.7.2 |
| IntelMausi | 1.0.8 |
| USBToolBox | 1.2.0 |

ACPI (SSDTTime): `SSDT-PLUG`, `SSDT-EC`, `SSDT-USBX`, `SSDT-AWAC`, `SSDT-PMC`, `SSDT-SBUS-MCHC`

## Einrichtung

### 1. SimpleHDA hinzufügen

Apple hat **AppleHDA ab Tahoe entfernt**, deshalb geht AppleALC allein nicht mehr.
SimpleHDA hat keine Lizenz für Weiterverteilung und ist darum nicht im Repo.
[SimpleHDA-Tahoe-v1.0.0-RELEASE.zip](https://github.com/laobamac/SimpleHDA/releases) herunterladen und
`SimpleHDA.kext` nach `EFI/OC/Kexts/` kopieren. Der Eintrag in der `config.plist` ist schon vorhanden.

### 2. SMBIOS

Mit [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) für **iMac20,2** erzeugen und in
`PlatformInfo → Generic` eintragen: `SystemSerialNumber`, `MLB`, `SystemUUID`.
`ROM` = MAC-Adresse des LAN-Anschlusses (`ifconfig en0 | grep ether`).
Seriennummer auf [checkcoverage.apple.com](https://checkcoverage.apple.com) prüfen: „ungültig“ ist gut.

### 3. BIOS-Einstellungen

| Einstellung | Wert |
|---|---|
| CSM | Disabled |
| Secure Boot | Other OS |
| Fast Boot | Disabled |
| VT-d | Disabled (oder `DisableIoMapper` = true, ist gesetzt) |
| Intel Virtualization (VT-x) | Enabled |
| Above 4G Decoding | Enabled |
| Primary Display | PEG |
| iGPU Multi-Monitor | Enabled |
| DVMT Pre-Allocated | 64M |
| XHCI Hand-off | Enabled |

## Wichtige Erkenntnisse

- **Installer hing bei „Waiting on boot-uuid-media“:** macOS sah den USB-Stick nicht. Lösung: USBToolBox + USB-Map.
- **Neustart-Schleife in der 2. Installationsphase:** `SecureBootModel = Disabled` (und WLAN/BT-Kexts während der Installation aus).
- **ASUS-Meldung „The system has POSTed in safe mode“ nach jedem Neustart:** `Kernel → Quirks → DisableRtcChecksum = true`.
- **Bluetooth unter Tahoe:** Die offizielle IntelBTPatcher 2.4.0 endet bei Sequoia, deshalb Fork 2.5.1 plus
  `bluetoothExternalDongleFailed = 00` und `bluetoothInternalControllerInfo = 00…` (14 Byte) in NVRAM Add **und** Delete
  ([Issue #505](https://github.com/OpenIntelWireless/IntelBluetoothFirmware/issues/505)).
- **WLAN und Bluetooth gleichzeitig** mit der offiziellen BT-Version 2.4.0 führten zum Absturz, mit dem Fork 2.5.1 läuft beides.
- **DisplayPort-Monitor schwarz** nach dem Textmodus: an einem DP-Anschluss der RX 6700 XT blieb er schwarz, am mittleren DP-Anschluss läuft er.
- **USB-Map:** Basis ist die Map von [Edmund-J/macDesk](https://github.com/Edmund-J/macDesk) (gleiches Board). Dort war nur ein Front-Port freigegeben.
  Statt HS11 (AIO) und HS13 (Aura) sind hier HS10/SS10 (zweiter Front-Port) aktiv.
- **macOS-Updates:** Vor jedem Update Kompatibilität von NootRX, SimpleHDA und dem Bluetooth-Fork prüfen.

## Credits

[Acidanthera](https://github.com/acidanthera) (OpenCore, Lilu, VirtualSMC, AppleALC, WhateverGreen, IntelMausi, BrcmPatchRAM) ·
[ChefKiss](https://github.com/ChefKissInc) (NootRX) ·
[OpenIntelWireless](https://github.com/OpenIntelWireless) (itlwm, IntelBluetoothFirmware) ·
[lshbluesky](https://github.com/lshbluesky/IntelBluetoothFirmware) (Bluetooth-Fork) ·
[laobamac](https://github.com/laobamac/SimpleHDA) (SimpleHDA) ·
[USBToolBox](https://github.com/USBToolBox) ·
[corpnewt](https://github.com/corpnewt) (SSDTTime, GenSMBIOS) ·
[Edmund-J/macDesk](https://github.com/Edmund-J/macDesk) (SSDTs, USB-Map-Basis) ·
[Dortania](https://dortania.github.io/OpenCore-Install-Guide/)
