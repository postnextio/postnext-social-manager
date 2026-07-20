# postnext-social-manager

A Claude Agent Skill for managing social media through [PostNext](https://postnext.io). It lets an agent connect-aware, upload media, compose, schedule, and publish posts across Twitter/X, Instagram, LinkedIn, Threads, YouTube, TikTok, and Bluesky, and read analytics - all via the PostNext public API with a single API key.

## Install

Copy this directory into your skills location (for Claude Code, a `skills/` directory the harness scans), then set your key:

```bash
export POSTNEXT_API_KEY=<your key>   # create at https://postnext.io/account/api-keys
```

The bundled `postnext` helper needs `curl` and `jq`. In an environment with no shell (for example claude.ai), the skill still works: every operation has a raw curl recipe in `references/`.

## Quick start

```bash
./postnext channels                                   # what is connected
./postnext post --provider twitter --text "Hello" --media ./hero.png
./postnext schedule --provider instagram --text "Launch" --media ./a.jpg --at 2026-07-25T14:00:00Z
./postnext analytics best-time twitter
```

Run `./postnext help` for the full command list.

## What is in here

| Path | What |
|------|------|
| `SKILL.md` | Skill entry point: setup, the core loop, and a router to the references. |
| `postnext` | Bash helper (curl + jq) that handles the API's error-prone parts. |
| `references/` | Per-area docs with raw curl recipes: posts, media, channels, analytics, errors. |
| `docs/` | Design spec. |

## Scope

Covers the PostNext social publishing and analytics loop. Out of scope (use the PostNext MCP at `mcp.postnext.io` or the web app): connecting channels, WordPress/blog posts, brand-profile edits, and team management.

## License

MIT. See `LICENSE`. Not affiliated with post-bridge; inspired by the shape of its social-manager skill.
