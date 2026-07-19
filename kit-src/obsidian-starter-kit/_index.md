# Self-Hosted Dev Workflow — Starter Playbooks

Drop this folder into your Obsidian vault (or anywhere Claude Code / your editor's AI can read it). It's the same pattern used to build the headless dev box tutorial this pack came with: a small library of procedures your AI assistant reads *before* doing repeatable infra work, so you stop re-explaining your own stack every session.

## How to wire it up

Add this to your project's `CLAUDE.md` (or your global `~/.claude/CLAUDE.md` if you want it available everywhere):

> Obsidian vault at `<path-to-your-vault>` is my playbook library — procedures and recipes that have actually worked. Before doing repeatable infra work (deploying to Coolify, editing a Cloudflare Tunnel, spinning up a new app), open `<path-to-your-vault>/_index.md` and follow the matching playbook.

That's the whole trick. From then on, instead of typing out your Coolify token location, your tunnel ID, and your gotchas every single time, your assistant just reads the file.

## Playbooks in this pack

- [[Deploy-Static-Site-To-Coolify]] — get a repo live on Coolify behind your own domain
- [[Add-Cloudflare-Subdomain]] — wire a new subdomain to a host port through a Cloudflare Tunnel
- [[Coolify-API]] — API auth, endpoints, and the gotchas that cost real time
- [[Cloudflared-Config]] — tunnel config file locations, restart vs reload, validation

## Conventions

- Fill in every `<placeholder>` with your own values before first use. Nothing in this pack has real credentials, tokens, or IDs in it.
- One pattern per heading, rule in bold, short "why" underneath.
- When you hit a new gotcha, add it as a heading in the matching file instead of starting a new note — this pack is meant to grow with you, not stay frozen.
