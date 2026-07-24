# Software Setup - OS Flashing, NVMe Migration & Boot Verification

This document covers everything from flashing Raspberry Pi OS to verifying the Pi 5 is booting from the NVMe SSD.



## Flashing the Micro SD Card

The SD card was flashed using Raspberry Pi Imager on a Windows machine. Attempts using Imager v1.9.6 on Fedora (via Flatpak) failed to write the WiFi and user account configuration correctly, so Windows was used as a workaround.

**Imager settings:**

- Device: Raspberry Pi 5
- OS: Raspberry Pi OS Lite (64-bit)
- Storage: 29.5GB SD card

**OS Customisation:**

- Hostname: `lp-pi5`
- Username: `lliiiamm`
- WiFi SSID and password configured
- SSH enabled via the Services tab
- Locale set to Europe/London, keyboard layout GB

Lite was chosen over the full desktop version because the Pi is headless, it runs as a server over SSH with no monitor attached. The desktop environment would be wasted overhead.



## First Boot Issues and Troubleshooting

After the initial flash on Fedora, the Pi failed to appear on the network. Troubleshooting steps taken:

**Network scanning from the Fedora ThinkPad:**

```bash
nmap -sn 192.168.0.0/24
```

This sends a ping sweep across all 256 addresses on the subnet and reports which hosts respond. The Pi did not appear.

**Checking the router admin page** at `192.168.0.1` - no new device listed.

**Pulling the SD card and inspecting the boot partition:**

```bash
cat /run/media/lliiiamm/bootfs/network-config
```

The file contained only commented-out template text. The WiFi credentials had not been written by Imager. The file was edited manually using nano and replaced with a working netplan configuration:

```yaml
network:
  version: 2
  wifis:
    wlan0:
      dhcp4: true
      optional: true
      access-points:
        "VM9960629":
          password: "YOUR_WIFI_PASSWORD"
      regulatory-domain: GB
```

An SSH enable file was also created on the boot partition:

```bash
touch /run/media/lliiiamm/bootfs/ssh
```

Raspberry Pi OS checks for a file named `ssh` on the boot partition at startup and enables the SSH service if it is present.

**Root cause:** Imager v1.9.6 on Fedora did not write the WiFi credentials or user account correctly. Reflashing via Windows resolved all three issues - WiFi, SSH, and user account - in one pass.

This confirmed:

- The Pi 5 was booting correctly
- WiFi configuration was working
- SSH was enabled
- The user account had been created correctly



## Verifying the NVMe SSD

After successfully SSHing in, the first priority was confirming the NVMe SSD was detected.

**System information check:**

```bash
hostname
whoami
cat /etc/os-release
```

Confirmed the Pi was running Debian 13 (Trixie) as `lliiiamm` on `lp-pi5`.

**Storage check:**

```bash
lsblk -o NAME,SIZE,TYPE,MOUNTPOINT,MODEL
```

The SSD appeared as:

```
nvme0n1     238.5G  BG6 KIOXIA 256GB
```

This confirmed the X1001 adapter and PCIe connection were working correctly. The blue LED on the X1001 confirms power but does not confirm communication, `lsblk` is the reliable check.



## Cloning the SD Card to NVMe

With the Pi still running from the SD card, the OS was cloned onto the NVMe SSD using `rpi-clone`.

**Installing rpi-clone:**

```bash
git clone https://github.com/geerlingguy/rpi-clone.git
cd ~/rpi-clone
sudo cp rpi-clone rpi-clone-setup /usr/local/sbin/
```

**Confirming installation:**

```bash
which rpi-clone
```

Returns `/usr/local/sbin/rpi-clone`.

**Verifying the disk layout before cloning:**

```bash
lsblk -o NAME,SIZE,MODEL
```

```
NAME          SIZE  MODEL
mmcblk0      29.5G
nvme0n1     238.5G  BG6 KIOXIA 256GB
```

**Running the clone:**

```bash
sudo rpi-clone nvme0n1 -f
```

The `-f` flag is required when the target drive has no existing partition table. The clone process created the boot and root partitions on the SSD, copied the full OS installation, and updated the boot configuration to point to the NVMe drive.

After completion:

```
nvme0n1
├─nvme0n1p1  bootfs
└─nvme0n1p2  rootfs
```



## Configuring NVMe Boot

Before removing the SD card, the bootloader configuration was checked:

```bash
vcgencmd bootloader_config | grep BOOT_ORDER
```

```
BOOT_ORDER=0xf461
```

NVMe boot was already supported and configured in the firmware. No changes were needed.

The Pi was shut down cleanly:

```bash
sudo shutdown now
```

The SD card was removed and the Pi was powered back on with only the NVMe SSD connected.



## Verifying NVMe Boot

After removing the SD card, SSH was re-established successfully. Boot device confirmed:

```bash
findmnt /
```

```
TARGET  SOURCE
/       /dev/nvme0n1p2
```

Full storage layout:

```bash
lsblk -o NAME,SIZE,MODEL,MOUNTPOINT
```

```
nvme0n1      238.5G  BG6 KIOXIA 256GB
├─nvme0n1p1  512M                      /boot/firmware
└─nvme0n1p2  238G                      /
```

The Pi was no longer running from the SD card.



## SSD Health Check

Before rebuilding services, the SSD health was verified using `nvme-cli`:

```bash
sudo apt install nvme-cli
sudo nvme smart-log /dev/nvme0n1
```

Results:

```
critical_warning  : 0
temperature       : 34°C
available_spare   : 100%
percentage_used   : 0%
media_errors      : 0
```

7 unsafe shutdowns were recorded from testing and hardware changes during setup. No media errors or warnings were present. The SSD is confirmed suitable for continuous 24/7 operation.



## Final Verification

A reboot test was performed to confirm the system comes back up from NVMe without the SD card:

```bash
sudo reboot
```

After reconnecting over SSH:

```bash
findmnt /
```

```
TARGET  SOURCE
/       /dev/nvme0n1p2
```

The migration is complete. The Pi 5 is running entirely from NVMe and is ready for Pi-hole and service reinstallation.
