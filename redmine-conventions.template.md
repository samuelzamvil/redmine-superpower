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
