# redmine-superpower

A public bolt-on for [superpowers](https://github.com/obra/superpowers) that
turns a Redmine instance into the memory for your superpowers workflow.

> **⚠️ Status: experiment, not an endorsement.** This is a personal experiment,
> published in the open so others can follow along — not a recommendation to
> adopt it. **Initial impressions are still out:** I'm partway through
> onboarding, which is a long process, and I don't yet have enough real usage
> to say whether the approach holds up.
>
> What I can report so far: the Redmine MCP is **slow** — operations take a
> while — but so far it has been **consistent and error-free**. I'm also
> running an **old Redmine 5** instance, whereas the skills target the 6.x
> status defaults; expect rough edges until this is exercised against 6.x.

**The inversion:** superpowers is the workflow; Redmine is its memory. One
skill watches which superpowers skill is active and does the right ticket
bookkeeping as a side effect — status follows phase, comments accrue as work
happens, and you are never asked to push paper. It never gates development to
update a ticket, and it never asks a question superpowers wouldn't have
stopped for anyway.

Five skills ship here, packaged as two plugins. Two are the core (the
**`redmine-superpower`** plugin):

- **`redmine-tracking`** — the runtime skill. Reads a per-repo conventions
  file, then mirrors each superpowers phase (brainstorm → plan → execute →
  debug → complete) into Redmine ticket state. Comments are free; questions
  are expensive.
- **`redmine-onboarding`** — the setup skill. Run once per repo/instance
  pair. Interviews you, writes `redmine-conventions.md` at the repo root,
  and verifies the instance from the agent account — reporting anything only
  an admin can fix.

Three more are optional — the **`redmine-superpower-quorum`** plugin, for
running two models against one ticket:

- **`redmine-collab-onboarding`** — appends the Collaboration section to your
  conventions file and verifies each declared agent account.
- **`redmine-coordinator`** — runs the superpowers workflow while routing
  questions to model reviewers through the ticket journal.
- **`redmine-reviewer`** — builds an independent prior and challenges the
  coordinator's claims through ticket comments.

See [`docs/design.md`](docs/design.md) for the full rationale and the signed
decisions log.

## Requirements

- **Claude Code** with the [superpowers](https://github.com/obra/superpowers)
  plugin installed.
- An **mcp-redmine** connection to a Redmine instance (6.x recommended; the
  status model uses the six 6.x defaults).
- A **non-admin agent account** on that instance with an Agent role that can
  view/edit issues, add notes, manage subtasks/relations, and set custom
  fields — but cannot delete or close, and has no project admin. The agent's
  non-admin API access cannot create trackers, statuses, custom fields, roles,
  or workflow grids; those are set up once in the Redmine web UI (onboarding
  tells you exactly what's missing).

## Install

Two plugins ship from this repo's marketplace (`samuelzamvil`):

- **`redmine-superpower`** — the base: `redmine-onboarding` + `redmine-tracking`.
- **`redmine-superpower-quorum`** — the multi-model add-on:
  `redmine-collab-onboarding` + `redmine-coordinator` + `redmine-reviewer`. It
  depends on the base, so installing it pulls the base in too.

**From the marketplace (recommended):**

```bash
/plugin marketplace add samuelzamvil/redmine-superpower
/plugin install redmine-superpower@samuelzamvil          # base only
/plugin install redmine-superpower-quorum@samuelzamvil   # add-on + base
```

Or run `marketplace add`, then browse and click through the interactive
`/plugin` menu.

**Direct zip import (single session):** download the per-plugin zips from a
[release](https://github.com/samuelzamvil/redmine-superpower/releases) and load
them for one session:

```bash
claude --plugin-dir ./redmine-superpower-v1.0.0.zip
# or fetch by URL:
claude --plugin-url https://github.com/samuelzamvil/redmine-superpower/releases/download/redmine-superpower-v1.0.0/redmine-superpower-v1.0.0.zip
```

Zip loading is per-session and does **no** dependency resolution, so to use the
quorum add-on this way, load **both** zips:

```bash
claude --plugin-dir ./redmine-superpower-v1.0.0.zip \
       --plugin-dir ./redmine-superpower-quorum-v1.0.0.zip
```

**Manual / dev (symlink, edits stay live):** the plugins are plain skill folders
under `plugins/<plugin>/skills/`, so you can still symlink or copy individual
skills:

```bash
git clone https://github.com/samuelzamvil/redmine-superpower
cd redmine-superpower
ln -s "$(pwd)/plugins/redmine-superpower/skills/redmine-onboarding" ~/.claude/skills/redmine-onboarding
ln -s "$(pwd)/plugins/redmine-superpower/skills/redmine-tracking"   ~/.claude/skills/redmine-tracking
```

For a copy install of any quorum skill, copy that plugin's `skills/shared/`
folder alongside it — the collaboration skills read `shared/collab-protocol.md`.
Symlink installs resolve it automatically from the link's real path.

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

## Versioning

Each skill declares a **conventions schema version** — the `Release:` line under
its title. `redmine-conventions.md` records the version it was written under:
`conventions_version:` for the base mappings, `collab_version:` inside the
Collaboration section.

When they diverge, `redmine-tracking` says so at session start — at most one
line per stale section — and names the skill to re-run. If your file is newer
than your installed skills, it says that instead: update the install rather
than re-running anything. It never blocks work. Re-running that skill reads its
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
wording never prompts a pointless re-onboarding. Both plugins currently ship at
package version `1.0.0`, matching the skills' `Release: 1`; the two version
lines are independent and will drift as either changes.

## License

[MIT](LICENSE).
