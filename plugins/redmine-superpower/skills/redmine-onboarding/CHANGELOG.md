# redmine-onboarding — conventions changelog

Read by this skill during a RE-VERIFY upgrade, and ONLY when the file's
`conventions_version:` differs from this skill's declared release. Each entry
records what the interview gained, so an upgrade asks only about genuinely new
fields.

Covers the BASE conventions only. The `## Collaboration (optional)` section has
its own changelog at `skills/redmine-collab-onboarding/CHANGELOG.md`.

## unversioned → 1

A file with no `conventions_version:` line was written before versioning
existed.

**Base conventions: no new fields.** A pre-v1 base section is already complete.
The upgrade adds the `conventions_version: 1` stamp and asks nothing.

**The first major change (PR #1): multi-model collaboration.** This is the
bolt-on's first feature beyond solo tracking. A coordinator session and one or
more reviewer sessions take a ticket through
design → spec → plan → implementation → PR review, conversing in the ticket
journal under explicit human authorization gates. It adds three skills —
`redmine-coordinator`, `redmine-reviewer`, `redmine-collab-onboarding` — a shared
rulebook at `skills/shared/collab-protocol.md`, and an optional
`## Collaboration (optional)` section to the conventions file.

It is opt-in and changes nothing about base tracking: a repo that never runs two
models needs none of it, and its base section stays complete as written. To
adopt it, run `redmine-collab-onboarding`, which owns and versions the
Collaboration section; base onboarding never touches it.

**Format note.** Pre-v1 files predate the current template and may use prose,
tables, or different field names. Preserve the file's existing style when
stamping it. Do not reformat it into the template.
