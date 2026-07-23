# redmine-onboarding — conventions changelog

Read by this skill during a RE-VERIFY upgrade, and ONLY when the file's
`conventions_version:` differs from this skill's declared release. Each entry
records what the interview gained, so an upgrade asks only about genuinely new
fields.

Covers the BASE conventions only. The `## Collaboration (optional)` section has
its own changelog: the `CHANGELOG.md` in the `redmine-collab-onboarding` skill
folder, shipped in the `redmine-superpower-quorum` plugin.

## 1 → 2

**Base conventions: no new fields.** The upgrade asks nothing and updates only
the `conventions_version` stamp.

**Skill selection.** `redmine-onboarding` now recognizes natural requests to
set up, update, fix, redo, or re-verify Redmine configuration, conventions,
and tracker/status mappings, including "update my redmine config."
`redmine-tracking` now recognizes common ticket references and ticket-state
questions.

**Write handling.** Successful Redmine 2xx responses are treated as success.
Tracking no longer reads solely to confirm each status, custom-field, parent,
or progress write. It reads when current state is needed and makes one fresh
terminal read before reporting Resolved or Rejected.

**Onboarding verification.** Instance verification now exercises the normal
status, branch, PR, and `done_ratio` writes once on one labeled fixture and
trusts their successful responses. It no longer probes a transition grid or
reads each write back. The `done_ratio` instance setting remains an admin
requirement that the non-admin agent reports rather than attempting to change.

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
rulebook (`collab-protocol.md`, in the `shared/` folder alongside those skills,
all shipped in the `redmine-superpower-quorum` plugin), and an optional
`## Collaboration (optional)` section to the conventions file.

It is opt-in and changes nothing about base tracking: a repo that never runs two
models needs none of it, and its base section stays complete as written. To
adopt it, run `redmine-collab-onboarding`, which owns and versions the
Collaboration section; base onboarding never touches it.

**Format note.** Pre-v1 files predate the current template and may use prose,
tables, or different field names. Preserve the file's existing style when
stamping it. Do not reformat it into the template.
