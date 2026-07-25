# Docker & Twingate Setup

## Docker Installation

After successfully migrating the Raspberry Pi 5 to the NVMe SSD, the next stage was preparing the system to run services.

The main goal was to use Docker as the base platform for running services in isolated containers. This makes future services easier to manage and avoids installing everything directly onto the operating system.

The first service planned for Docker was the Twingate Connector, which provides secure remote access into the home network without requiring open ports on the router.

Docker was already present on the system. Confirmed with:

```bash
docker --version
```

```
Docker version 29.6.2
```

The standard test container was run to verify the daemon was functioning:

```bash
docker run hello-world
```

```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

This confirmed:

- Docker daemon was running
- Containers could be created successfully
- The Pi 5 was able to pull images from Docker Hub
- ARM64 container images were supported correctly

## Docker Compose Verification

```bash
docker compose version
```

```
Docker Compose version v5.3.1
```

No existing Compose projects were running, as expected on a fresh install:

```bash
docker compose ls
```

## Docker Service Verification

```bash
systemctl status docker --no-pager -l
```

```
Active: active (running)
```

This confirmed:

- Docker was enabled through systemd
- Docker would automatically start after reboot
- The Docker daemon was operating correctly

One warning was present:

```
WARNING: No swap limit support
```

This is a common warning on Raspberry Pi systems and does not affect the planned services. The Twingate Connector has very low resource requirements so this can be safely ignored.

## Installing the Twingate Connector

Twingate was installed to solve the remote access problem. Without it, accessing the Pi or internal services from outside the home network would require exposing ports externally.

How Twingate works:

- The Raspberry Pi runs a Connector inside the home network
- The Connector creates an outbound encrypted connection to Twingate
- Remote devices connect through Twingate without requiring port forwarding

An existing Twingate network was already in place from the previous setup:

```
Network:  192.168.0.0/24
Name:     Homelab Network
```

The old connector from the previous Pi setup was removed before creating a new one.

## Creating the New Connector

A new connector was created from the Twingate Admin Console using the Docker deployment method.

Rather than running the generated Docker command directly, it was converted into a Docker Compose deployment for easier management. The container configuration is stored in a file and can be recreated or updated without needing to re-enter the original command.

```bash
mkdir -p ~/docker/twingate
cd ~/docker/twingate
nano compose.yaml
```

```yaml
services:
  twingate-connector:
    image: twingate/connector:1
    container_name: twingate-lp-pi5-tg

    restart: unless-stopped
    pull_policy: always

    sysctls:
      net.ipv4.ping_group_range: "0 2147483647"

    environment:
      TWINGATE_NETWORK: "liamp"
      TWINGATE_ACCESS_TOKEN: "ACCESS_TOKEN"
      TWINGATE_REFRESH_TOKEN: "REFRESH_TOKEN"
      TWINGATE_LABEL_HOSTNAME: "lp-pi5"
      TWINGATE_LABEL_DEPLOYED_BY: "docker"
```

> Note: Replace `ACCESS_TOKEN` and `REFRESH_TOKEN` with the values generated from the Twingate Admin Console when creating the connector. These are unique to each connector deployment.

## Deploying the Connector

```bash
docker compose up -d
```

```
✔ Image twingate/connector:1 Pulled
✔ Network twingate_default Created
✔ Container twingate-lp-pi5-tg Started
```

Verified the container was running:

```bash
docker ps
```

```
twingate/connector:1    Status: Up About a minute (healthy)
```

Checked the connector logs:

```bash
docker logs -f twingate-lp-pi5-tg
```

The connector went through the expected startup sequence:

```
State: Offline
State: Authentication
State: Authentication
State: Online
```

This confirmed:

- The connector container was running
- The authentication tokens were valid
- The Raspberry Pi successfully connected to Twingate
- The Pi was now acting as a Twingate gateway into the network

## Creating the SSH Resource

The connector provides the route into the network. Resources define what is actually reachable through it. An SSH resource was created in the Twingate Admin Console to allow access to the Pi.

The Pi's IP was confirmed first:

```bash
hostname -I
```

```
192.168.0.124
```

The other addresses shown (172.17.0.1, 172.18.0.1) are internal Docker bridge networks and were ignored.

Resource configuration:

```
Name:     Raspberry Pi 5 SSH
Address:  192.168.0.124
Port:     TCP 22 only
ICMP:     Allowed
Policy:   Everyone (single user homelab)
```

## Testing SSH Access

The first SSH test was performed locally from the Fedora ThinkPad. The connection initially returned:

```
WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!
```

This was expected. The Pi had been rebuilt, so the SSH host key was different from the one previously stored. The old entry was removed:

```bash
ssh-keygen -R 192.168.0.124
```

```
Host 192.168.0.124 found: line 4
known_hosts updated
```

SSH was then confirmed working:

```bash
ssh lliiiamm@192.168.0.124
```

This confirmed:

- SSH was running correctly
- The Pi user account was working
- The Pi was reachable on the network
- The SSH resource configuration was correct

## Current Status

Completed:

- Raspberry Pi OS running from NVMe SSD
- Docker installed and verified
- Docker Compose working
- Twingate Connector deployed and authenticated
- SSH Resource created
- SSH access confirmed locally

Remaining:

The final validation will be performed from an external network by disconnecting from home WiFi and connecting via Twingate Client to confirm the full remote access path is working end to end.
