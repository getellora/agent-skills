# Ellora agent skills

Ellora checks writing before it ships. It reviews a draft against the team's saved brand-voice profile (VoiceDNA), strips flat, generic AI phrasing (AI Editor), and enforces the rules in a writing brief (Editorial). It runs as a hosted MCP server, so the agent you already use does the writing and Ellora does the review.

This repository holds the skills that teach an agent to connect to Ellora and run that review loop on every draft. The front-door skill is the same file served at https://getellora.com/SKILL.md.

| | |
|---|---|
| MCP server | `https://api.getellora.com/v1/external/mcp` (remote HTTP, OAuth, no API key) |
| Skills | [`skills/ellora`](skills/ellora/SKILL.md) connects and runs the loop. [`skills/ellora-review`](skills/ellora-review/SKILL.md) is the full loop procedure. |
| Claude Code plugin | marketplace `getellora`, plugin `ellora`. Bundles the MCP server and both skills. |
| Account | https://app.getellora.com/register (free Solo plan) |

## Install

Every path below needs a free Solo account at https://app.getellora.com/register. Sign up with your email, accept the terms, and enter the code you receive. The first time your client calls Ellora, it opens a browser window where you sign in and approve the connection. There is no API key.

### Claude Code: plugin with skills and the MCP server

```bash
claude plugin marketplace add getellora/agent-skills
claude plugin install ellora@getellora
```

The plugin registers the MCP server as `plugin:ellora:ellora` and loads both skills. Run `/mcp` inside Claude Code and authenticate that server.

### claude.ai, Claude Desktop, and Cowork

- Plugin: open **Customize > Plugins > Add > Add marketplace**, enter `getellora/agent-skills`, and install **Ellora**. Then connect Ellora from the plugin's **Connectors** tab.
- Connector only: open **Customize > Connectors > Add custom connector** and enter `https://api.getellora.com/v1/external/mcp`.

### Cursor

Open this link to add the server, or add the JSON entry below to `~/.cursor/mcp.json`:

`cursor://anysphere.cursor-deeplink/mcp/install?name=ellora&config=eyJ1cmwiOiJodHRwczovL2FwaS5nZXRlbGxvcmEuY29tL3YxL2V4dGVybmFsL21jcCJ9`

### VS Code

Open this link, or run **MCP: Add Server** and choose HTTP:

`vscode:mcp/install?%7B%22name%22%3A%22ellora%22%2C%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A//api.getellora.com/v1/external/mcp%22%7D`

### Codex

```bash
codex mcp add ellora --url https://api.getellora.com/v1/external/mcp
codex mcp login ellora
```

### Grok

- Grok Build reads Claude Code marketplaces, so the Claude Code commands above also work there. To add only the server: `grok mcp add --transport http ellora https://api.getellora.com/v1/external/mcp`.
- grok.com: open https://grok.com/connectors, select **New Connector > Custom**, and enter the server URL.

### Gemini CLI

```bash
gemini mcp add --transport http ellora https://api.getellora.com/v1/external/mcp
```

Then run `/mcp auth ellora` inside Gemini CLI.

### ChatGPT

Until the ChatGPT directory listing is live, add Ellora as a custom connector with the server URL. Your workspace may need developer mode turned on for custom connectors.

### Any other MCP client

Add a remote HTTP (Streamable HTTP) server named `ellora`:

```json
{
  "mcpServers": {
    "ellora": {
      "type": "http",
      "url": "https://api.getellora.com/v1/external/mcp"
    }
  }
}
```

VS Code uses the same entry under a `servers` key instead of `mcpServers`.

### Skills only, for any agent that reads skills from GitHub

```bash
npx skills add getellora/agent-skills
```

This installs both skills into every supported agent it finds, including Claude Code, Cursor, Codex, Gemini CLI, Windsurf, Grok, and GitHub Copilot. Add `-g` for a user-wide install, or `--skill ellora` for the front-door skill only. To copy by hand, put `skills/ellora` in `.agents/skills/` (Codex, Cursor, Cline, Gemini CLI, GitHub Copilot), `.claude/skills/` (Claude Code), `.windsurf/skills/`, or `.grok/skills/`.

### From the URL

Tell your agent: `set up https://getellora.com/SKILL.md`. The agent fetches the skill, saves it, and connects.

## Data and privacy

The plugin holds Markdown skills and JSON manifests. It runs no code on your machine and installs no packages.

It connects one remote MCP server, https://api.getellora.com/v1/external/mcp, over HTTPS. You sign in to it with OAuth in your browser, and the token stays with your MCP client.

When you ask for a review, the model sends the draft text and the IDs of your voice profile and writing brief to that server. It also reports the client and model name when it knows them. Ellora stores each draft it reviews and the review result in your own workspace, so you can see them in the app. A passing review also updates the voice profile it used. If you approve it, the model can save writing rules or a context document to your workspace too. Nothing goes to any other host.

Ellora's privacy policy is at https://getellora.com/privacy/ and its terms are at https://getellora.com/terms/.

## Package files

| File | Used by |
|---|---|
| `.claude-plugin/plugin.json`, `.claude-plugin/marketplace.json`, `.mcp.json` | Claude Code, claude.ai, Cowork, Grok Build, and the Claude directory |
| `plugin.json`, `mcp.json`, `assets/` | The ChatGPT plugin directory (Agent Plugins format, with `extensions.com.openai`) |
| `server.json` | The official MCP Registry, published by `.github/workflows/publish-mcp-registry.yml` as `io.github.getellora/ellora` |
| `skills/` | Every client above |

## What the skill does

Once loaded, the agent:

1. Checks whether the `ellora_*` tools are available in the session.
2. If they are not, walks the user through adding the server and signing in, or registering.
3. Once connected, runs the review loop before it shows any draft: pick a voice profile, pick a writing brief, start a session, review, rewrite from the returned guidance, and repeat until verification passes.

The skill triggers on mentions of Ellora, deslop or AI slop, brand voice QA, voice or tone checks, style guides, writing briefs, voice profiles, and requests to review or rewrite a draft.

## Keeping the copies in sync

The copy served at https://getellora.com/SKILL.md comes from the private `ellora-website` repository, and this note is for the maintainers of both. A change to the skill lands in both places in the same pass: `skills/ellora/SKILL.md` here and `public/SKILL.md` there. Bump `metadata.version` in both files and `version` in `.claude-plugin/plugin.json` and `plugin.json`. Claude Code keeps plugin users on the old version until that number changes. When the server's tool surface changes, set `version` in `server.json` to the server's new `#[Version]` and run the registry workflow.

## Links

- Website and demo: https://getellora.com
- App: https://app.getellora.com
- Questions: hello@getellora.com

## License

MIT. See [LICENSE](LICENSE).
