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

**What else shipped in this release:** multi-model collaboration —
`redmine-collab-onboarding`, `redmine-coordinator`, `redmine-reviewer`, and the
optional `## Collaboration (optional)` section of the conventions file. It is
opt-in: a repo that never runs two models needs none of it. To adopt it, run
`redmine-collab-onboarding`.

**Format note.** Pre-v1 files predate the current template and may use prose,
tables, or different field names. Preserve the file's existing style when
stamping it. Do not reformat it into the template.
