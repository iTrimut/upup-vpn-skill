---
name: upup-vpn
description: |
  Manage the "upup" VPN infrastructure for 夏安 (Xia'an).
  Use when the user asks about:
  - VPN connection/disconnection on any device (Mac, iPhone, Android)
  - VPN server status, migration, or troubleshooting
  - Setting up VPN on a new device
  - Shadowsocks configuration (both server and client)
  - Managing DigitalOcean droplets (creating, deleting, migrating)
  - Checking/changing IP location via the VPN
  Any VPN or proxy-related tasks for the upup service.
  Credentials and server details are in the references.
---

# upup-vpn

## Overview

The upup VPN provides US West (San Francisco) internet access for 夏安's devices.
It uses two parallel services on the same DigitalOcean droplet:

- **Shadowsocks** (primary) — SOCKS5 proxy, easy to set up on any device
- **IPsec/L2TP + IKEv2** (fallback) — Full VPN tunnel, native OS support

## Server Infrastructure

The server is a **DigitalOcean droplet** in San Francisco (SFO3).

### When a user asks for VPN help:

1. **Check if the server is running** — SSH in and verify services
2. **Check which client they're on** — Different setup per platform
3. **Use credentials from reference** — Never invent or ask user for creds

### Quick Reference

All credentials, server IPs, and client configs are in `references/credentials.md`.
Read that file at the start of any VPN task.

## Client Setup Guides

### macOS (Shadowsocks — current primary)

The Shadowsocks client is installed via Homebrew:

```
/opt/homebrew/opt/shadowsocks-libev/bin/ss-local -c /opt/homebrew/etc/shadowsocks-libev.json
```

A menu bar toggle app exists at `~/Desktop/US VPN.app`.
A terminal menu script exists at `/Users/xiaan/Documents/VPNTool/ss_vpn.sh`.

#### If the app needs to be rebuilt:

Recreate using `osacompile` (in `/Users/xiaan/Documents/VPNTool/`):
1. Update `ss_vpn.sh` with current config
2. Use `osacompile -o "VPN Tool.app"` with embedded AppleScript
3. Copy the script to `Contents/Resources/`
4. Deploy to `~/Desktop/US VPN.app`

### iPhone / iPad

Use **Shadowrocket** or **Outline** from App Store.
QR code config: `ss://` URL with base64-encoded method:password.

### Android

Use **Shadowsocks** app from Google Play or GitHub releases.
QR code or manual config using credentials from reference.

### Direct IKEv2 (any platform)

Mobileconfig / .sswan / .p12 files are on the server at `/root/vpnclient.*`.
Re-export if needed: `sudo ikev2.sh --exportclient <name>`.

## Maintenance Tasks

### Server status check
```bash
ssh -i ~/.ssh/id_ed25519 root@144.126.210.202 "systemctl status ipsec shadowsocks-libev --no-pager | grep Active"
```

### Restart services
```bash
ssh -i ~/.ssh/id_ed25519 root@144.126.210.202 "systemctl restart ipsec shadowsocks-libev"
```

### Firewall rules
UFW is enabled. Ports: 22(SSH), 500/4500(IPsec), 8388(Shadowsocks).

## Known Issues

- `.mobileconfig` on macOS 26.5 installs as managed profile but may not create a `scutil --nc` service.
- The AppleScript `.app` wrapper may show "unverified developer" on first launch.
- If machine's IP has network issues, try restarting `ss-local`.
