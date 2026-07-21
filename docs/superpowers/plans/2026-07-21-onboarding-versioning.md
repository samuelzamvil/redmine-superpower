# Onboarding Versioning Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Stamp a conventions schema version into every skill and into
`redmine-conventions.md`, so an already-onboarded repo learns it predates a
release and is told which skill to re-run.

**Architecture:** One repo-wide integer schema counter, shipping as v1. Each
`SKILL.md` declares it in its body (the only location guaranteed to be in the
model's context at load). `redmine-conventions.md` carries `conventions_version:`
in the base section and `collab_version:` inside the Collaboration section, each
written only by the skill that owns that section. `redmine-tracking` compares and
advises; it never reads a changelog and never gates. Each onboarding skill keeps
a `CHANGELOG.md` in its own folder, opened only during an upgrade.

**Tech Stack:** Markdown only. No build, no test runner, no dependencies.

## Global Constraints

- Ships as **v1**. Files with no stamp are **unversioned (pre-v1)**.
- **Never name a local or private project** — no repo names, instance
  hostnames, or project identifiers — in any file, commit message, or
  rationale. This repo is public.
- Verification is by **reading the written skill text**, not by running code.
  There is no test harness; "the test" is a grep plus a walkthrough.
- **Mutual preservation is unchanged.** `redmine-onboarding` never touches the
  Collaboration section; `redmine-collab-onboarding` never touches anything
  above it. Each stamps only its own section.
- Tracking's version check must **never block work** and must **never open a
  changelog**.
- Branch: `onboarding-versioning`. Tasks 1–5 are the feature; **Task 6 is a
  strictly separate commit** and must not be folded into any other.
- Do not stage `README.md` before Task 5 — it carries an unrelated
  uncommitted status banner belonging to the human.

---

### Task 1: Declare the release in all five skills and the template

**Files:**
- Modify: `skills/redmine-onboarding/SKILL.md:6`
- Modify: `skills/redmine-tracking/SKILL.md:6`
- Modify: `skills/redmine-collab-onboarding/SKILL.md:6`
- Modify: `skills/redmine-coordinator/SKILL.md:6`
- Modify: `skills/redmine-reviewer/SKILL.md:6`
- Modify: `redmine-conventions.template.md`

**Interfaces:**
- Produces: the literal line `**Release:** v1` in each skill body, directly
  under the H1. Tasks 3 and 4 compare against this. Produces the stamp field
  names `conventions_version:` and `collab_version:`, which Tasks 2–5 all use
  verbatim.

- [ ] **Step 1: Add the release line to each of the five skills**

Insert immediately after the H1 line, followed by a blank line. For the two
onboarding skills use the changelog-bearing form:

```markdown
**Release:** v1 (conventions schema version — see `CHANGELOG.md` in this folder)
```

For `redmine-tracking`, `redmine-coordinator`, and `redmine-reviewer` use:

```markdown
**Release:** v1 (conventions schema version)
```

Exact H1 lines to insert after:
- `skills/redmine-onboarding/SKILL.md` → `# Redmine Onboarding`
- `skills/redmine-tracking/SKILL.md` → `# Redmine Tracking for Superpowers`
- `skills/redmine-collab-onboarding/SKILL.md` → `# Redmine Collab Onboarding`
- `skills/redmine-coordinator/SKILL.md` → `# Redmine Coordinator`
- `skills/redmine-reviewer/SKILL.md` → `# Redmine Reviewer`

Do not touch frontmatter. The SKILL.md schema has no `version` field, and
unknown-key leniency is documented for `plugin.json` but not for skills.

- [ ] **Step 2: Verify all five carry the line**

Run: `grep -c "^\*\*Release:\*\* v1" skills/*/SKILL.md`

Expected: five lines, each ending `:1` —

```
skills/redmine-collab-onboarding/SKILL.md:1
skills/redmine-coordinator/SKILL.md:1
skills/redmine-onboarding/SKILL.md:1
skills/redmine-reviewer/SKILL.md:1
skills/redmine-tracking/SKILL.md:1
```

- [ ] **Step 3: Add the base stamp to the template**

In `redmine-conventions.template.md`, inside the first fenced block, add the
stamp as the first field. The block currently opens:

```markdown
# Redmine conventions
# Managed by redmine-onboarding. Do not edit on a work branch.

project_identifier: example-project
```

Change it to:

```markdown
# Redmine conventions
# Managed by redmine-onboarding. Do not edit on a work branch.

# Schema version this file was written under. Compared against the skills'
# declared release; a mismatch prompts a re-run of redmine-onboarding.
conventions_version: 1

project_identifier: example-project
```

- [ ] **Step 4: Add the collab stamp to the template**

In the same file, inside the Collaboration fenced block, add the stamp as the
first field. The block currently opens:

```markdown
# Collaboration — managed by redmine-collab-onboarding. Do not edit on a work branch.

- coordinator_account: <redmine login>        # posts contracts, artifacts, recaps
```

Change it to:

```markdown
# Collaboration — managed by redmine-collab-onboarding. Do not edit on a work branch.

- collab_version: 1                           # schema version of this section
- coordinator_account: <redmine login>        # posts contracts, artifacts, recaps
```

- [ ] **Step 5: Verify both template stamps**

Run: `grep -n "conventions_version:\|collab_version:" redmine-conventions.template.md`

Expected: exactly two matches, `conventions_version: 1` appearing before
`collab_version: 1`.

- [ ] **Step 6: Commit**

```bash
git add skills/redmine-onboarding/SKILL.md skills/redmine-tracking/SKILL.md \
        skills/redmine-collab-onboarding/SKILL.md skills/redmine-coordinator/SKILL.md \
        skills/redmine-reviewer/SKILL.md redmine-conventions.template.md
git commit -m "feat: declare conventions schema v1 in all skills and the template"
```

---

### Task 2: Write the two changelogs

**Files:**
- Create: `skills/redmine-onboarding/CHANGELOG.md`
- Create: `skills/redmine-collab-onboarding/CHANGELOG.md`

**Interfaces:**
- Consumes: the release value `v1` and the stamp names from Task 1.
- Produces: `CHANGELOG.md` at a path each onboarding skill can reach as a
  sibling of its own `SKILL.md`. Task 4 reads these.

**Why two files:** they travel with the skill under every install mode, so
nothing resolves a path across folders.

- [ ] **Step 1: Write the base changelog**

Create `skills/redmine-onboarding/CHANGELOG.md`:

```markdown
# redmine-onboarding — conventions changelog

Read by this skill during a RE-VERIFY upgrade, and ONLY when the file's
`conventions_version:` differs from this skill's declared release. Each entry
records what the interview gained, so an upgrade asks only about genuinely new
fields.

Covers the BASE conventions only. The `## Collaboration (optional)` section has
its own changelog at `skills/redmine-collab-onboarding/CHANGELOG.md`.

## unversioned → v1

A file with no `conventions_version:` line was written before versioning
existed.

**Base conventions: no new fields.** A pre-v1 base section is already complete.
The upgrade adds the `conventions_version: 1` stamp and asks nothing.

**What else shipped in this release:** multi-model collaboration —
`redmine-collab-onboarding`, `redmine-coordinator`, `redmine-reviewer`, and the
optional `## Collaboration (optional)` section of the conventions file. It is
opt-in: a repo that never runs two models needs none of it. To adopt it, run
`redmine-collab-onboarding`.

**Format note.** Pre-v1 files predate the current template and may use prose,
tables, or different field names. Preserve the file's existing style when
stamping it. Do not reformat it into the template.
```

- [ ] **Step 2: Write the collab changelog**

Create `skills/redmine-collab-onboarding/CHANGELOG.md`:

```markdown
# redmine-collab-onboarding — collaboration changelog

Read by this skill during a RE-VERIFY upgrade, and ONLY when the section's
`collab_version:` differs from this skill's declared release.

Covers the `## Collaboration (optional)` section only. The base conventions have
their own changelog at `skills/redmine-onboarding/CHANGELOG.md`.

## unversioned → v1

A Collaboration section with no `collab_version:` line was written by the
release that introduced collaboration, before versioning existed. Every field of
the current interview already existed then, so the upgrade adds the
`collab_version: 1` stamp and asks nothing.

A repo with NO Collaboration section is not stale — collaboration is opt-in, and
its absence is a choice. Run this skill in fresh mode to adopt it.
```

- [ ] **Step 3: Verify both exist and are sibling to their skill**

Run: `ls skills/*/CHANGELOG.md`

Expected:

```
skills/redmine-collab-onboarding/CHANGELOG.md
skills/redmine-onboarding/CHANGELOG.md
```

- [ ] **Step 4: Verify no private project names leaked**

Build the pattern locally from the private repo names, project identifiers, and
instance hostnames you work with. **Do not commit the pattern** — writing those
names into a tracked file is itself the leak this step exists to catch.

```bash
PRIVATE='<name1>|<name2>|<host.example.com>'   # fill in locally, never commit
grep -rniE "$PRIVATE" skills/ ; echo "exit=$?"
```

Expected: `exit=1` (no matches).

- [ ] **Step 5: Commit**

```bash
git add skills/redmine-onboarding/CHANGELOG.md skills/redmine-collab-onboarding/CHANGELOG.md
git commit -m "feat: add per-skill conventions changelogs"
```

---

### Task 3: Add the version check to redmine-tracking

**Files:**
- Modify: `skills/redmine-tracking/SKILL.md` — insert a new section after the
  `## Conventions file (read first, read-only)` section (which ends at the
  LIVE DATA bullet) and before `## The ticket rule`

**Interfaces:**
- Consumes: `**Release:** v1` from Task 1; the stamp names
  `conventions_version:` / `collab_version:`.
- Produces: the advisory behavior. Nothing later depends on it.

- [ ] **Step 1: Write the failing check — confirm no version logic exists yet**

Run: `grep -n "conventions_version\|collab_version" skills/redmine-tracking/SKILL.md ; echo "exit=$?"`

Expected: `exit=1`. This is the "test fails first" state — tracking currently
has no way to detect a stale file.

- [ ] **Step 2: Insert the version check section**

Insert immediately before the line `## The ticket rule`:

```markdown
## Version check (advisory, at the conventions read)

This skill declares its release at the top of this file. `redmine-conventions.md`
records the release it was written under. Compare them while reading the
conventions, emit the advisory below, then carry on.

This check NEVER gates work and NEVER opens a changelog. Naming the skill to
re-run is its entire job.

Find the stamps by scanning the file for a `conventions_version:` line, and —
only within a `## Collaboration` section — a `collab_version:` line. Do not
assume the template's layout: files written before v1 predate that template and
may use prose, tables, or different field names.

| File state | Say |
| --- | --- |
| No `conventions_version:` found | "Conventions predate v1. Run `redmine-onboarding` to update." |
| `conventions_version:` lower than this release | "Conventions at vN, skills at vM. Run `redmine-onboarding` to update." |
| `conventions_version:` higher than this release | "Conventions at vN, skills at vM — this install is older than the file. Update the installed skills." |
| No `## Collaboration` section at all | Say NOTHING. Absence is opt-out, not drift. |
| Collaboration section present, `collab_version:` missing or lower | "Collaboration section at vN, skills at vM. Run `redmine-collab-onboarding` to update." |
| Conventions file missing or unreadable | Existing behavior above. Add nothing here. |

At most one line per section, at most once per session — a repo can be stale in
both the base and the Collaboration section, which is two lines total, said
once. Never repeat them later in the session.
```

- [ ] **Step 3: Verify the section landed in the right place**

Run: `grep -n "^## " skills/redmine-tracking/SKILL.md | head -6`

Expected: `## Version check (advisory, at the conventions read)` appears
AFTER `## Conventions file (read first, read-only)` and BEFORE
`## The ticket rule`.

- [ ] **Step 4: Walk the table against the spec scenarios**

Read the inserted table and confirm each row by inspection:

1. Unversioned file, no collab section → row 1 fires, row 4 silences collab.
   Exactly one line, naming `redmine-onboarding`. ✓
2. Unversioned file, collab section present → rows 1 and 5 fire. Two lines. ✓
3. Both stamps current → no row matches. Silent, no changelog read. ✓
4. Stamp ahead of release → row 3, advises updating the install, NOT re-running
   onboarding (that could not fix a stale install). ✓
5. Conventions file absent → row 6 defers to existing behavior, adds nothing. ✓
6. Lenient parse → the "scan for a line" instruction, not a layout assumption. ✓

- [ ] **Step 5: Verify the no-changelog property holds**

Run: `grep -in "changelog" skills/redmine-tracking/SKILL.md`

Expected: exactly ONE match — the prohibition itself, "NEVER opens a
changelog". Any second match means tracking has been given an instruction to
READ a changelog, which breaks the happy-path-costs-nothing property.

- [ ] **Step 6: Commit**

```bash
git add skills/redmine-tracking/SKILL.md
git commit -m "feat: advise on conventions version drift from redmine-tracking"
```

---

### Task 4: Add the upgrade flow to both onboarding skills

**Files:**
- Modify: `skills/redmine-onboarding/SKILL.md` — insert after the
  `## Mode detection` section (after the Collaboration-section-ownership
  paragraph), before `## Fresh mode: interview`
- Modify: `skills/redmine-collab-onboarding/SKILL.md` — insert after the
  `## Mode detection` section, before `## Fresh mode: interview`

**Interfaces:**
- Consumes: `CHANGELOG.md` from Task 2, sibling to each `SKILL.md`; the
  `**Release:** v1` line from Task 1.
- Produces: the upgrade behavior. Nothing later depends on it.

- [ ] **Step 1: Confirm neither skill has version logic yet**

Run: `grep -n "conventions_version\|collab_version\|CHANGELOG" skills/redmine-onboarding/SKILL.md skills/redmine-collab-onboarding/SKILL.md ; echo "exit=$?"`

Expected: only the `**Release:**` lines from Task 1 match (they name
`CHANGELOG.md`). No stamp-handling instructions exist yet.

- [ ] **Step 2: Insert the version step into redmine-onboarding**

Insert immediately before `## Fresh mode: interview`:

```markdown
## Version step (RE-VERIFY only, before the instance diff)

1. Scan `redmine-conventions.md` for a `conventions_version:` line. Scan the
   WHOLE file — pre-v1 files predate the current template, so do not assume a
   position, a fenced block, or the template's field names.
2. If it equals this skill's declared release, SKIP this section entirely. Do
   not open `CHANGELOG.md`; do not raise the subject with the human.
3. Otherwise read `CHANGELOG.md` from this skill's own folder and present ONLY
   the entries between the file's version (absent means pre-v1) and this
   release.
4. Interview for fields those entries introduce and the file lacks. Never
   re-ask a field the file already answers — this is an upgrade, not a
   re-onboarding.
5. Write `conventions_version: <release>` along with any approved changes,
   under the standalone-commit rule below. Preserve the file's existing style;
   do not reformat a pre-v1 file into the template.

If `CHANGELOG.md` is missing or unreadable, say so and do NOT write a version
stamp. A repeated warning next session is a better failure than a silent false
"up to date".

Stamp only `conventions_version:`. The Collaboration section's `collab_version:`
belongs to `redmine-collab-onboarding` and is never touched here.
```

- [ ] **Step 3: Insert the version step into redmine-collab-onboarding**

Insert immediately before `## Fresh mode: interview`:

```markdown
## Version step (RE-VERIFY only, before the instance diff)

1. Scan the `## Collaboration (optional)` section for a `collab_version:` line.
   Scan the whole section — a pre-v1 section may not follow the current
   template's layout.
2. If it equals this skill's declared release, SKIP this section entirely. Do
   not open `CHANGELOG.md`; do not raise the subject with the human.
3. Otherwise read `CHANGELOG.md` from this skill's own folder and present ONLY
   the entries between the section's version (absent means pre-v1) and this
   release.
4. Interview for fields those entries introduce and the section lacks. Never
   re-ask a field the section already answers — this is an upgrade, not a
   re-onboarding.
5. Write `collab_version: <release>` along with any approved changes, under the
   standalone-commit rule below.

If `CHANGELOG.md` is missing or unreadable, say so and do NOT write a version
stamp. A repeated warning next session is a better failure than a silent false
"up to date".

Stamp only `collab_version:`, inside the Collaboration section. Everything above
that section — including `conventions_version:` — belongs to
`redmine-onboarding` and is never touched here.
```

- [ ] **Step 4: Verify placement in both files**

Run: `grep -n "^## " skills/redmine-onboarding/SKILL.md skills/redmine-collab-onboarding/SKILL.md | grep -A1 -B1 "Version step"`

Expected: in each file, `## Version step (RE-VERIFY only, before the instance
diff)` appears after `## Mode detection` and before `## Fresh mode: interview`.

- [ ] **Step 5: Verify mutual preservation survived**

Run: `grep -n "collab_version" skills/redmine-onboarding/SKILL.md`

Expected: exactly one match — the line stating that `collab_version:` is never
touched here. Base onboarding must never be instructed to WRITE it.

Run: `grep -n "conventions_version" skills/redmine-collab-onboarding/SKILL.md`

Expected: exactly one match — the line stating it belongs to base onboarding.

- [ ] **Step 6: Walk the upgrade scenarios**

By inspection of the two inserted sections:

1. Stamps current → step 2 short-circuits. No changelog opened, no questions. ✓
2. Pre-v1 file → step 3 presents the `unversioned → v1` entry only. Per the
   changelogs from Task 2, neither adds fields, so step 4 asks nothing and
   step 5 stamps. ✓
3. Changelog missing → stamp withheld, warning repeats next session. ✓
4. Other section survives byte-for-byte → step 5 plus the closing paragraph. ✓

- [ ] **Step 7: Commit**

```bash
git add skills/redmine-onboarding/SKILL.md skills/redmine-collab-onboarding/SKILL.md
git commit -m "feat: add version upgrade step to both onboarding skills"
```

---

### Task 5: Update the README

**Files:**
- Modify: `README.md:25` (the "Two skills ship here" list)
- Modify: `README.md:56-69` (the Install section)
- Modify: `README.md` — add a Versioning section after "The conventions file"

**Interfaces:**
- Consumes: everything from Tasks 1–4.
- Produces: nothing consumed by later tasks.

**Note:** `README.md` carries an uncommitted status banner added by the human,
directly under the intro paragraph. Leave it exactly as it is. It is additive
and in a different section; `git add README.md` in Step 6 will sweep it into
this commit, which is acceptable and expected — do not revert or rewrite it.

- [ ] **Step 1: Replace the skill list**

Replace the line `Two skills ship here:` and the two bullets under it with:

```markdown
Five skills ship here. Two are the core:

- **`redmine-tracking`** — the runtime skill. Reads a per-repo conventions
  file, then mirrors each superpowers phase (brainstorm → plan → execute →
  debug → complete) into Redmine ticket state. Comments are free; questions
  are expensive.
- **`redmine-onboarding`** — the setup skill. Run once per repo/instance
  pair. Interviews you, writes `redmine-conventions.md` at the repo root,
  and verifies the instance from the agent account — reporting anything only
  an admin can fix.

Three more are optional, for running two models against one ticket:

- **`redmine-collab-onboarding`** — appends the Collaboration section to your
  conventions file and verifies each declared agent account.
- **`redmine-coordinator`** — runs the superpowers workflow while routing
  questions to model reviewers through the ticket journal.
- **`redmine-reviewer`** — builds an independent prior and challenges the
  coordinator's claims through ticket comments.
```

- [ ] **Step 2: Replace the symlink install recipe**

Replace the fenced bash block under **Personal (symlink, edits stay live):**
with:

```bash
git clone https://github.com/samuelzamvil/redmine-superpower
cd redmine-superpower
for s in redmine-onboarding redmine-tracking redmine-collab-onboarding \
         redmine-coordinator redmine-reviewer; do
  ln -s "$(pwd)/skills/$s" ~/.claude/skills/"$s"
done
```

- [ ] **Step 3: Fix the per-repo install note**

Replace the paragraph beginning **Per-repo (scoped to one consuming repo):**
with:

```markdown
**Per-repo (scoped to one consuming repo):** copy the skill folders you want
into that repo's `.claude/skills/`. If you copy any of the three collaboration
skills, copy `skills/shared/` alongside them — they read
`shared/collab-protocol.md`. Symlink installs resolve it automatically from the
link's real path, so it needs no separate link.
```

- [ ] **Step 4: Add the Versioning section**

Insert after the "## The conventions file" section, before "## License":

```markdown
## Versioning

Each skill declares a **conventions schema version** — the `Release:` line under
its title. `redmine-conventions.md` records the version it was written under:
`conventions_version:` for the base mappings, `collab_version:` inside the
Collaboration section.

When they diverge, `redmine-tracking` says so once at session start and names
the skill to re-run. It never blocks work. Re-running that skill reads its
`CHANGELOG.md`, shows only what changed between your version and the current
one, and asks only about genuinely new fields — it is an upgrade, not a
re-onboarding.

Files written before versioning existed carry no stamp. They are treated as
pre-v1 and upgraded the same way; your file's existing format is preserved
rather than rewritten into the template.

A repo with no Collaboration section is never reported as stale. Collaboration
is opt-in, and its absence is a choice.

This schema version is deliberately separate from the package version: it bumps
only when the onboarding interview changes, so a release that touches only
wording never prompts a pointless re-onboarding.
```

- [ ] **Step 5: Verify the README claims match reality**

Run: `grep -c "Two skills ship here" README.md ; echo "expect 0"`

Expected: `0`.

Run: `grep -o "redmine-[a-z-]*" README.md | sort -u`

Expected: all five skill names present — `redmine-collab-onboarding`,
`redmine-coordinator`, `redmine-onboarding`, `redmine-reviewer`,
`redmine-tracking` (plus `redmine-superpower`, the repo name).

- [ ] **Step 6: Commit**

```bash
git add README.md
git commit -m "docs: cover all five skills, fix install recipe, document versioning"
```

---

### Task 6: Reword the REST-API limitation (SEPARATE COMMIT)

**Files:**
- Modify: `skills/redmine-onboarding/SKILL.md` — the NEEDS ADMIN bullet in the
  `## Report` section (was line 84 before Task 1; Task 1 inserts two lines
  above it, so locate it by the text `the REST API`, not by line number)
- Modify: `skills/redmine-collab-onboarding/SKILL.md` — the NEEDS ADMIN bullet
  in the `## Report` section (same shift; locate by text)
- Modify: `README.md` — the agent-account bullet in the `## Requirements`
  section (locate by the text `the REST API`)

**Interfaces:** none. This task is independent of Tasks 1–5 and shares only the
branch. Do not fold it into another commit.

**Why:** the current wording says "the REST API cannot create or modify any of
these", which is wrong — the API can, given an admin token. What actually
constrains the agent is its own non-admin access. Both skills are reworded
together to keep the pair consistent.

- [ ] **Step 1: Reword in redmine-onboarding**

The line currently reads:

```markdown
  custom fields, roles, workflow grid, instance settings — the REST API
  cannot create or modify any of these), phrased as concrete steps.
```

Change to:

```markdown
  custom fields, roles, workflow grid, instance settings — the agent's
  non-admin API access cannot create or modify any of these), phrased as
  concrete steps.
```

- [ ] **Step 2: Reword in redmine-collab-onboarding**

The line currently reads:

```markdown
  accounts, project memberships, roles and permissions — the REST API
  cannot create or modify any of these), phrased as concrete steps.
```

Change to:

```markdown
  accounts, project memberships, roles and permissions — the agent's
  non-admin API access cannot create or modify any of these), phrased as
  concrete steps.
```

- [ ] **Step 3: Reword in the README**

In the `## Requirements` section, the agent-account bullet currently reads:

```markdown
  fields — but cannot delete or close, and has no project admin. The REST API
  cannot create trackers, statuses, custom fields, roles, or workflow grids;
  those are set up once in the Redmine web UI (onboarding tells you exactly
  what's missing).
```

Change to:

```markdown
  fields — but cannot delete or close, and has no project admin. The agent's
  non-admin API access cannot create trackers, statuses, custom fields, roles,
  or workflow grids; those are set up once in the Redmine web UI (onboarding
  tells you exactly what's missing).
```

- [ ] **Step 4: Verify the old phrasing is gone repo-wide**

Run: `grep -rn "the REST API" --include="*.md" . ; echo "exit=$?"`

Expected: `exit=1` (no matches anywhere — skills, README, and template).

Note: `docs/` contains historical spec and plan files from the previous
feature. If any of those match, leave them alone — they are a record of what
was written at the time, not live instructions. Report them instead of editing.

Run: `grep -rn "non-admin API access" --include="*.md" . | grep -v "^./docs/" | wc -l`

Expected: `3` — the two skills and the README.

- [ ] **Step 5: Commit**

```bash
git add skills/redmine-onboarding/SKILL.md skills/redmine-collab-onboarding/SKILL.md README.md
git commit -m "docs: attribute the setup limitation to non-admin access, not the API"
```

---

## Final verification

- [ ] **Run the full leak check across everything committed on this branch**

Using the same locally-built pattern as Task 2 Step 4, which is never committed:

```bash
PRIVATE='<name1>|<name2>|<host.example.com>'   # fill in locally, never commit
git diff main...HEAD | grep -niE "$PRIVATE" ; echo "exit=$?"
```

Expected: `exit=1`.

- [ ] **Confirm the commit shape**

```bash
git log --oneline main..HEAD
```

Expected: six commits — the spec, then Tasks 1–5, then Task 6 last and alone.

- [ ] **Confirm tracking still carries no instruction to read a changelog**

```bash
grep -in "changelog" skills/redmine-tracking/SKILL.md
```

Expected: exactly ONE match — the prohibition, "NEVER opens a changelog".
