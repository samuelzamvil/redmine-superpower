# Onboarding versioning — design

**Status:** approved for planning
**Date:** 2026-07-21
**Ships as:** release v1

## Problem

`redmine-conventions.md` is written once per repo and then left alone. When a
release adds a field to the interview — as PR#1 did, adding the whole
`## Collaboration (optional)` section — an already-onboarded repo has no way to
learn that it is missing something. The file is not wrong, so nothing complains;
it is merely old, and staleness is invisible.

This matters now rather than later because the next scheduled work is the
sandbox dry-run (see Follow-ups), whose entire purpose is to shake out changes
to the collab section. Without stamps in place first, those changes land in
every already-onboarded repo as drift nobody can detect. Versioning has to
precede the first change it is meant to announce.

## Numbering

One release counter for the whole repo. This change ships as **v1**. Everything
before it is **unversioned (pre-v1)**.

Because PR#1 merged and no config update has happened since, every
`redmine-conventions.md` in existence today is both unversioned *and* predates
the Collaboration section. A single changelog entry — "unversioned → v1" —
therefore covers the entire real-world gap: what PR#1 added, plus the
introduction of versioning itself.

Independent per-skill counters were rejected: one repo, one PR cadence, one
changelog. Several counters would buy precision that has to be maintained by
hand, and a release that changes nothing about a given section simply produces
no mismatch for it, which is the same outcome.

## Where versions live

### In each skill

A `Release: v1` line in the body, directly under the H1.

Skill bodies are loaded in full when the skill loads, so the current release is
always already in context — the happy path costs no extra read. This is why the
stamp is not in frontmatter: Claude Code's skill schema defines `name` and
`description`, and correctness should not depend on unknown keys surviving.

All five skills carry the line, including `redmine-coordinator` and
`redmine-reviewer`, which own no config but should be able to report their
release.

#### Why not frontmatter, and why not plugin.json

Settled during design; recorded so it is not re-opened.

**Frontmatter.** The documented SKILL.md schema has no `version` field. The
Agent Skills spec does define a `metadata` map for arbitrary keys, and its own
example stores a version there — but whether `metadata` is surfaced into the
model's context at load is undocumented, and the check cannot depend on
undocumented behavior. Notably, lenient handling of unrecognized keys *is*
documented for `plugin.json` and is *not* documented for SKILL.md, so an unknown
top-level key is a worse bet still. The body is the only location guaranteed to
be in context.

`metadata: version:` was considered as a second, spec-sanctioned declaration
alongside the body line and rejected: nothing reads it today, and two version
markers in one file can drift. Drift is uniquely damaging in a field whose only
job is to be trustworthy. Add it when a consumer exists — most likely at plugin
packaging.

**plugin.json.** Reading `${CLAUDE_PLUGIN_ROOT}/.claude-plugin/plugin.json`
would break three properties this design depends on: it exists only under a
plugin install (the documented install here is symlink or copy), it adds a file
read to every session, and `version` is optional — when omitted, Claude Code
falls back to the git commit SHA, which is not comparable to a release counter.

**Two different versions, deliberately not conflated:**

| | Form | Meaning | Bumps when |
| --- | --- | --- | --- |
| Plugin version | semver, in `plugin.json` | Distribution identity | Any published release |
| Conventions schema version | integer, in skill body + conventions file | Compatibility marker | The interview changes |

These legitimately diverge — plugin `1.4.2` may still be schema `2`. Tying them
would prompt a pointless re-onboarding on every patch release. The changelog maps
one to the other.

### In the consuming repo

`redmine-conventions.md` carries two stamps, drawn from the same numbering but
written at different times by different skills, per the existing ownership
split:

- `conventions_version: 1` — base section, written by `redmine-onboarding`
- `collab_version: 1` — inside the Collaboration section, written by
  `redmine-collab-onboarding`

Mutual preservation is unchanged. Each skill stamps only its own section and
must not touch the other's stamp.

### The changelog

`CHANGELOG.md` inside each onboarding skill folder, scoped to the section that
skill owns:

- `skills/redmine-onboarding/CHANGELOG.md` — base fields
- `skills/redmine-collab-onboarding/CHANGELOG.md` — collab fields

They travel with the skill under every install mode (symlink, copy, future
plugin), so nothing resolves a path across folders and the `shared/` install gap
is not made load-bearing. The cost is that the release numbering appears in two
files; this is accepted in exchange for zero cross-folder resolution.

A shared manifest under `skills/shared/` was rejected for this reason: it would
extend the `readlink -f ../shared/` dependency that `redmine-collab-onboarding`
currently carries alone to `redmine-tracking`, which runs every session and must
never fail in a way that blocks work.

## Detection — `redmine-tracking`

At the conventions read it already performs on load, tracking compares the
file's stamps against its own `Release:` line. It emits **at most one advisory
line per session**, then continues.

Tracking never reads a changelog and never gates. Its entire job here is to say
"you are behind, run this skill."

| File state | Behavior |
| --- | --- |
| Base stamp missing | Advise: conventions predate v1 — run `redmine-onboarding` |
| Base stamp < release | Advise: conventions at vN, skills at vM — run `redmine-onboarding` |
| Stamp > release | Advise: the file was written by a newer release — update the installed skills |
| No Collaboration section | **Silent.** Absence is opt-out, not drift — never advise |
| Collab section present, stamp missing or stale | Advise: run `redmine-collab-onboarding` |
| Conventions file missing or unreadable | Existing behavior, unchanged |

The "stamp > release" row advises a different action from the others: running
onboarding cannot fix a stale *install*, so the advice is to update the skills,
not to re-run the interview.

## Upgrade — the onboarding skills

RE-VERIFY mode gains a version step, running before the instance diff:

1. Read the stamp for the section this skill owns.
2. If it matches the current release, skip — no changelog read, no questions.
3. On drift, read `CHANGELOG.md` from this skill's own folder and present
   **only** the entries between the file's version and the current release.
4. Interview for genuinely new fields only. Fields the file already has are not
   re-asked; this is an upgrade, not a re-onboarding.
5. Stamp the new release on write, following the existing standalone-commit rule
   (default branch, nothing else in the commit).

If the changelog is missing or unreadable, report it and **do not bump the
stamp**. A repeated warning is a better failure than a silent false "up to
date."

### Parsing must be lenient

Detect the stamp by scanning the file for a `conventions_version:` line
(and, within the Collaboration section, `collab_version:`) rather than assuming
the template's layout.

Pre-v1 files are by definition written in whatever format predated the template,
and the upgrade flow has to *read* such a file in order to upgrade it. Observed
variations include prose and tables rather than the template's YAML-in-a-fence,
differing custom-field names, and extra sections the template does not define. A
strict parser would misread the first file it ever meets.

## Documentation

- `redmine-conventions.template.md` — show both stamp lines in their sections.
- `README.md` — add a short Versioning section explaining the stamps and the
  update prompt. Fix "Two skills ship here" (five now ship). Extend the install
  recipe to cover all five skill folders plus `shared/`.

The README work is feature scope, not adjacent cleanup: tracking's collab
advisory tells a human to run `redmine-collab-onboarding`, and today's README
gives them no way to have installed it. An advisory pointing at an uninstallable
skill is a dead end.

An uncommitted status banner currently sits at the top of `README.md`. It is the
human's in-flight edit, additive, and in a different section; leave it alone.

## Verification

This repo is markdown with no test harness, so verification is a walkthrough of
the written skill text against each scenario below. The plan should carry these
as explicit checks rather than leaving them implied.

1. Unversioned file, no collab section → exactly one advisory, naming
   `redmine-onboarding`; nothing said about collab.
2. Unversioned file, collab section present → advisories for both.
3. Both stamps current → completely silent; no changelog read.
4. Stamp ahead of release → report only, no "run onboarding" advice.
5. Conventions file absent → existing behavior, no new output.
6. Upgrade run → only the entries between versions are presented; existing
   fields are not re-asked; the other section survives byte-for-byte.
7. Changelog missing during upgrade → reported, stamp not bumped.
8. Lenient parse → a stamp is found in a file that does not follow template
   layout.

## Scope

**In:** everything above, as one commit; the REST-API phrasing reword in both
onboarding skills ("the agent's non-admin API access cannot create these") as a
second commit on the same branch.

**Out:**

- The sandbox dry-run (Follow-up 1). Deferred by the human. Its output is a
  strong candidate for v2.
- Any change to write-verification behavior. Investigated during design and
  found already correct: `redmine-tracking` trusts 2xx for plain comments and
  requires read-back only for workflow-gated writes — status, custom fields,
  parent. Onboarding's read-backs are its acceptance test and stay.
- A plugin package. Noted in the README as the eventual fix for the install
  story; not this change.

## Follow-ups on record

1. **Sandbox dry-run** — the real acceptance test: N=1 full cycle, then N=2 with
   personas, plus the five probes (gate-hold, stall, waiver, TOCTOU, resume).
   Requires `redmine-collab-onboarding` against a sandbox project first.
2. **README update** — folded into this change.
3. **REST-API phrasing** — folded into this change as a separate commit.
