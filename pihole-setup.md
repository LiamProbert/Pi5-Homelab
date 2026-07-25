# Pi-hole Setup

## Why Pi-hole

After getting Docker and Twingate running, the next service I wanted to deploy was Pi-hole.

The main reason for using Pi-hole was to have a network-wide DNS ad blocker instead of relying on individual browser extensions or device-specific blockers.

Pi-hole works by acting as the DNS server for devices on the network. Whenever a device tries to access a domain, the request is sent through Pi-hole first. If the domain matches a known advertising or tracking domain, Pi-hole blocks the request before it can leave the network.

The secondary reason for deploying Pi-hole was the visibility it provides. The dashboard allows me to see DNS requests being made across the network, which gives a better understanding of what devices are communicating with in the background.

This is useful for troubleshooting, identifying unwanted traffic, and generally getting more visibility over what is happening on my network.

## Deployment Method

Like the Twingate Connector, Pi-hole was deployed using Docker.

The reason for keeping Pi-hole containerised was consistency with the rest of the setup. Instead of installing services directly onto Raspberry Pi OS, Docker keeps everything separated and makes future maintenance easier.

A dedicated directory was created for the Pi-hole Docker Compose configuration:

```bash
mkdir -p ~/docker/pihole
cd ~/docker/pihole
nano compose.yaml
```

The Compose configuration used was:

```yaml
services:
  pihole:
    image: pihole/pihole:latest
    container_name: pihole
    restart: unless-stopped

    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "80:80/tcp"

    environment:
      TZ: "Europe/London"
      WEBPASSWORD: "yourpasswordhere"

    volumes:
      - ./etc-pihole:/etc/pihole
      - ./etc-dnsmasq.d:/etc/dnsmasq.d

    dns:
      - 127.0.0.1
      - 1.1.1.1
```

> Note: Replace `yourpasswordhere` with your own password. This is used to access the Pi-hole admin interface.

The container was deployed with:

```bash
docker compose up -d
```

After deployment, both Docker services were confirmed running:

```bash
docker ps
```

```
pihole               Up (healthy)
twingate-lp-pi5-tg   Up (healthy)
```

This confirmed Pi-hole was running alongside the existing Twingate Connector without any issues.

## Pi-hole Web Interface

The Pi-hole admin interface is available at:

```
http://192.168.0.124/admin
```

The dashboard provides:

- Live DNS query logs
- Number of blocked requests
- Client activity
- Blocklist management
- Query statistics

During setup the browser showed a "Not secure" warning. This is expected because Pi-hole does not automatically configure HTTPS. Since the interface is only accessible on the local network, this is acceptable for now. If remote access to the dashboard is needed in the future, this can be handled through Twingate rather than exposing the interface publicly.

## Password Reset

During setup the Pi-hole web interface password was reset using:

```bash
docker exec pihole pihole setpassword
```

To set a specific password:

```bash
docker exec pihole pihole setpassword yournewpassword
```

This allows access to the admin interface to be restored without recreating the container.

## DNS Architecture Decision

Before pointing devices at Pi-hole, a decision was needed on how DNS would be handled across the network.

### Option A: Pi-hole Handles DHCP

Pi-hole takes over DHCP as well as DNS. Every device automatically receives Pi-hole as its DNS server with no manual configuration required. The downside is that if the Pi goes offline, DHCP stops working and the whole household network is eventually affected.

### Option B: Virgin Media Router Handles DHCP

The router continues handling DHCP. DNS is manually set to `192.168.0.124` on individual devices.

Option B was chosen. The main reason was the previous experience where Pi-hole handling DHCP caused the entire household internet to go down when the Pi failed. With Option B, if the Pi goes offline only devices configured to use Pi-hole lose DNS resolution. The router continues functioning normally.

An important consideration with Option B: DNS settings are configured per network on phones and laptops. When a device leaves the house and connects to another network or mobile data, it automatically switches to that network's DNS. No manual switching needed when leaving the house.

## Pointing Devices at Pi-hole

### Fedora ThinkPad

```bash
nmcli con mod "SSID" ipv4.dns "192.168.0.124"
nmcli con up "SSID"
```

**Failsafe if something goes wrong:**

```bash
nmcli con mod "SSID" ipv4.dns ""
nmcli con up "SSID"
```

This clears the manual DNS and returns the ThinkPad to automatic DNS from the router.

### iPhone

Settings → WiFi → tap the network → Configure DNS → Manual → add `192.168.0.124`

This only applies while connected to the home WiFi. On any other network the device reverts to automatic DNS.

## Disabling Pi-hole

Since the Virgin Media router is still handling DHCP, Pi-hole can be stopped without affecting the entire network:

```bash
docker compose stop
```

The Pi-hole web interface also provides a disable option with selectable timers, which is useful for quickly checking if Pi-hole is causing an issue with a specific site or service.
