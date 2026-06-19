# bruzl-plugins

Public Claude plugin marketplace for [**Bruzl**](https://bruzl.de) — a recipe web app.

The `bruzl` plugin bundles two things into one installable unit:

- **The Bruzl MCP server** (`https://bruzl.de/api/mcp`) — programmatic access to your cookbook (the *what*).
- **The `recipe-import` skill** — the procedural know-how for turning a URL, pasted text, or a photo/PDF into a correctly-formatted German Bruzl recipe and saving it (the *how*).

Everything here is non-sensitive: the MCP URL is public and login-gated (RLS-scoped), and the bundled OAuth client is a **public PKCE client with no secret**.

## Install

```
/plugin marketplace add lksrpp/bruzl-plugins
/plugin install bruzl@bruzl-plugins
```

On first connect, Claude Code runs OAuth discovery against `bruzl.de/api/mcp` (Supabase as the authorization server) and signs you into your Bruzl account. No manual client-id entry, no secret.

## Standalone MCP (non-plugin clients)

Point any MCP client at `https://bruzl.de/api/mcp` and supply the public `client_id` (DCR stays off).

## Layout

```
.claude-plugin/marketplace.json   # marketplace entry → ./plugin
plugin/
├── .claude-plugin/plugin.json    # plugin manifest
├── .mcp.json                     # the Bruzl MCP server (public PKCE OAuth client)
└── skills/
    └── recipe-import/            # the import skill (SKILL.md + references/)
```

## Status

🚧 **Work in progress (v0.1.0).** Scaffold + manifests are in place; the `recipe-import` skill content and the real OAuth `client_id` are being filled in.
