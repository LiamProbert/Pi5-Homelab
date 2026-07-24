# Physical Build — Raspberry Pi 5 NVMe HAT Assembly

This document covers the hardware assembly of the Raspberry Pi 5 with the Geekworm X1001 NVMe HAT and Kioxia BG6 SSD, including the original build and the later reassembly into the P579 case with active cooler.

---

## Hardware

| Item | Details |
|---|---|
| Raspberry Pi 5 | 8GB RAM |
| Geekworm X1001 | PCIe to M.2 NVMe HAT, supports 2230/2242/2260/2280 |
| Kioxia BG6 | 256GB M.2 2230 NVMe SSD |
| PCIe FFC Ribbon Cable | SupTronics RPi5, 30mm |
| Geekworm P579 | Case with H500 official active cooler |
| Screws | 6 used from X1001 kit, 2 spare |

---

## ESD Precautions

Before handling any boards, ground yourself by touching a metal radiator or the metal chassis of a plugged-in appliance. Hold all boards by their edges and avoid touching gold contacts, chips, or board surfaces. Set boards down on a wooden surface or anti-static bag, never on carpet or fabric.

---

## SSD Installation

1. Seat the Kioxia BG6 into the M.2 slot on the X1001 at the 2230 position (the shortest slot position)
2. Press the drive down at a slight angle and secure it with the brass standoff and silver screw
3. The drive should sit flat against the board with no flex or movement

---

## PCIe Ribbon Cable

The PCIe FFC ribbon cable connects the Pi 5 to the X1001 HAT. Getting this right matters — a poorly seated cable means the SSD won't be detected.

**On the Pi 5:**
- The connector is the small flat one directly above where "PCIe" is printed on the board, near the HDMI ports
- It is not the UART connector and not the camera or display connectors
- Flip the latch up with a fingernail — it hinges, it does not come off
- Slide the cable in with gold contacts facing down towards the PCB
- Press the latch down to lock

**On the X1001:**
- Same process, gold contacts face down towards the board

---

## HAT Mounting

The X1001 HAT mounts on top of the Pi 5 via standoffs. Once the ribbon cable is connected at both ends and the SSD is seated, secure the HAT using the standoffs and screws from the kit.

---

## Reassembly with P579 Case and H500 Active Cooler

The P579 case arrived with the H500 official active cooler. The original red Pi 5 case could not be used because the X1001 HAT does not fit inside it.

Reassembly order:

1. Remove the X1001 HAT from the Pi by lifting the ribbon cable latches at both ends and disconnecting the cable
2. Clip the H500 cooler onto the Pi's SoC — it sits directly on the main processor chip
3. Plug the fan cable into the 4-pin fan header on the Pi board
4. Reconnect the ribbon cable: gold contacts face down on both connectors, press latches to lock
5. Mount the X1001 HAT back on top using the standoffs
6. Place the whole assembly into the P579 case

The fan is controlled automatically by the Pi based on temperature. No configuration is required.

---

## Notes

- The heatsink already on the Pi is sufficient for light workloads like Pi-hole without the fan attached
- The blue LED on the X1001 confirms the adapter is receiving power, but does not confirm the SSD is communicating — use `lsblk` or `nvme list` over SSH to verify detection
- If the SSD is not detected after assembly, reseat the ribbon cable at both ends before assuming a hardware fault
