---
name: redmine-onboarding
description: Use when setting up redmine-tracking for a repo for the first time, when redmine-tracking reports missing or invalid conventions, or when the human asks to re-verify or update the Redmine conventions after changing their instance. Interviews the human, writes redmine-conventions.md, verifies the instance from the agent account, and reports anything only an admin can fix.
---

# Redmine Onboarding

Run once per repo/instance pair, and again only after structural changes to
the Redmine instance. Produces exactly one artifact: `redmine-conventions.md`
at the repo root. Everything else is verification and reporting.

## Mode detection

- No `redmine-conventions.md` in the repo → FRESH mode.
- File exists → RE-VERIFY mode: probe the instance, diff discovered
  structure against the file, present the diff, and update ONLY what the
  human approves. Never edit on your own initiative.

## Fresh mode: interview

Brainstorming style — one question at a time, propose defaults from what
you can discover before asking. Establish:

1. Redmine project identifier (list projects the agent account can see;
   confirm the choice).
2. Tracker role mapping: which tracker plays epic, which plays task, which
   plays subtask. Propose from the project's enabled trackers.
3. Status role mapping: backlog, active, blocked, ceiling (agent stops
   here), rejected, closed (human-only). Propose from the instance's
   status list; instances vary wildly — never assume default names.
4. Custom field names for branch and PR URL.
5. Branch naming pattern (default: `task-{id}-{slug}`).

What does NOT go in the file: the Epic list (live Redmine data), issue
IDs, anything queryable at runtime. Structural mappings only.

## Write and commit

1. Write `redmine-conventions.md` from the template, with the answers.
2. Show it to the human for approval.
3. Commit it as a STANDALONE commit on the DEFAULT branch — nothing else
   in the commit, never on a work branch, never bundled with other work.
   This is the single sanctioned no-ticket repo write; it exists so active
   ticket branches can rebase over conventions changes without conflict.

## Verify the instance (from the agent account, always)

Run these against the configured project; they double as the acceptance
test for the instance setup. Redmine returns 204 for workflow-forbidden
changes without applying them, so every probe is write-then-read-back.

1. Membership: the agent account can see the project.
2. Trackers and statuses named in the conventions all exist.
3. Epics resolve: at least one issue with the epic-role tracker exists.
4. Grid probe on a labeled fixture issue: attempt each transition the
   tracking skill uses (backlog→active, active→blocked, blocked→active,
   active→ceiling, ceiling→active); read back; confirm applied. Attempt a
   transition to closed; confirm it does NOT apply (if it applies, warn
   the human: the agent role can close tickets, which the workflow forbids
   by design).
5. Custom fields round-trip: set branch and PR fields, read back.
6. done_ratio: write a value, read back. If it does not stick, report:
   instance setting "calculate done ratio" must be "use the issue field".
7. Delete or close nothing: leave the fixture issue for the human, labeled
   as a fixture.

## Report

End with two lists:
- WORKING: what was verified.
- NEEDS ADMIN: anything only the admin web UI can fix (trackers, statuses,
  custom fields, roles, workflow grid, instance settings — the REST API
  cannot create or modify any of these), phrased as concrete steps.

Do not proceed to normal tracking work in the same session unless the
verification passed clean or the human explicitly says to continue.

Optionally: to set up multi-model collaboration for this repo, run
`redmine-collab-onboarding` once this verification passes clean.

## Ground rules

- Fixtures are labeled as fixtures and live only in the configured
  project; nothing is created the human didn't approve.
- Read/refusal paths never mutate. One agent writes at a time.
- Re-verify mode never deletes conventions the human didn't ask to remove.
