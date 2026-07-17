# redmine-superpower

A public bolt-on for [superpowers](https://github.com/obra/superpowers) that
turns a Redmine instance into the memory for your superpowers workflow.

**The inversion:** superpowers is the workflow; Redmine is its memory. One
skill watches which superpowers skill is active and does the right ticket
bookkeeping as a side effect — status follows phase, comments accrue as work
happens, and you are never asked to push paper. It never gates development to
update a ticket, and it never asks a question superpowers wouldn't have
stopped for anyway.

Two skills ship here:

- **`redmine-tracking`** — the runtime skill. Reads a per-repo conventions
  file, then mirrors each superpowers phase (brainstorm → plan → execute →
  debug → complete) into Redmine ticket state. Comments are free; questions
  are expensive.
- **`redmine-onboarding`** — the setup skill. Run once per repo/instance
  pair. Interviews you, writes `redmine-conventions.md` at the repo root,
  and verifies the instance from the agent account — reporting anything only
  an admin can fix.

See [`docs/design.md`](docs/design.md) for the full rationale and the signed
decisions log.

## Requirements

- **Claude Code** with the [superpowers](https://github.com/obra/superpowers)
  plugin installed.
- An **mcp-redmine** connection to a Redmine instance (6.x recommended; the
  status model uses the six 6.x defaults).
- A **non-admin agent account** on that instance with an Agent role that can
  view/edit issues, add notes, manage subtasks/relations, and set custom
  fields — but cannot delete or close, and has no project admin. The REST API
  cannot create trackers, statuses, custom fields, roles, or workflow grids;
  those are set up once in the Redmine web UI (onboarding tells you exactly
  what's missing).

## Install

The skills are plain Claude Code skills — install them personally (every
project sees them) or per-repo.

**Personal (symlink, edits stay live):**

```bash
git clone https://github.com/samuelzamvil/redmine-superpower
cd redmine-superpower
ln -s "$(pwd)/skills/redmine-onboarding" ~/.claude/skills/redmine-onboarding
ln -s "$(pwd)/skills/redmine-tracking"   ~/.claude/skills/redmine-tracking
```

**Per-repo (scoped to one consuming repo):** copy the two skill folders into
that repo's `.claude/skills/`.

Personal skills live at `~/.claude/skills/<name>/SKILL.md`; project skills at
`.claude/skills/<name>/SKILL.md`. A plugin package may come later.

## Quickstart

1. In the repo you want tracked, with mcp-redmine configured, run:

   > Run redmine-onboarding.

   Answer the interview. It writes and commits `redmine-conventions.md`
   (structural mappings only — no live data) as a standalone commit on your
   default branch, then runs the instance verification.

2. Fix anything on onboarding's **NEEDS ADMIN** list in the Redmine web UI
   (workflow grid, custom fields, the `done_ratio = use the issue field`
   instance setting, etc.), then re-run onboarding until it reports clean.

3. Work normally with superpowers. `redmine-tracking` rides along: it opens
   or finds a ticket at the first repo write, advances status as you move
   through phases, records the branch and PR in custom fields, and stops at
   **Resolved** — the human always closes tickets, the agent never does.

## The conventions file

[`redmine-conventions.template.md`](redmine-conventions.template.md) documents
the one artifact onboarding produces. It holds only the structural mappings
that aren't discoverable from the API (which tracker/status plays each role,
custom field names, branch pattern). Live data — the Epic list, issue IDs —
is always queried fresh and never written to the file.

## License

[MIT](LICENSE).
