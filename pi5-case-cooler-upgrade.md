# Raspberry Pi 5 Case Upgrade and NVMe Verification

## Why the Case Upgrade

After getting Docker, Twingate and Pi-hole running, I still had the Pi sitting bare on my desk and needed to install a case and some sort of cooling solution. I decided to go for the Geekworm P579-H500 case, which came bundled with the official active cooler (H500).

When I began to install the active cooler to the Pi, I realised I forgot to remove the heatsink that was previously installed onto the CPU. This meant the cooler would not sit flush and do its job properly. I began to twist the heatsink to the right and left to break the seal, however there was no give.

The solution I came up with was to stress test the Pi for 15 minutes in hope that the thermal paste holding the heatsink down would become malleable enough for it to pop right off, which it did.

```bash
stress -c 4 -t 600
```

I monitored this in a separate window using `htop`.

---

## Installing the Case

After I confirmed that the cooler was seated properly and plugged in correctly, I booted into the Pi to confirm that I hadn't knocked anything loose or broken anything. Everything was working as planned and I began to seat it inside its case.

---

## Recovering the NVMe Boot

After fitting the Pi into its case, I expected it to boot straight from the NVMe SSD. Instead, the Pi would power on but never become available over SSH.

Initially, it looked like the SSD had corrupted or I had messed something up.

To investigate, I temporarily removed the NVMe drive and booted the Pi from the original microSD card. This confirmed the hardware itself was working correctly and that the problem was isolated to the NVMe installation.

The SSD was then mounted manually so I could compare its filesystem against the working microSD installation. During this investigation I discovered several important differences.

The cloned NVMe installation was missing the NetworkManager configuration that had been generated after the initial Raspberry Pi OS setup. As a result, the system had no valid network configuration after booting from the SSD, making it appear completely offline.

The following directories were compared between the SD card and the cloned NVMe installation:

```
/etc/netplan
```

and

```
/etc/NetworkManager
```

The NetworkManager generated Netplan configuration files existed on the SD card but were either missing or empty on the cloned SSD.

After confirming this, the working Netplan configuration was copied across to the NVMe installation.

```bash
sudo cp /etc/netplan/*.yaml /mnt/nvmeroot/etc/netplan/
sudo chmod 600 /mnt/nvmeroot/etc/netplan/*.yaml
```

Once the files had been copied, I confirmed that the wireless and Ethernet configurations were now present on the SSD.

The Wi-Fi configuration contained my saved wireless network along with the NetworkManager UUID, allowing the Pi to reconnect automatically after booting.

After unmounting the SSD and rebooting from NVMe, the Pi immediately reappeared on the network and SSH access was restored.

In the end, the clone itself had been successful. The issue was not caused by the SSD, Docker, or Raspberry Pi OS, but by the cloned installation missing the generated network configuration files required for NetworkManager to bring up the interfaces.

This took around an hour and a half to diagnose and resolve, but the process gave me a much better understanding of how Raspberry Pi OS stores its network configuration and reinforced the importance of verifying a cloned system before assuming the migration has completed successfully.

---

## Verifying the NVMe Migration

To confirm I was definitely working from my SSD and not the micro SD, I ran the following:

```bash
findmnt /
```

```text
TARGET SOURCE         FSTYPE OPTIONS
/      /dev/nvme0n1p2 ext4   rw,noatime
```

And the storage layout:

```bash
lsblk
```

```text
nvme0n1
├─nvme0n1p1   /boot/firmware
└─nvme0n1p2   /
```

The Pi is booting entirely from the NVMe SSD. The microSD card is no longer in use.

---

## Verifying Docker Services

After the reassembly, I confirmed both Docker containers came back up automatically.

```bash
docker ps
```

```text
pihole               Up (healthy)
twingate-lp-pi5-tg   Up (healthy)
```

Both services survived the migration and case swap without needing any reconfiguration.

---

## Verifying SSH Access

One of the most important checks after any hardware change is making sure remote management still works.

```bash
ssh lliiiamm@192.168.0.124
```

The connection completed successfully. SSH started automatically after boot, so the Pi can continue to be managed remotely without needing a monitor or keyboard plugged in.

---

## Verifying the Case Fan

After fitting the case, I wanted to confirm the fan was being detected by the OS.

```bash
cat /sys/class/thermal/cooling_device*/type
```

```text
pwm-fan
```

The OS recognised the fan hardware. I also checked the CPU temperature at idle:

```bash
vcgencmd measure_temp
```

```text
46°C
```

The fan uses automatic PWM control, so it stays quiet or completely stopped while the system is idle. It only ramps up once the CPU hits predefined temperature thresholds. Under load, the CPU temperature climbed steadily towards 60°C, confirming the thermal monitoring was working as expected.

---

## Unexpected SSH Behaviour

During testing, one SSH session became unresponsive and stopped accepting keyboard input.

The Pi itself was still online. I confirmed this with:

```bash
ping 192.168.0.124
```

ICMP responses came back consistently. Opening a new SSH session from another terminal worked without any issues, so the problem was isolated to the original session rather than the Pi itself.

No reboot or further troubleshooting was needed.

---

## Final Checks

Before calling the migration complete, I confirmed:

- Pi is booting from the NVMe SSD.
- Case fan detected and responding to temperature.
- SSH accessible remotely.
- Docker started automatically on boot.
- Pi-hole reporting healthy.
- Twingate Connector reporting healthy.
- Pi seated securely inside the case.

At this point the Raspberry Pi is fully operational in the Geekworm P579-H500 case, booting from NVMe storage and running all previously deployed services.
