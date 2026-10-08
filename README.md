# cloudflare-tunnel-setup

Turn your personal computer into a server that is reachable from the internet, **without** port forwarding, a public IP, or exposing your home network. Traffic goes through [Cloudflare Tunnel](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/) (`cloudflared`), which makes an outbound-only connection from your PC to Cloudflare.

## What this does (and does not) do

- Works behind NAT/CGNAT and on Wi-Fi. No router configuration needed.
- Hides your home IP address. Gives you HTTPS automatically.
- Lets you put authentication (Cloudflare Access) in front of any service.
- Does **not** make your connection faster. Traffic takes an extra hop through Cloudflare, and your upload speed and Wi-Fi quality still set the ceiling. See [docs/wifi-tips.md](docs/wifi-tips.md).
- Cloudflare's terms restrict using the free tunnel for large video/media streaming; check their current terms before serving heavy media.

## Prerequisites

- A free Cloudflare account.
- A domain whose DNS is managed by Cloudflare (needed for the CLI method; the dashboard method also needs one for public hostnames).
- A service running locally, e.g. `python3 -m http.server 8080`.

## Option A: Dashboard (remotely managed) — easiest

1. Cloudflare dashboard → **Zero Trust** → **Networks** → **Tunnels** → **Create a tunnel** → choose **Cloudflared**.
2. Name it (e.g. `home-pc`), then copy the install command shown for your OS and run it on your PC. It installs `cloudflared` as a background service.
3. Add a **Public hostname**: e.g. `app.example.com` → Service `HTTP` → `localhost:8080`.
4. Visit `https://app.example.com`.

## Option B: CLI (locally managed)

```bash
# 1. Install cloudflared
#    macOS:   brew install cloudflared
#    Windows: winget install Cloudflare.cloudflared
#    Linux:   see https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/downloads/

# 2. Authenticate (opens browser; pick your domain)
cloudflared tunnel login

# 3. Create the tunnel (prints a UUID and writes a credentials JSON)
cloudflared tunnel create home-pc

# 4. Create config (see examples/config.example.yml), then route DNS
cloudflared tunnel route dns home-pc app.example.com

# 5. Test run
cloudflared tunnel run home-pc

# 6. Run at boot
sudo cloudflared service install        # Linux/macOS
# Windows (admin PowerShell): cloudflared service install
```

Config lives at `~/.cloudflared/config.yml` (Linux/macOS) or `%USERPROFILE%\.cloudflared\config.yml` (Windows). Copy [examples/config.example.yml](examples/config.example.yml) and fill in your values.

## Exposing SSH / RDP / other TCP services

Publish the hostname with `service: ssh://localhost:22` (or `rdp://localhost:3389`), then protect it with **Cloudflare Access**. On the client machine:

```bash
# ~/.ssh/config
Host home-pc
  HostName ssh.example.com
  ProxyCommand cloudflared access ssh --hostname %h
```

## Security checklist

- Put **Cloudflare Access** (Zero Trust → Access → Applications) in front of anything that is not meant to be public. Free for up to 50 users.
- Never commit `~/.cloudflared/*.json` or tunnel tokens. This repo's `.gitignore` blocks them.
- Bind local services to `127.0.0.1`, not `0.0.0.0`; the tunnel reaches them via localhost.
- Keep the OS and `cloudflared` updated (`cloudflared update` or your package manager).
- Use a strong password / SSH keys only; disable password SSH login.

## Troubleshooting

| Symptom | Check |
| --- | --- |
| 502 Bad Gateway | Local service not running, or wrong port/scheme in `service:` |
| 1033 / tunnel not found | `cloudflared` not running; check `cloudflared tunnel info home-pc` |
| DNS not resolving | Hostname must be proxied (orange cloud) CNAME to `<UUID>.cfargotunnel.com` |
| Dies when PC sleeps | A sleeping PC is offline; disable sleep or accept downtime |
| Logs | `cloudflared tunnel --loglevel debug run home-pc` |
