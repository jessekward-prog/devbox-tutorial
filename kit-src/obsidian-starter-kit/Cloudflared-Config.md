# Cloudflared Tunnel Config — Reference

## File locations

- **`/etc/cloudflared/config.yml`** — this is what the systemd service actually reads.
- `~/.cloudflared/config.yml` — often stale. `cloudflared tunnel route dns` uses the certificate that lives alongside it, but the *running tunnel* does not read this copy.

Editing the home-dir copy and wondering why nothing changed is the single most common mistake here — always confirm you're editing `/etc/cloudflared/config.yml`.

## Routing pattern

One ingress rule per hostname, routed straight to a host port — not through a reverse proxy like Traefik or nginx:

```yaml
tunnel: <tunnel-id>
credentials-file: /home/<user>/.cloudflared/<tunnel-id>.json

ingress:
  - hostname: app.yourdomain.com
    service: http://localhost:3000
  - service: http_status:404
```

The catch-all `http_status:404` rule must always be **last** — anything below it never matches.

## Restart, not reload

`systemctl reload cloudflared` isn't wired up on the systemd unit. Use `restart` instead — it's a few seconds of blip across *every* hostname on this tunnel, not just the one you're changing, so batch multiple hostname additions into one edit when you can.

## Validate before restarting

```bash
cloudflared tunnel --config /etc/cloudflared/config.yml ingress validate
```

Flag order matters: `ingress validate --config FILE` (config *after* the subcommand) silently no-ops and still exits `0` — so a `&&`-chained script can look like it validated when it didn't. Run validation as its own step and actually read the output.

## Common failure modes

- App shows Cloudflare's "502 / origin unregistered" on every subdomain at once, not just one → check for a crash-looping container elsewhere on the box (`docker ps -a` for `Restarting` status). Rapid container restarts can destabilize the host's networking enough to periodically drop the tunnel's edge connections entirely.
- New hostname 404s → the ingress rule is below the `http_status:404` catch-all, or the restart didn't actually happen.
