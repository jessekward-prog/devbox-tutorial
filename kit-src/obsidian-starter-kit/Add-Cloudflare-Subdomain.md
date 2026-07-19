# Add Cloudflare Subdomain

Wire a new `*.<your-domain.com>` hostname to a host port through your Cloudflare Tunnel.

Pairs with: [[Cloudflared-Config]], [[Deploy-Static-Site-To-Coolify]].

## Inputs

- `<subdomain>` — the hostname you want, e.g. `app.yourdomain.com`. **Decide this before running anything** — DNS records go live immediately and are awkward to undo cleanly.
- `<host_port>` — the host-side port from your Coolify port mapping (e.g. `3010` from `3010:3000`).
- `<tunnel-id>` — from `cloudflared tunnel list`.

## Steps

**1. Create the DNS record** (no sudo needed — uses your local `~/.cloudflared/cert.pem`):
```bash
cloudflared tunnel route dns <tunnel-id> <subdomain>
```

**2. Edit the tunnel config — safe-edit pattern** (don't hand-edit a shared config with `sudo nano`; one fat-finger takes down every other app behind this tunnel):
```bash
sudo cp /etc/cloudflared/config.yml /tmp/cf-current
cp /tmp/cf-current /tmp/cf-new
# now edit /tmp/cf-new — insert the new ingress block above the http_status:404 catch-all
sudo diff -u /etc/cloudflared/config.yml /tmp/cf-new
# review the diff, then apply:
sudo cp /etc/cloudflared/config.yml /etc/cloudflared/config.yml.bak.$(date +%s) \
  && sudo mv /tmp/cf-new /etc/cloudflared/config.yml \
  && sudo systemctl restart cloudflared
```

New ingress block — **must be above** the catch-all rule:
```yaml
  - hostname: <subdomain>
    service: http://localhost:<host_port>
```

**3. Validate before restarting (cheap insurance):**
```bash
cloudflared tunnel --config /etc/cloudflared/config.yml ingress validate
```

**4. Test:**
```bash
curl -I https://<subdomain>
```

## Gotchas

- **`systemctl reload cloudflared` isn't supported** on the systemd unit — always `restart`. Expect a few seconds of blip across *every* hostname on this tunnel, not just the one you're adding.
- **The real config is `/etc/cloudflared/config.yml`, not `~/.cloudflared/config.yml`.** Editing the home-dir copy does nothing to the running service.
- **Flag order matters for validation, and a wrong order still exits 0.** It's `cloudflared tunnel --config FILE ingress validate` (flag before the subcommand). `... ingress validate --config FILE` prints a usage error but still exits `0`, so a `&&` chain sails right past a validation that never ran.
- Coolify may also generate Traefik labels for the same hostname — harmless, since the tunnel bypasses Traefik entirely and hits the host port directly.

## Rollback

```bash
sudo mv /etc/cloudflared/config.yml.bak.<timestamp> /etc/cloudflared/config.yml \
  && sudo systemctl restart cloudflared
```
