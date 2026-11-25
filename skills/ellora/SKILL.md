---
name: ellora
description: >-
  Connect to Ellora, the hosted editor that reviews drafts for brand voice,
  generic AI phrasing (slop), contradictions and writing-brief rules (a
  remote MCP server). Keep it on for all prose the team writes. Use this
  skill before you write, edit or present prose that goes out in the team's
  voice: blog posts, landing pages, launch announcements, customer emails,
  docs, social posts, newsletters, release notes. Use it when the user says
  any of: Ellora, deslop, desloping, de-slop, AI slop, slop, sounds like AI,
  humanize this, brand voice, brand voice QA, voice QA, voice check, tone
  check, style guide, writing brief, voice profile, or asks you to review,
  polish or rewrite a draft. When the user does not name Ellora, still check
  for the ellora_* tools before you draft. The skill handles three states:
  no Ellora account, Ellora added but not connected, and Ellora connected.
  When Ellora is connected, run the review loop before you show a draft. Do
  not wait for the user to ask.
metadata:
  author: Ellora
  version: "1.7.1"
---

# Ellora

Ellora checks writing before it ships. Each review runs four checks:

- **VoiceDNA** checks that the draft sounds like the team's saved voice profile.
- **AI Editor** finds generic AI phrasing.
- **Narrative** finds contradictions and unsupported leaps.
- **Editorial** checks the draft against the writing brief.

Ellora is a hosted MCP server. The user writes in the AI app they already use.
You write the draft, Ellora reviews it, and you fix what Ellora finds.

| | |
|---|---|
| MCP server | `https://api.getellora.com/v1/external/mcp` |
| Transport | Remote HTTP (Streamable HTTP) |
| Sign-in | **OAuth only**. There is no API key. |
| Tools | 12 tools and 1 prompt. All names start with `ellora_`. |
| App (team admins) | https://app.getellora.com (voice profiles, briefs, tokens) |
| Website and demo | https://getellora.com |
| Skill source | https://github.com/getellora/agent-skills (the same file is at https://getellora.com/SKILL.md) |

## Plans

Every new account starts on the **Free** plan. There is no plan to select at
sign-up. The user creates the account at https://app.getellora.com/register.

The Free plan includes:

- one active voice profile
- one connected AI app (MCP connection)
- 50,000 words checked each calendar month (UTC)
- one review that runs at a time
- no team members

Accounts that existed before the Free plan keep their earlier limits.

The **Team** plan is for in-house teams that need more voice profiles and members.
Email us at hello@getellora.com or set up a call at https://cal.com/getellorademo/client-demo.
The dashboard Pricing page and https://getellora.com/#pricing show the plans,
with no prices. Ellora can upgrade an existing account after we talk.

## Find the state

Do this one time, at the start. All the steps after it depend on the result.

1. Call `ellora_profiles_fetch_active`.
   - The call returns an object with a `voice_profiles` array (the array can
     be empty). Go to **State 3: connected**.
   - The call returns `401` or `403`, or the client says the server needs
     authentication. Go to **State 2** and sign in.
   - The tool does not exist, or the server is not set up. Go to **State 2**
     and add the server.
2. If the user cannot sign in because they have no Ellora account, go to
   **State 1**.

Never fake a connection. Never make up review results.

Tell the user in one sentence which state you found and what you will do.
Then do it.

## State 1: The user has no account

Send the user to https://app.getellora.com/register and tell them to select
**Create account**. The user then:

1. Enters their email address.
2. Accepts the Terms (https://getellora.com/terms/) and the Privacy Policy
   (https://getellora.com/privacy/).
3. Enters the code or opens the magic link from the email.

The user must read and accept the terms. Do not accept the terms, tick the
box or sign up for the user. Do not ask the user to send the code in the chat.
Do not say that the account exists until the user confirms the email code
or magic link. If the user already has an account, this step signs them in.
It does not change their plan and does not make a second workspace.

After sign-in, go to State 2 and connect the AI app.

Do not go around Free plan limits:

- If a voice profile exists, use it. Do not make a second profile or a second
  workspace to get past the limit.
- If Ellora refuses a review because the monthly word limit is reached, tell
  the user when the limit resets. The error message says when. Do not try
  again.

For Team access:
Email us at hello@getellora.com or set up a call at https://cal.com/getellorademo/client-demo.

If sign-up or sign-in fails, tell the user the exact error. Offer to continue
without an Ellora review. Never show a draft that Ellora did not review as
reviewed.

## State 2: Ellora is not connected

### Add the MCP server

Add a remote HTTP server with the name `ellora` and the URL
`https://api.getellora.com/v1/external/mcp`. Use the row for your client.

**Command-line clients**

| Client | Command |
|---|---|
| Command Code | `cmd mcp add --transport http ellora https://api.getellora.com/v1/external/mcp` |
| Claude Code | `claude mcp add --transport http ellora https://api.getellora.com/v1/external/mcp` |

**OpenCode** uses an `mcp` block, not `mcpServers`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "mcp": {
    "ellora": {
      "type": "remote",
      "url": "https://api.getellora.com/v1/external/mcp"
    }
  }
}
```

Put the block in `opencode.json` in the project, or in
`~/.config/opencode/opencode.json` for all projects. Then run
`opencode mcp auth ellora`.

**Gemini CLI** adds the server from the terminal:

```bash
gemini mcp add --transport http ellora https://api.getellora.com/v1/external/mcp
```

**Clients with a config file** (Cursor, Windsurf, Cline and most others): add
this entry to the MCP settings, then restart the client.

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

**VS Code** uses a different shape: `"type": "http"` (not `streamableHttp`),
under a `servers` key (not `mcpServers`):

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

Claude Code can instead install the Ellora plugin, which adds the server and
this skill together. Run `claude plugin marketplace add getellora/agent-skills`,
then `claude plugin install ellora@getellora`. Claude Code lists a server from
the plugin as `plugin:ellora:ellora`.

OpenCode can load this skill from the same repository. Run
`npx skills add getellora/agent-skills -a opencode` (add `-g` to install for
all projects), or copy `skills/ellora` to `.agents/skills/ellora/` or
`~/.config/opencode/skills/ellora/`.

**Hosted chat clients** (ChatGPT, Claude Desktop, claude.ai, Perplexity and
similar): open the connector or MCP settings, add a **custom remote
connector**, and enter the URL above. Do not add headers or keys.

**Other agents:** use the URL above. The server is a remote HTTP MCP server
that uses OAuth. Set it up the way that client sets up remote MCP servers. Do
not guess a token.

### Sign in

The client must run the OAuth sign-in. You cannot do it with curl.

- Command Code: run `cmd mcp auth ellora`, or run `/mcp` and authenticate.
- Claude Code: run `/mcp` and authenticate `ellora`. For a server added with
  `claude mcp add`, `claude mcp login ellora` also works. For a server from
  the plugin, use `/mcp`, because its name is `plugin:ellora:ellora`.
- OpenCode: run `opencode mcp auth ellora`, or run `/mcp` and authenticate.
- Gemini CLI: run `/mcp auth ellora`.
- Other clients: start the connect or authenticate action and follow the
  browser.

A browser opens on `app.getellora.com`. The user signs in and approves the
connection. The client then keeps and refreshes the tokens. If no browser
opens, show the user the URL. The user opens it and pastes the callback URL
back to you.

After sign-in, do the state check again, or run `cmd mcp list`, `/mcp` or
`opencode mcp list`. The server must show **connected**, not *needs
authentication*.

If the user has no account, go to State 1. If an existing account cannot
sign in, tell the user the error and send them to hello@getellora.com. A new
registration does not get around a suspended account.

## State 3: Ellora is connected

From now on, Ellora is on by default. When you write or largely rewrite prose
of the types below, run the review loop before you show the draft. Do this
even when the user does not say "Ellora". The first draft the user sees must
already be reviewed.

Review these: blog posts and articles, landing-page and website copy, launch
and release announcements, customer emails, docs for an external
audience, social posts, newsletters, decks and outreach.

Skip these: code, config, commit messages, internal scratch notes, short
chat replies, and text the user calls a rough draft to throw away. If you are
not sure, ask one time: "Do you want me to run this through the Ellora voice
check?"

When the connection is new, confirm it in one line. Say that you will run
every draft through the voice check. Then continue with the user's writing
task.

## The review loop

The `ellora-review` skill, if installed, gives the full loop. The server also
gives a short version as the `ellora_review_loop` MCP prompt. If the two are
different, follow this skill and `next_action`.

1. **Select a voice profile.** Call `ellora_profiles_fetch_active`.
   - One profile: use it.
   - More than one: ask the user which one. Teams often keep one profile for
     each writer, or for each author they want to sound like.
   - None: a team admin can make one with `ellora_profile_create` from 2 to 20
     samples of the team's own writing. Then use the `voice_profile_id` it
     returns. Do not send the user to another place to make one.

   Never review without a voice profile. The id must belong to a saved,
   active profile. If the user is not an admin and no profile exists, stop.
   Tell the user that an admin must make a profile first. Do not try to get
   around this.
2. **Select a writing brief.** Call `ellora_briefs_fetch_active` and use the
   same steps. If no brief exists, make an inline `writing_brief_json` for
   this session only. Ellora does not save it.

   **A brief is mandatory.** `ellora_session_start` fails without a
   `writing_brief_id` or a `writing_brief_json`. Send the JSON as an
   **object**, not as a string. The required fields are `version` (`"1"`),
   `audience`, `intent` and `format`. `format` is one of `general`, `social`,
   `deck`, `outreach`, `blog`, `audit` or `website`.
3. **Start the session.** Call `ellora_session_start` with the profile and
   the brief. They stay locked for the full session. Keep the
   `content_session_id` it returns, and send it with every later call for
   this piece of content. In the same call, you can also send `model_name`,
   `model_class`, `model_version`, `thinking_effort`, `mcp_client_name` and
   `mcp_client_version`. These fields are optional and for analytics only.
   They do not change the review. The response also carries `voice_extras`:
   the writer's own extra words (hedges, talk markers, asides and the like)
   with how many they usually use in 400 words. Keep their rate when you
   write: do not strip these words out, and do not pile them on. A phrase
   under `signature_phrases` is a preference, never a requirement.
4. **Get positioning (optional).** Before you write, you can call
   `ellora_positioning_recommendations`. Its advice is advisory only.
5. **Review.** Call `ellora_review` with the `content_session_id` and the
   current draft as `content`. Do not send a profile or a brief, because the
   session already has them. `content` can have up to 100,000 characters.
   Split longer text into successive parts.
6. **Read `convergence_recommendation`, then do what `next_action` says.**
   These are two fields. `convergence_recommendation` is `keep_rewriting`,
   `ready_to_verify` or `diminishing_returns`. `next_action` tells you what
   to do next. Each item below is one `convergence_recommendation` value. For
   every value, follow `next_action`.
   - `keep_rewriting`: mandatory findings are open, or a verification failed
     and review rounds are left. The response includes `rewrite_guidance`. Go to
     step 7. **Do not show the draft to the user yet.**
   - `ready_to_verify`: no mandatory findings are open. On the first review
     of the session, when no judgment-required finding is open, `next_action`
     tells you to deliver the draft as it is. On later reviews, the response
     includes `verification`. Deliver the draft when `verification.passed` is
     `true`. If a judgment-required finding is open, `next_action` names it.
     Get a decision from a person before you deliver.
   - `diminishing_returns`: the mandatory findings stopped going down, you
     reached `max_iterations`, or the verification failed on the last
     allowed review round. Stop the loop. Tell the user what is still wrong and ask
     what to do.

   **Blue findings never block.** They are optional suggestions, counted in
   `current_finding_counts.blue`. They never change
   `convergence_recommendation`, are never in the edit scope of
   `rewrite_guidance`, and never need another review round by themselves. On a
   `ready_to_verify` review, `next_action` also counts the open blue
   suggestions. You can apply them or not.
7. **Rewrite, then review again.** When the response includes
   `rewrite_guidance`, use it to rewrite the content. Then **go back to step
   5** with the new text and the same `content_session_id`. Send all of the
   revised content, not only the changed sentences.
   - The **edit scope** covers only the mandatory red and yellow findings.
     (`rewrite_guidance` also has a blue section.) Change only the sentences
     that the edit scope names.
   - A mandatory VoiceDNA finding for case style, point of view or question
     rate applies to all sentences, even though Ellora reports it on
     sentence 1.
   - After a failed verification, there is no edit scope. `next_action` names
     what failed. `verification.failure_reasons` lists all the reasons.
     `verification.meaning_changes` gives the details, and so does
     `verification.lost_key_words` when key words were lost. `rewrite_guidance`
     then ends with a "Restore the first draft's meaning" section. That
     section lists the same reasons and quotes the first draft. Put those
     facts back. Do not put back phrasing that the first review flagged.
     Change nothing else.
8. **Verification and learning are automatic.** There is no separate verify
   call.
   - A review call that passes and is not the first in the session includes
     the `verification` block. When that verification succeeds against a
     saved profile, the response also includes a `learning_event`.
   - A failed verification records no `learning_event` and sets the top-level
     `passed` to `false`. `convergence_recommendation` is then
     `keep_rewriting` (with `rewrite_guidance`) until the review-round limit. After
     that, it is `diminishing_returns` without `rewrite_guidance`. In both
     cases, follow `next_action`.
   - `learning_event` is `recorded`, or `nothing_to_learn` when the review
     resolved no findings or repeats an earlier result. A recorded event does
     not edit the profile. It changes later rewrite guidance for that
     profile. For a test or a trial run, use a test profile, not the real
     profile of a writer.
9. **Teach the profile after each accepted rewrite.** When the user accepts a
   rewrite, call `ellora_learn_from_accepted_rewrite` with the same
   `content_session_id`. Send both texts. `original_text` is the draft before
   the rewrite. `accepted_text` is the text that the user accepted, with the
   user's own edits. Add typed rules for what changed, as
   `{type, value, scope?, evidence?}`. `type` is one of `avoid_phrase`,
   `prefer_phrase`, `max_sentence_words`, `no_opener`, `punctuation_limit`,
   `pov`, `question_rate_max` or `freeform`. Ellora runs no model. It keeps
   the rules plus word counts and short phrases from `accepted_text`, never
   the texts. A word stops being flagged once it appears in two of the
   writer's pieces: build samples, or accepted pieces a team admin approved.
   - Do not set `user_approved`. The rules are then suggestions. Ellora
     promotes a rule when it occurs again in other sessions.
   - Set `user_approved: true` only when the user explicitly approved the rules
     in that call. This promotes them at once, and it needs a team admin. If
     the response says `approval: "admin_required"`, tell the user that
     Ellora saved the rules as suggestions for an admin.
   - If the user undoes or rejects a learned rule, send its `rule_key` in
     `rejected_rule_keys`.

   A promoted structured rule comes back from `ellora_review` as a
   `voice_dna` finding with a type that starts with `learned-`. It is blue
   while it is advisory and yellow when it is mandatory.

Repeat steps 5, 6 and 7 until step 6 tells you to stop. Do not stop early
because the draft looks good enough. The `iteration` and `max_iterations`
values tell you how many review rounds you have.

Use one session for one piece of content. For a different piece, or a
different profile or brief, call `ellora_session_start` again.

When the loop ends, show the final draft and a short summary: the number of
review rounds, what changed in general, and what is still flagged that you chose to
keep. Do not show the raw finding JSON unless the user asks.

## Tools

| Tool | Use it to |
|---|---|
| `ellora_session_start` | Start a session for one piece of writing. Locks the profile and the brief. |
| `ellora_review` | Review the draft. Adds rewrite guidance and verifies automatically. |
| `ellora_learn_from_accepted_rewrite` | Send both texts and typed rules after the user accepts a rewrite. The rules are suggestions unless the user approved them. |
| `ellora_profile_create` | **Admins only.** Make a voice profile from 2 to 20 samples. |
| `ellora_profiles_fetch_active` / `_by_id` | Find a voice profile id. |
| `ellora_briefs_fetch_active` / `_by_id` | Find a writing brief id. |
| `ellora_context_fetch_active` / `_by_id` | Read the team's company context documents. |
| `ellora_context_capture` | Save a context document during a session. An admin activates it. |
| `ellora_positioning_recommendations` | Get optional positioning and messaging advice. |

Prompts: `ellora_review_loop`.

## Rules

1. **Never make up results.** If the tools are not connected, say so and
   continue without Ellora. Do not describe a finding, a score or a `passed`
   result that `ellora_review` did not return.
2. **Do not go around access or plan limits.** Sign-up needs the user to
   accept the terms and confirm the email. A connection needs OAuth. The Free
   plan allows one voice profile, one connected AI app, 50,000 words checked
   each month, and no team members. For the Team plan, see the dashboard
   Pricing page or https://getellora.com/#pricing (no prices shown).
   Email us at hello@getellora.com or set up a call at https://cal.com/getellorademo/client-demo.
3. **Do not show a draft as reviewed if Ellora did not review it.** Do not
   say that a draft passed when you stopped on `diminishing_returns`.
4. **Follow `next_action`**, not your own judgment, to decide whether to
   continue the loop.
5. **A brief is mandatory.** If the team has no brief, make the inline
   `writing_brief_json`. Do not skip the review.
6. **Use one session for one piece of content.**
