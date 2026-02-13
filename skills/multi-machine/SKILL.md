---
name: multi-machine
description: "Set up OpenClaw across multiple machines: gateway/node networking, same-subnet config, Tailscale VPN, discovery, and troubleshooting. Use when connecting a second computer, setting up remote nodes, or diagnosing network issues."
metadata:
  {
    "openclaw":
      {
        "emoji": "🌐",
        "requires": { "bins": ["openclaw"] },
      },
  }
---

# Multi-Machine Setup

Run OpenClaw gateway and nodes across multiple machines on the same network.

## Architecture

```
[Gateway Machine]          [Node Machine(s)]
    openclaw gateway  ←→   openclaw node run
    :3000 (LAN bind)       connects to gateway IP
```

The **gateway** is the central hub. **Nodes** connect to the gateway to receive work.

## Quick Setup

### 1. Gateway Machine

```bash
# Start gateway bound to LAN (not just localhost)
openclaw gateway run --bind lan --port 3000
```

Verify:
```bash
openclaw status --deep
# Should show "Listening on 0.0.0.0:3000" or LAN IP
```

### 2. Node Machine

```bash
# Connect to gateway
openclaw node run --gateway http://<GATEWAY_IP>:3000
```

Replace `<GATEWAY_IP>` with the gateway machine's local IP (e.g., `192.168.1.x`).

Find gateway IP:
- **Windows**: `ipconfig | findstr "IPv4"`
- **macOS/Linux**: `ifconfig | grep "inet " | grep -v 127.0.0.1`

### 3. Verify Connection

On gateway machine:
```bash
openclaw nodes status
```

Should show the connected node.

## Network Options

### Same Subnet (Simplest)

Both machines on the same WiFi/LAN. No extra config needed.

Requirements:
- Both on same network (e.g., `192.168.1.x/24`)
- No firewall blocking port 3000 between machines
- Gateway binds to LAN, not loopback

### Tailscale VPN (Recommended for Remote)

```bash
# Install on both machines
# https://tailscale.com/download

# Use Tailscale IP instead of LAN IP
openclaw node run --gateway http://<TAILSCALE_IP>:3000
```

Benefits: encrypted, works across networks, no port forwarding.

### Router Config (Advanced)

If machines are on different subnets:
1. Port forward `3000` to gateway machine
2. Allow traffic between subnets
3. Use gateway's WAN IP from node machine

## Troubleshooting

### Node Can't Connect

```bash
# Test connectivity from node machine
curl http://<GATEWAY_IP>:3000/health

# Check firewall (Windows)
netsh advfirewall firewall show rule name=all | findstr "3000"

# Check firewall (Linux)
sudo ufw status | grep 3000

# Check firewall (macOS)
sudo pfctl -s rules | grep 3000
```

### Gateway Not Discoverable

```bash
# Verify gateway is listening on LAN (not 127.0.0.1)
# Windows:
netstat -an | findstr "3000"
# Linux/macOS:
ss -ltnp | grep 3000
```

If showing `127.0.0.1:3000`, restart with `--bind lan`.

### Conflicting Gateways

Only ONE gateway should run on the network. If you had a previous gateway:

```bash
# Kill all gateway processes
# Windows:
taskkill /f /im node.exe /fi "WINDOWTITLE eq openclaw*"
# Linux/macOS:
pkill -f "openclaw.*gateway"
```

### Device Pairing Issues

```bash
# Check device identity
openclaw doctor

# Reset pairing if needed
openclaw node forget --all
```
