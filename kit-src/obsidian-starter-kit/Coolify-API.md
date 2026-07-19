# Coolify API — Quick Reference

Coolify v4 exposes a full REST API. There's no public OpenAPI spec — the fastest way to learn an endpoint's shape is to `POST` an empty `{}` and read the validation error listing required fields, or `GET` an existing resource and read its field names back.

## Auth

Bearer token from **Coolify dashboard → Profile → Keys & Tokens**. Store it outside your repo — e.g. `~/.config/coolify/token`, mode `600`.

```bash
curl -s http://localhost:8000/api/v1/version \
  -H "Authorization: Bearer $(cat ~/.config/coolify/token)"
```

## Useful endpoints

- `GET /api/v1/version` — smoke test that auth works
- `GET /api/v1/projects`, `/servers`, `/applications`, `/databases`, `/security/keys`
- `POST /api/v1/deploy?uuid=<app_uuid>&force=true` — the real "pull latest, rebuild, deploy" call
- `PATCH /api/v1/applications/{uuid}/envs/{env_uuid}` — update an env var by UUID (matches by `key` in the body)
- `DELETE /api/v1/applications/{uuid}/envs/{env_uuid}` — remove one

## Gotchas

- **`is_build_time` on an env var POST 422s.** Strip it — Coolify rejects it as "not allowed".
- **Env var POSTs don't upsert.** Posting the same key twice creates a duplicate with non-deterministic ordering at runtime. Always `PATCH` by UUID to update; `DELETE` stale duplicates individually.
- **Postgres from Coolify's built-in template rejects SSL.** Append `?sslmode=disable` to any connection string built from `internal_db_url`, or the app boot-loops with "server does not support SSL connections".
- **`instant_deploy: true` on application create is unreliable.** The app record gets created but the build often just never fires. Always follow with an explicit `POST /deploy?uuid=...&force=true`.
- **`/applications/{uuid}/restart` and `/start` only relaunch the cached image** — neither pulls new code. Use `/deploy` when you actually want the latest commit shipped.
