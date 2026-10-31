---
name: ellora-review
description: >-
  Run the Ellora brand-voice + AI-Editor review loop on prose written or
  edited for the team — blog posts, marketing copy, launch announcements,
  customer emails, docs, landing-page and social copy. Use whenever you
  draft or substantially revise content of this kind, BEFORE showing it to
  the user, even if "Ellora" is not mentioned by name. Walks through
  starting a content session, reviewing against the team's saved voice
  profile and writing brief, rewriting, and looping until the draft
  converges (or verification passes). Requires the Ellora MCP server to be
  connected.
---

# Ellora review loop

The Ellora MCP server (`ellora_*` tools) is this team's automated writing QA:
**VoiceDNA** checks a draft against a saved brand-voice profile, **AI Editor**
strips AI-tell / slop, and an optional **Editorial** pack checks format-level
brief rules. This skill is the procedure for using it well — in particular,
for *looping until the draft is actually done* instead of stopping after one
or two passes.

## When to use this

Use it whenever you produce or meaningfully rewrite prose that is meant to go
out in the team's voice:

- blog posts, articles, newsletters
- landing-page / website copy, marketing one-pagers
- launch or release announcements
- customer-facing emails, in-app copy, release notes
- social posts, docs written for an external audience

You do **not** need the user to say "Ellora", "run the review loop", or
"check the voice" — if the content is of this kind, run the loop before you
present a draft.

**Skip it for:** code, config, commit messages, internal scratch notes,
quick chat replies, or content the user explicitly says is a rough
throwaway. If in doubt, ask: "Want me to run this through the Ellora voice
check?"

## Prerequisites

The Ellora tools must be connected: `ellora_session_start`, `ellora_review`,
`ellora_learn_from_accepted_rewrite`, `ellora_profile_create`,
`ellora_profiles_fetch_active`,
`ellora_profiles_fetch_by_id`, `ellora_briefs_fetch_active`,
`ellora_briefs_fetch_by_id`, `ellora_context_fetch_active`,
`ellora_context_fetch_by_id`, `ellora_context_capture`,
`ellora_positioning_recommendations`. If they are not available, tell the user
the Ellora MCP server isn't connected and continue without it — don't
fabricate results.

The older `ellora_analyze`, `ellora_verify`, `ellora_rewrite_prompt`,
`ellora_prepare_rewrite` and `ellora_apply_rewrite` tools no longer exist —
they were merged into `ellora_review`. One `ellora_review` call analyzes the
draft, returns rewrite guidance when findings are open, and auto-verifies once
the draft passes. There is no separate verify call.

## The loop

1. **Pick a voice profile.** Call `ellora_profiles_fetch_active`.
   - Exactly one → use it.
   - More than one → ask the user which one this piece is for.
   - None → a team admin can create one with `ellora_profile_create` from
     2-20 samples of the team's own writing, then use the id it returns. If
     they aren't a team admin, tell them a profile has to exist first — you
     can't run VoiceDNA without one.

2. **Pick a writing brief.** Call `ellora_briefs_fetch_active`. Same pattern:
   one → use it; several → ask; none → construct an inline `writing_brief_json`
   for this session only (it is never saved). **A brief is mandatory** —
   `ellora_session_start` fails without either a `writing_brief_id` or a
   `writing_brief_json`.

3. **Start the session.** Call `ellora_session_start` with the chosen
   `voice_profile_id` and `writing_brief_id` / `writing_brief_json`. These
   are **locked for the whole session** — every later call reuses them.
   Keep the returned `content_session_id` and pass it to every call below,
   for this one piece of content. The response also tells you
   `max_iterations` (the loop budget).
   - Pass `writing_brief_json` as a JSON **object**, never as a stringified
     one. Required fields: `version` (`"1"`), `audience`, `intent`, `format`.
   - `format` must be one of: `general`, `social`, `deck`, `outreach`,
     `blog`, `audit`, `website`.

4. **Optionally pull positioning.** When messaging guidance would help, call
   `ellora_positioning_recommendations` with the `content_session_id` before
   drafting. Its output is advisory and never changes `ellora_review`.

5. **Review.** Call `ellora_review` with the `content_session_id` and the
   current draft as `content`.

6. **Read `convergence_recommendation`, then do what `next_action` says.**
   Separate fields: `convergence_recommendation` is `keep_rewriting`,
   `ready_to_verify`, or `diminishing_returns`; `next_action` is the imperative
   summary (and may name open blue suggestions). The response also carries
   `iteration` / `max_iterations`. Follow `next_action`:
   - **`keep_rewriting`** → open mandatory findings and/or a failed verification
     with passes left. The response includes `rewrite_guidance`. Go to step 7.
     **Do not show the draft to the user yet.**
   - **`ready_to_verify`** → no mandatory findings remain. On the session's
     first review, `next_action` says deliver as-is. On later reviews, this
     response includes `verification` — deliver when `verification.passed` is
     `true`. If a judgment-required finding is open, `next_action` names it:
     get a human decision before you deliver.
   - **`diminishing_returns`** → mandatory findings stalled, you hit
     `max_iterations`, or verification failed on the last allowed pass. Stop
     looping. Tell the user what is still wrong and ask how to proceed.

   **Blue findings never gate.** Optional suggestions in
   `current_finding_counts.blue`. They never change `convergence_recommendation`,
   never appear in `rewrite_guidance`'s edit scope, and never require another
   pass on their own. Read `next_action` for open blue notes; apply or ignore.

7. **Rewrite, then re-review.** When the response includes `rewrite_guidance`,
   use it to rewrite the content yourself, then **go back to step 5** with the
   new text and the same `content_session_id`. Send the whole revised content,
   not only the changed sentences. **Edit scope** covers only mandatory Red/Yellow
   findings (`rewrite_guidance` also has a Blue section). Change only the sentences
   Edit scope names. A case-style, point-of-view, or question-rate finding covers
   every sentence even though it is reported on sentence 1. After a failed verify
   there is no Edit scope. `next_action` names what failed, and
   `verification.meaning_changes` and `verification.lost_key_words` give the
   detail. Put those facts back without restoring phrasing the first review
   flagged, and change nothing else.

   Keep going around 5 → 6 → 7 until step 6 sends you out of the loop. Do
   not stop early just because the draft "seems good enough" — the
   `iteration` / `max_iterations` budget is there so you can keep going with
   confidence; that's the whole point of this skill.

8. **Verification is automatic.** Once a call passes and it is not the
   session's first review, that same response includes a `verification`
   block comparing the final text against the session's original content —
   and, on success against a saved profile, a `learning_event`. There is no
   separate verify call to make. Stop and deliver once you see
   `verification.passed: true`. A failed verify records no `learning_event` and sets
   top-level `passed` to `false`. `convergence_recommendation` becomes
   `keep_rewriting` (with `rewrite_guidance`) until the pass limit, then
   `diminishing_returns` without `rewrite_guidance`. Follow `next_action` in
   either case. That learning write needs no extra call. A passing review
   against a saved profile changes that profile: for a test or dry run, use a
   scratch profile, not a writer's real one.

9. **Teach the profile only on explicit approval.** If the user accepts the
   delivered rewrite AND approves one or more concise rules you extracted
   from it, call `ellora_learn_from_accepted_rewrite` with the same
   `content_session_id`, those rules, and `user_approved: true`. Do not call
   it on your own inference.

## What to tell the user at the end

Once the loop finishes, present the final draft plus a short summary:

- how many passes it took (`iteration`)
- what changed between the first draft and the final one, at a high level
- anything still flagged that you chose not to act on, and why (e.g.
  judgment-required advisories, or a `diminishing_returns` stop)

Don't dump raw finding JSON at the user unless they ask for it.

## Notes

- One session = one piece of content. A different article, or a switch to a
  different voice profile / brief, means a **new** `ellora_session_start`.
- Only `ellora_review` writes history rows (visible read-only in the Ellora
  app under Content Sessions).
- The MCP server also exposes an `ellora_review_loop` prompt with this same
  sequence — this skill is the fuller version, with the "keep looping" and
  "when to use" guidance the prompt can't carry.
