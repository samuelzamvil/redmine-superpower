---
name: redmine-onboarding
description: 'Use when the human asks to set up, update, change, fix, redo, or re-verify Redmine config, configuration, conventions, or status/tracker mappings for a repo — e.g. "update my redmine config", "redo the redmine setup", "point this repo at redmine", "my redmine status mappings are wrong". Also use when Redmine work is requested in a repo with no conventions/redmine.md, or redmine-tracking reports that file missing, stale, or invalid. Base conventions only — collaboration setup uses redmine-collab-onboarding.'
---

# Redmine Onboarding

**Release:** 2 (conventions schema version — see `CHANGELOG.md` in this folder)

Run once per repo/instance pair, and again only after structural changes to
the Redmine instance. Produces exactly one artifact: the conventions file.
Everything else is verification and reporting.

## Locating the conventions file

Resolve in this order and stop at the first hit:

1. `conventions/redmine.md` — the current location.
2. `redmine-conventions.md` at the repo root — the older layout, still valid.

**Never relocate a file you found.** A repo using the root layout keeps it;
moving it is the repo's decision, not this skill's. Write back to wherever
resolution landed. Only FRESH mode chooses a location, and it chooses
`conventions/redmine.md`.

## Mode detection

- Neither location resolves → FRESH mode.
- File exists at either location → RE-VERIFY mode: probe the instance, diff
  discovered structure against the file, present the diff, and update ONLY
  what the human approves. Never edit on your own initiative.

**Collaboration section ownership.** If the file contains any section whose
heading begins `## Collaboration`, preserve it byte-for-byte. Match the
heading loosely: a section written before v1 may not carry the `(optional)`
suffix, and it is protected all the same. This
skill owns only the base conventions above it and never creates, edits,
reorders, or deletes the Collaboration section — that section belongs to
`redmine-collab-onboarding` exclusively. A RE-VERIFY diff must never
present the Collaboration section as drift, even though it maps to no
instance structure this skill probes.

## Version step (RE-VERIFY only, before the instance diff)

1. Scan `conventions/redmine.md` for a `conventions_version:` line. Scan the
   WHOLE file — pre-v1 files predate the current template, so do not assume a
   position, a fenced block, or the template's field names.
2. If it equals this skill's declared release, SKIP this section entirely. Do
   not open `CHANGELOG.md`; do not raise the subject with the human.
3. If the file's version is HIGHER than this skill's declared release, STOP
   the version step here: report that this install is older than the file, do
   not read `CHANGELOG.md`, and do not write a stamp. Continue to the instance
   diff. Never stamp a version backwards.
4. Otherwise read `CHANGELOG.md` from this skill's own folder and present ONLY
   the entries between the file's version (absent means pre-v1) and this
   release.
5. Interview for fields those entries introduce and the file lacks. Never
   re-ask a field the file already answers — this is an upgrade, not a
   re-onboarding.
6. Write `conventions_version: <release>` along with any approved changes,
   under the standalone-commit rule below. Preserve the file's existing style;
   do not reformat a pre-v1 file into the template.

If `CHANGELOG.md` is missing or unreadable, say so and do NOT write a version
stamp. A repeated warning next session is a better failure than a silent false
"up to date".

Stamp only `conventions_version:`. The Collaboration section's `collab_version:`
belongs to `redmine-collab-onboarding` and is never touched here.

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

1. Write the conventions file from the template, with the answers. In FRESH
   mode the destination is `conventions/redmine.md`, creating the directory
   if absent; in RE-VERIFY it is whichever path resolution found, unmoved.
   Never copy the template's Collaboration section — fresh runs write only
   the base conventions. If the file already exists (RE-VERIFY), merge the
   approved base-field changes into it rather than regenerating the whole
   file, so an existing Collaboration section survives untouched.
   In FRESH mode, write `conventions_version:` as the first field, set to this
   skill's declared release — take that value from the `Release:` line at the
   top of this file, never from the template, so a fresh file is never born
   stale. In RE-VERIFY mode the Version step above already governs the stamp;
   write none here.
2. Show it to the human for approval.
3. Commit it as a STANDALONE commit on the DEFAULT branch — nothing else
   in the commit, never on a work branch, never bundled with other work.
   This is the single sanctioned no-ticket repo write; it exists so active
   ticket branches can rebase over conventions changes without conflict.

## Verify the instance (from the agent account, always)

Run these against the configured project; they double as the acceptance
test for the instance setup. Treat every successful 2xx response as success;
do not read solely to confirm a successful write.

1. Membership: the agent account can see the project.
2. Trackers and statuses named in the conventions all exist.
3. Epics resolve: at least one issue with the epic-role tracker exists.
4. On one labeled fixture issue, exercise the normal tracking writes once:
   set the active status, branch, PR, and `done_ratio`. Successful 2xx
   responses pass; do not read them back.
5. Report that the instance setting "calculate done ratio" must be
   "use the issue field"; a non-admin account cannot inspect or change it.
6. Delete or close nothing: leave the fixture issue for the human, labeled
   as a fixture.

## Report

End with two lists:
- WORKING: what was verified.
- NEEDS ADMIN: anything only the admin web UI can fix (trackers, statuses,
  custom fields, roles, workflow grid, instance settings — the agent's
  non-admin API access cannot create or modify any of these), phrased as
  concrete steps.

Do not proceed to normal tracking work in the same session unless the
verification passed clean or the human explicitly says to continue.

Optionally: to set up multi-model collaboration for this repo, run
`redmine-collab-onboarding` once this verification passes clean.

## Ground rules

- Fixtures are labeled as fixtures and live only in the configured
  project; nothing is created the human didn't approve.
- Read/refusal paths never mutate. One agent writes at a time.
- Re-verify mode never deletes conventions the human didn't ask to remove.
