# Raspberry Pi 5 - NVMe SSD Migration & Homelab Server Build

A hands-on homelab project documenting the migration of a Raspberry Pi 5 from a failing microSD card to a Kioxia NVMe SSD, restoring it as a stable 24/7 home server.



## Overview

The Pi 5 was originally running Pi-hole on a microSD card. After three days of continuous operation the SD card failed, taking the entire home network offline and every device lost internet access because Pi-hole was also handling DHCP.

The solution was to migrate the OS to an NVMe SSD via a PCIe HAT, making the Pi reliable enough to run as a permanent home server.

**The outcome:** Pi 5 booting from NVMe, SSH accessible, SSD health verified, ready for Pi-hole and service reinstallation.


## Hardware

| Component | Details |
|---|---|
| Raspberry Pi 5 | 8GB RAM |
| Geekworm X1001 | PCIe to M.2 NVMe HAT |
| Kioxia BG6 | 256GB M.2 2230 NVMe SSD (TLC NAND) |
| Geekworm P579 | Case with H500 active cooler |
| Virgin Media Hub | Home router (192.168.0.1) |



## Skills Demonstrated

- Linux CLI (Fedora and Raspberry Pi OS)
- Network troubleshooting (nmap, ping, SSH)
- Hardware assembly (PCIe HAT, ribbon cable, active cooler)
- Disk management (lsblk, rpi-clone, findmnt)
- Storage verification (nvme-cli, SMART data)
- Cloud-init and netplan configuration
- Boot order and EEPROM configuration on Pi 5



## Project Files

- [Physical Build](./physical-build.md) — Hardware assembly, PCIe HAT installation, ribbon cable routing, and case build
- [Software Setup](./software-setup.md) — OS flashing, SSH configuration, NVMe detection, SD to NVMe clone, boot verification, and SSD health check



## Final System State

```
hostname:    lp-pi5
boot device: /dev/nvme0n1p2
SSD model:   BG6 KIOXIA 256GB
SSD health:  0 errors, 34°C, 100% spare capacity
```

The Pi 5 is now running entirely from NVMe and survives reboots without the SD card present.
