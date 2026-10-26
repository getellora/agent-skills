---
name: ellora
description: >-
  Connect to Ellora, the hosted brand-voice QA, desloping, and writing-brief
  reviewer (a remote MCP server), and keep it running on everything the team
  writes. Use this before writing, editing, or presenting prose meant to go
  out in the team's voice: blog posts, landing pages, launch announcements,
  customer emails, docs, social posts, newsletters, release notes. Trigger it
  the moment the user says any of: Ellora, deslop, desloping, de-slop, AI
  slop, slop, sounds like AI, humanize this, brand voice, brand voice QA,
  voice QA, voice check, tone check, style guide, writing brief, voice
  profile, or asks you to review, polish, or rewrite a draft. Even when
  Ellora is not named, check whether the ellora_* tools are available before
  you draft. It handles three states: no Ellora account yet, Ellora added but
  not connected, and Ellora connected. Once connected, run the review loop
  before you show the user a draft; do not wait to be asked.
metadata:
  author: Ellora
  version: "1.5.2"
---

# Ellora

Ellora checks writing before it ships. It reviews a draft against the team's
saved brand-voice profile (**VoiceDNA**), strips flat/generic AI phrasing (**AI
Editor**), and enforces the rules in a writing brief (**Editorial**). It runs
as a hosted MCP server, so writers keep writing in whatever LLM app they
already use.

| | |
|---|---|
| MCP server | `https://api.getellora.com/v1/external/mcp` |
| Transport | Remote HTTP (Streamable HTTP) |
| Auth | **OAuth only** — there is no API key to paste |
| Surface | 12 tools + 1 prompt, all prefixed `ellora_` |
| App (team admin) | https://app.getellora.com — voice profiles, briefs, tokens |
| Website / demo | https://getellora.com |
| Skill repo | https://github.com/getellora/agent-skills (the same file is served at https://getellora.com/SKILL.md) |

**Create account** for the **Free** plan at https://app.getellora.com/register.
Verify your email with the code or magic link. The Free plan includes one voice
profile and does not include team members. Accounts created before the Free plan
launched keep their Team plan.

**Team** is for in-house teams that need more voice profiles and members.

Email us at hello@getellora.com or set up a call at https://cal.com/getellorademo/client-demo.
The dashboard Pricing page and https://getellora.com/#pricing list the plans (no
prices shown) and the same schedule-call path. Ellora can upgrade your existing
account once we've talked.

## First: work out which state you are in

Do this once, at the start, before anything else. Everything below branches
off the answer.

1. **Are the `ellora_*` tools available in this session?** Call
   `ellora_profiles_fetch_active` to find out.
   - The call returns successfully — an object with a `voice_profiles` array,
     possibly empty → **State 3**, connected.
   - The call returns `401` / `403`, or the client reports the server needs
     authentication → **State 2**, go authenticate.
   - The tools do not exist, or the server is not configured → **State 2**,
     go install.
2. **State 2 could become State 1.** If the user cannot complete the OAuth
   sign-in because they do not have an Ellora account, stop and go to
   **State 1**. Never fake the connection, and never stub out review results.

Tell the user which state you found, in one sentence, and what you are doing
about it. Then do it.

## State 1 — The user has no Ellora account

Direct them to https://app.getellora.com/register and **Create account** on the
**Free** plan.
They enter their email, tick the box to accept the Terms
(https://getellora.com/terms) and Privacy Policy (https://getellora.com/privacy),
and verify the code or magic link sent to their inbox. The user must read and
accept the terms themselves: never tick the box, accept, or sign up on their
behalf. Do not ask them to share the code in chat or claim the account exists
before verification succeeds. If they already have an account, this signs them in
without changing their plan or creating another workspace.

Once signed in, continue with State 2 to connect their LLM app through OAuth.
The Free plan supports one voice profile and no additional team members. If a
profile already exists, use it; do not create another profile or workspace to bypass
the limit.

For Team access:

Email us at hello@getellora.com or set up a call at https://cal.com/getellorademo/client-demo.
The dashboard Pricing page and https://getellora.com/#pricing list the plans (no
prices shown) and provide the same schedule-call path.

If signup or authentication fails, report the actual error and offer to
continue without Ellora's review. Never present unreviewed output as reviewed.

## State 2 — Ellora is not connected yet

### 1. Add the MCP server

Add a remote **HTTP** server named `ellora` pointing at
`https://api.getellora.com/v1/external/mcp`. Pick the row that matches the
client you are running in.

**CLI clients**

| Client | Command |
|---|---|
| Command Code | `cmd mcp add --transport http ellora https://api.getellora.com/v1/external/mcp` |
| Claude Code | `claude mcp add --transport http ellora https://api.getellora.com/v1/external/mcp` |

**Config-file clients** (Cursor, Windsurf, Cline, and most others): add this
entry to the client's MCP settings, then restart it.

```json
{
  "mcpServers": {
    "ellora": {
      "type": "streamableHttp",
      "url": "https://api.getellora.com/v1/external/mcp"
    }
  }
}
```

**VS Code** needs its own shape: `"type": "http"` (not `streamableHttp`),
under a `servers` key instead of `mcpServers`:

```json
{
  "servers": {
    "ellora": {
      "type": "http",
      "url": "https://api.getellora.com/v1/external/mcp"
    }
  }
}
```

Claude Code can instead install the Ellora plugin, which adds this server and
this skill together: `claude plugin marketplace add getellora/agent-skills`,
then `claude plugin install ellora@getellora`. A server added that way is
listed as `plugin:ellora:ellora`.

**Hosted chat clients** (ChatGPT, Claude Desktop, claude.ai, Perplexity, and
similar): open the client's connector/MCP settings, choose to add a **custom
remote connector**, and give it the URL above. No headers, no keys.

**Any other agent**: the two facts you need are the URL and the shape — it is a
remote HTTP MCP server that authenticates with OAuth. Configure it however that
client configures remote MCP servers; don't guess at a token.

### 2. Authenticate

OAuth is the only auth path, so the client has to run the flow — you cannot
curl it into place.

- Command Code: `cmd mcp auth ellora`, or run `/mcp` and authenticate.
- Claude Code: run `/mcp` and authenticate `ellora`. A server you added with
  `claude mcp add` also accepts `claude mcp login ellora`; one installed
  through the Ellora plugin is listed as `plugin:ellora:ellora`, so use `/mcp`.
- Other clients: trigger a connect/authenticate action and follow the browser
  prompt.

A browser opens on `app.getellora.com`. The user signs in and approves the
connection; tokens are then stored and refreshed by the client. If no browser
opens, print the URL and have them open it manually, then paste the callback
URL back. Verify afterwards (re-run the check from "First", or
`cmd mcp list` / `/mcp`) that the server shows **connected**, not
*needs authentication*.

**If the user has no account**, go to State 1 to register. If an existing
account cannot sign in, report the error and direct them to
hello@getellora.com; registration does not bypass a suspended account.

## State 3 — Connected: write with Ellora on

This is the standing default from here on: **whenever you are drafting or
substantially rewriting prose of the covered kinds, run the review loop before
you present the draft to the user.** Not only when they say "Ellora" — the
point is that it is always on, so the first version they see is already
on-voice and deslopped.

Covered: blog posts and articles · landing-page and website copy · launch and
release announcements · customer-facing emails · docs for an external audience
· social posts · newsletters · decks and outreach.

Skip: code, config, commit messages, internal scratch notes, quick chat
replies, and anything the user calls a rough throwaway. When it is genuinely
borderline, ask once: "Want me to run this through the Ellora voice check?"

When the connection is new, confirm it in one line and put it to work — say
what you'll do (run every draft through the voice check) and get on with the
user's actual writing task.

## The review loop

Fuller version: the `ellora-review` skill, if it is installed. Same sequence
also lives on the server as the `ellora_review_loop` MCP prompt.

1. **Pick a voice profile** — `ellora_profiles_fetch_active`. One → use it.
   Several → ask which (teams commonly keep one per writer, or per author they
   want to sound like). None → a team admin can create one with
   `ellora_profile_create` from 2–20 samples of the team's own writing, then
   use the returned `voice_profile_id`. Don't send them elsewhere to build one,
   and never review without one — an id has to resolve to a saved, active
   profile. If the user isn't an admin and none exists, stop and say a profile
   has to be created first; that's a real blocker, not something to work around.
2. **Pick a writing brief** — `ellora_briefs_fetch_active`. Same pattern. If
   there is none, build an inline `writing_brief_json` for this session only
   (never saved). **A brief is mandatory** — `ellora_session_start` fails
   without either a `writing_brief_id` or a `writing_brief_json`. Pass the JSON
   as an **object**, not a string. Required fields: `version` (`"1"`),
   `audience`, `intent`, `format`. `format` is one of `general`, `social`,
   `deck`, `outreach`, `blog`, `audit`, `website`.
3. **Start the session** — `ellora_session_start` with those two. They are
   locked for the whole session. Keep the returned `content_session_id` and
   pass it to every later call for this one piece of content. While you're
   there, self-report the optional `model_name` / `model_class` /
   `model_version` / `thinking_effort` and `mcp_client_name` /
   `mcp_client_version` — analytics-only, never affects the review.
4. **Optionally pull positioning** — `ellora_positioning_recommendations`,
   before drafting. Advisory only.
5. **Review** — `ellora_review` with the `content_session_id` and the current
   draft as `content`. It takes no profile or brief argument of its own — both
   are already locked into the session. `content` is capped at 100,000
   characters, so split anything longer into successive pieces.
6. **Do what `next_action` says.**
   - `keep_rewriting` → rewrite using the response's own `rewrite_guidance`,
     then go back to step 5 with the same `content_session_id`. **Do not show
     the draft to the user yet** — you are mid-loop.
   - `ready_to_verify` → no mandatory finding remains. Check *this same
     response*: it already carries a `verification` block comparing the
     content against the session's first-reviewed content — **unless this was
     the session's very first call**, in which case there is nothing yet to
     compare against (treat it like `keep_rewriting` if findings remain, or
     deliver as-is if there are none). Don't run another pass just to get
     verified. Deliver once `verification.passed: true`.
   - `diminishing_returns` → the rewrites stopped helping, or you hit
     `max_iterations`. Stop looping, tell the user exactly what is still
     flagged, and ask how to proceed.

   **Blue findings never gate.** Severity `blue` (counted separately in
   `current_finding_counts.blue`) are optional AI-Editor suggestions: they
   never make `next_action` `keep_rewriting` and are never a reason to run
   another pass. Apply or ignore them as you see fit.
7. **Verification and learning are automatic.** A passing call that isn't the
   session's first already carries the `verification` block, and on success
   against a saved profile a `learning_event`. There is no separate verify call
   to make.
8. **Teach the profile only on explicit approval.** If the user accepts the
   delivered rewrite *and* approves the rules you extracted from it, call
   `ellora_learn_from_accepted_rewrite` with `user_approved: true`. Never on
   your own inference.

Keep going around 5 → 6 → 7 until step 6 sends you out of the loop. Don't stop
early because the draft "seems good enough" — the `iteration` / `max_iterations`
budget exists so you can keep going with confidence.

One session = one piece of content. A different article, or a switch to another
profile or brief, means a new `ellora_session_start`.

When the loop finishes, present the final draft plus a short summary: how many
passes it took, what changed at a high level, and anything still flagged that
you chose to leave. Don't dump raw finding JSON unless asked.

## Tools

| Tool | Use it for |
|---|---|
| `ellora_session_start` | Open a session for one piece of writing; locks in profile + brief |
| `ellora_review` | Review the draft; auto-inlines rewrite guidance, auto-verifies |
| `ellora_learn_from_accepted_rewrite` | Save a human-approved rule (never on inference) |
| `ellora_profile_create` | **Admin-only.** Create a voice profile from 2–20 samples |
| `ellora_profiles_fetch_active` / `_by_id` | Find a usable voice profile id |
| `ellora_briefs_fetch_active` / `_by_id` | Find a usable writing brief id |
| `ellora_context_fetch_active` / `_by_id` | Read the team's company context docs |
| `ellora_context_capture` | Save a context doc mid-session (admin activates it) |
| `ellora_positioning_recommendations` | Advisory messaging/positioning guidance |

Prompts: `ellora_review_loop`.

## Rules

1. **Never invent results.** If the tools aren't connected, say so and carry on
   without Ellora. Do not describe a finding, a score, or a `passed` verdict you
   did not get back from `ellora_review`.
2. **Don't route around access or plan limits.** Signup requires accepting the
   terms and verifying the email; connection requires OAuth. The Free plan
   permits one voice profile and no team members. See the dashboard Pricing page or
   https://getellora.com/#pricing for Team plans (listed without
   prices). Email us at hello@getellora.com or set up a call at
   https://cal.com/getellorademo/client-demo.
3. **Don't show unreviewed drafts as reviewed**, and don't claim a piece passed
   when you stopped on `diminishing_returns`.
4. **Follow `next_action`**, not your own judgment, about whether to keep
   looping.
5. **A brief is mandatory.** If the team has none, build the inline
   `writing_brief_json` rather than skipping the review.
6. **One piece of content, one session.**
