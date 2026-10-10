# Postbase Agent Skills

Skills that teach coding agents (Claude Code, Cursor, Codex and others) to
draft, schedule, check and fix social media posts through the
[Postbase](https://www.postbase.so) REST API.

```bash
npx skills add postbasehq/skills
```

Then create an API key in Postbase → Developers and make it available to your
agent as `POSTBASE_API_KEY`.

| Skill | What it covers |
| ----- | -------------- |
| `postbase-post` | Draft or schedule posts and threads, per-channel text, safe retries with idempotency keys |
| `postbase-manage-posts` | Check whether a post went out, edit, reschedule, retry failed channels, cancel |
| `postbase-media` | Attach images and video from the media library or from public links |

Try: *"Schedule a LinkedIn post and a shorter X post for tomorrow at 9am announcing this release."*

## Skills or MCP?

Use these skills when your agent works from a terminal or repo and can make HTTP
calls. If your AI tool supports MCP (Claude, ChatGPT, Cursor), the hosted server
at `https://mcp.postbase.so/mcp` gives it the same actions as tools, with
sign-in instead of an API key. See the [MCP docs](https://docs.postbase.so/mcp/overview).

## Docs

- [REST API](https://docs.postbase.so/api/authentication)
- [MCP server](https://docs.postbase.so/mcp/overview)

AGPL-3.0, like Postbase.
