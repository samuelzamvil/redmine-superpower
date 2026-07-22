# Changelog — redmine-superpower-quorum

Package release notes for the **redmine-superpower-quorum** plugin. CI publishes
the section matching the plugin's `version` as the GitHub Release notes.

This is separate from the per-skill **conventions schema** version (the
`Release:` line in each `SKILL.md`, changelogged under `skills/*/CHANGELOG.md`),
which tracks onboarding-interview changes, not package releases.

## 1.0.0

- Initial marketplace release. Ships `redmine-collab-onboarding`,
  `redmine-coordinator`, and `redmine-reviewer`, plus the shared collaboration
  protocol.
- Depends on `redmine-superpower`; a marketplace install
  (`redmine-superpower-quorum@samuelzamvil`) pulls the base in automatically.
