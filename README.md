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

### Any agent that reads skills from GitHub

```bash
npx skills add getellora/agent-skills
```

This installs both skills into every supported agent it finds on your machine, including Claude Code, Cursor, Codex, Gemini CLI, Windsurf, Grok, and GitHub Copilot. Add `-g` for a user-wide install, or `--skill ellora` for the front-door skill only.

### Claude Code plugin (skills plus the MCP server)

```bash
claude plugin marketplace add getellora/agent-skills
claude plugin install ellora@getellora
claude mcp login ellora
```

The plugin registers the `ellora` MCP server and both skills. The last command runs the OAuth sign-in in your browser.

### Claude Code, skills only

```bash
claude mcp add --transport http ellora https://api.getellora.com/v1/external/mcp
claude mcp login ellora
```

Then copy `skills/ellora` into `~/.claude/skills/` (user scope) or `.claude/skills/` (project scope).

### Cursor, Windsurf, Cline, Codex, and other config-file clients

1. Add the MCP server to the client's MCP settings and restart the client:

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

2. Copy `skills/ellora` into the client's skills directory: `.cursor/skills/`, `.agents/skills/`, `~/.codex/skills/`, or the equivalent.

3. Complete the OAuth prompt the client shows on first use.

### Hosted chat clients (ChatGPT, claude.ai, Claude Desktop, Perplexity)

Add a custom remote connector with the server URL above. Then paste the contents of `skills/ellora/SKILL.md` into the project or system instructions, because these clients do not load skill files.

### From the URL

Tell your agent: `set up https://getellora.com/SKILL.md`. The agent fetches the skill, saves it, and connects.

## What the skill does

Once loaded, the agent:

1. Checks whether the `ellora_*` tools are available in the session.
2. If they are not, walks the user through adding the server and signing in, or registering.
3. Once connected, runs the review loop before it shows any draft: pick a voice profile, pick a writing brief, start a session, review, rewrite from the returned guidance, and repeat until verification passes.

The skill triggers on mentions of Ellora, deslop or AI slop, brand voice QA, voice or tone checks, style guides, writing briefs, voice profiles, and requests to review or rewrite a draft.

## Keeping the copies in sync

`skills/ellora/SKILL.md` must match `public/SKILL.md` in the `ellora-website` repository, which is what https://getellora.com/SKILL.md serves. Change one, change the other in the same pass, and bump `metadata.version` in both.

## Links

- Website and demo: https://getellora.com
- App: https://app.getellora.com
- Questions: hello@getellora.com

## License

MIT. See [LICENSE](LICENSE).
