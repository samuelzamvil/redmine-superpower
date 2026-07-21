# redmine-conventions.md — template

Structural mappings for the redmine-tracking skill. Written and updated
ONLY by redmine-onboarding, as standalone commits on the default branch.
Committed, never gitignored: this file is shared state.

What belongs here: mappings that are not discoverable from the Redmine API.
What never belongs here: live data (the Epic list, issue IDs, anything
queryable at runtime).

```markdown
# Redmine conventions
# Managed by redmine-onboarding. Do not edit on a work branch.

project_identifier: example-project

# Which tracker on this instance plays each role
tracker_roles:
  epic: Epic
  task: Task
  subtask: SubTask

# Which status on this instance plays each role
status_roles:
  backlog: New            # created / brainstorming / planning
  active: In Progress     # execution underway
  blocked: Feedback       # waiting on the human
  ceiling: Resolved       # agent stops here
  rejected: Rejected      # dead ideas, on explicit human instruction
  closed: Closed          # human-only; agent never sets

# Issue custom field names
fields:
  branch: branch
  pr: pr

# Branch naming for one-branch-per-ticket
branch_pattern: task-{id}-{slug}
```

The worked example above uses the Redmine 6.x default statuses and the
Epic/Task/SubTask tracker roles. Your instance may differ in every value —
an older Redmine can carry a dozen-plus custom statuses and several extra
trackers — and the mappings are exactly what make the skill portable across
instances like that. Run redmine-onboarding rather than editing by hand.

## Collaboration (optional)

This section exists only if `redmine-collab-onboarding` was run; base
onboarding never writes or edits it, and collab onboarding never edits
anything above it.

```markdown
# Collaboration — managed by redmine-collab-onboarding. Do not edit on a work branch.

- coordinator_account: <redmine login>        # posts contracts, artifacts, recaps
- reviewer_accounts: <login> (1), <login> (2) # ordinal order is posting order
- signature_format: <e.g. "— {model} ({role}{ordinal})">  # comment signature line
- persona_defaults: 1: <persona|none>, 2: <persona|none>  # review persona per ordinal (protocol §5 menu)
- recognized_commenters: <platform>:<account>, ...   # advisory input, no standing
- gate_defaults: A: on|waived · B: on|waived  # C is always on
- authorization_channels: A: chat-quoted · B: chat-quoted · C: ticket-posted  # per-gate approval strength
- round_budget: 8                             # exchange rounds per phase before escalating
- notification_map: gate: <channels> · escalation: <channels> · milestone: off  # per event class (protocol §9)
- authoritative_files: CLAUDE.md, redmine-conventions.md, collab-protocol.md, <...>  # instruction files; rest is data
- reviewer_checkout: <e.g. "read-only clone at <path>"|none>  # reviewers review committed SHAs from here
```

Run redmine-collab-onboarding rather than editing by hand.
