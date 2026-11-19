# Notes for maintainers

These notes moved here from the README. They are for the people who maintain this repository and the website.

## Package files

| File | Used by |
|---|---|
| `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `.mcp.json` | Claude Code, claude.ai, Cowork, Grok Build, and the Claude directory |
| `plugin.json`, `mcp.json`, `assets/icon.png`, `assets/logo.png` | The ChatGPT plugin directory (Agent Plugins format, with `extensions.com.openai`) |
| `server.json` | The official MCP Registry, published by `.github/workflows/publish-mcp-registry.yml` as `io.github.getellora/ellora` |
| `skills/` | Every client |
| `assets/readme/` | The README images only. Each image has a light and a dark file. |
| `assets/logo.svg` | The source drawing for `icon.png` and `logo.png`. No client loads it. |

Every `mcpServers` copy-paste block in the README uses `streamableHttp`, the spelling Cline requires. VS Code is the exception: it needs `http` under a `servers` key. `.mcp.json` keeps `type: http` (the value Claude Code's docs show for this transport). `mcp.json` and `server.json` keep `streamable-http` (the exact value their own schemas require).

## Keep the copies in sync

The copy served at https://getellora.com/SKILL.md comes from the private `ellora-website` repository. A change to the skill lands in both places in the same pass: `skills/ellora/SKILL.md` here and `public/SKILL.md` there. Bump `metadata.version` in both files and `version` in `.claude-plugin/plugin.json` and `plugin.json`. Claude Code keeps plugin users on the old version until that number changes. In `ellora-website`, also update the version in `test/skill-md-frontmatter.test.mjs` and, when the review loop section changes, `REVIEW_LOOP_SHA256` in `test/prw7-skill-review-loop.test.mjs`. When the server's tool surface changes, set `version` in `server.json` to the server's new `#[Version]` and run the registry workflow.

## Contributors

Do not add `Co-authored-by` lines for AI agents to commits on `main`, and remove them from squash-merge messages. GitHub lists every co-author on the repository's contributors list.
