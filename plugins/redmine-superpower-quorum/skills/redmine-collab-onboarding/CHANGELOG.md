# redmine-collab-onboarding — collaboration changelog

Read by this skill during a RE-VERIFY upgrade, and ONLY when the section's
`collab_version:` differs from this skill's declared release.

Covers the `## Collaboration (optional)` section only. The base conventions have
their own changelog: the `CHANGELOG.md` in the `redmine-onboarding` skill
folder, shipped in the base `redmine-superpower` plugin.

## 1 → 2

**Collaboration conventions: no new fields.** The upgrade asks nothing and
updates only the `collab_version` stamp.

**Skill selection.** Quoted YAML descriptions prevent ticket examples such as
`#N` from truncating skill metadata. `redmine-collab-onboarding` now recognizes
requests to update collaboration configuration, add reviewer models, change
review personas, or re-verify authorization gates. Coordinator and reviewer
descriptions now recognize more natural role-specific session requests and
distinguish the two sides explicitly.

**Account verification.** A successful 2xx response to the labeled-fixture
comment write is treated as success. Collab onboarding no longer reads the
comment back solely to confirm that write. Reads used to establish watcher
state during live collaboration are unchanged.

## unversioned → 1

The Collaboration section arrived in PR #1 — the first major change to the
bolt-on — before versioning existed, so a section with no `collab_version:` line
predates the stamp but already carries the full v1 interview: the coordinator
and reviewer accounts, the human account and signature format, the persona
defaults and recognized commenters, the gate defaults and authorization
channels, the round budget, the notification map, the authoritative-files list,
and the reviewer checkout. Every one of those fields already existed then, so
the upgrade adds the `collab_version: 1` stamp and asks nothing.

A repo with NO Collaboration section is not stale — collaboration is opt-in, and
its absence is a choice. Run this skill in fresh mode to adopt it.
