# Project skills

These skills load automatically for anyone who opens this repository in Claude Code.

## mcp-server-dev (vendored)

`build-mcp-server`, `build-mcp-app` and `build-mcpb` are copied unchanged from Anthropic's
[`mcp-server-dev`](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/mcp-server-dev)
plugin in `anthropics/claude-plugins-official`, at commit
`6985b2d88dc08b4a0d202f5f0301f02e360b9cb1` (2026-11-24). They are licensed under Apache 2.0
(see `LICENSE` in each skill folder).

Start with `build-mcp-server`; it hands off to the other two.

To update, copy `plugins/mcp-server-dev/skills/*` from a newer commit of that repo over these
folders and update the commit hash above.

**Known difference:** `build-mcp-server/references/deploy-cloudflare-workers.md` still uses Cloudflare's
`McpAgent` class, which Cloudflare has since deprecated. The bootcamp steps use the stateless
`createMcpHandler` instead, and their prompts say so; follow the step prompts where they differ.
