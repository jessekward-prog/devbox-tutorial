# Deploy (Static Site or Node App) To Coolify

End-to-end recipe for getting a repo live on your Coolify instance, fronted by your own domain via a Cloudflare Tunnel.

Pairs with: [[Coolify-API]], [[Cloudflared-Config]], [[Add-Cloudflare-Subdomain]].

## 1. Prep the repo

Files Coolify expects to find:
- `Dockerfile` — slim base image, install deps, `EXPOSE <port>`, `CMD [...]`
- `.gitignore` — `node_modules`, `.env`, logs
- **No secrets committed.** Env vars go into Coolify directly, never into the repo.

## 2. Push the repo

A public repo is simplest to start with — no deploy keys needed:
```bash
gh repo create <repo-name> --public --source=. --remote=origin --push
```
Private repos need a per-repo SSH deploy key — see the gotcha at the bottom before reaching for `/applications/public` on a private repo.

## 3. Create the project + application

```
POST /api/v1/projects { "name": "<project-name>" }
# → returns project uuid

POST /api/v1/applications/public {
  "project_uuid": "<from above>",
  "server_uuid": "<your Coolify server uuid>",
  "environment_name": "production",
  "git_repository": "https://github.com/<owner>/<repo>",
  "git_branch": "main",
  "build_pack": "dockerfile",
  "ports_exposes": "<container-port>",
  "ports_mappings": "<host-port>:<container-port>",
  "domains": "https://<subdomain>.<your-domain.com>",
  "instant_deploy": false
}
```

## 4. Deploy

```
POST /api/v1/deploy?uuid=<app_uuid>&force=true
```

**`instant_deploy: true` on the create call is unreliable** — the app gets created but the build often never fires. Always follow up with an explicit `/deploy` call.

Tail progress:
```
GET /api/v1/deployments/{deployment_uuid}
```

## 5. Wire the subdomain

Follow [[Add-Cloudflare-Subdomain]] using the host port from step 3.

## Common failure modes

- **Private repo + `/applications/public` = silent fail.** Build container starts, then `git ls-remote` errors with `could not read Username for 'https://github.com'`. Either flip the repo public, or mint a deploy key and use `/applications/private-deploy-key` instead.
- **`POST .../envs` does not upsert.** Calling it twice on the same key creates duplicates with non-deterministic precedence. Use `PATCH .../envs/{env_uuid}` to update an existing one.
- **App boot-loops with "server does not support SSL connections".** If it talks to a Coolify-managed Postgres, append `?sslmode=disable` to the connection string.
- **`/applications/{uuid}/restart` and `/start` only relaunch the cached image** — they don't pull new code. Use `/deploy` for that.
