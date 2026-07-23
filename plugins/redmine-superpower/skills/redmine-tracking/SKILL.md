---
name: redmine-tracking
description: 'Use whenever any superpowers skill is active in a session that modifies the repository, or when the human references a ticket or issue number ("#123", "ticket 123", "issue 42", "continue 142"). Also use when the human asks what state a ticket is in.'
---

# Redmine Tracking for Superpowers

**Release:** 1 (conventions schema version)

## Core principle

Superpowers is the workflow; Redmine is its memory. This skill watches which
superpowers skill is running and does the right ticket bookkeeping as a side
effect. It never blocks development to do paperwork, and it never asks the
human a question that superpowers wouldn't have stopped for anyway.

**Comments are free, questions are expensive.** Write to the ticket
liberally; interrupt the human only at the stopping points listed below.

## Conventions file (read first, read-only)

At session start (before any Redmine access), read `redmine-conventions.md`
from the repo root. It supplies the structural mappings for this
repo/instance pair: project identifier, which tracker plays each role
(epic/task/subtask), which status plays each role (backlog / active /
blocked / ceiling / rejected / closed), custom field names, branch pattern.

- If the file is missing or fails a basic sanity check (project not found,
  a mapped tracker/status absent), do NOT limp along or guess: tell the
  human to run `redmine-onboarding`, and touch nothing in Redmine. This
  gate fires once per repo, ever.
- The conventions file is READ-ONLY to this skill. If mid-session the
  conventions prove wrong (a status renamed under you), stop, report, and
  point at re-running onboarding. Never edit the file on a work branch.
- LIVE DATA never comes from the file: the Epic list is whatever exists in
  Redmine with the epic-role tracker, queried fresh. Adding an Epic in
  Redmine requires no conventions change.

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
| Collaboration section present, no `collab_version:` found | "Collaboration section predates v1. Run `redmine-collab-onboarding` to update." |
| Collaboration section present, `collab_version:` lower than this release | "Collaboration section at vN, skills at vM. Run `redmine-collab-onboarding` to update." |
| Collaboration section present, `collab_version:` higher than this release | "Collaboration section at vN, skills at vM — this install is older than the file. Update the installed skills." |
| Conventions file missing or unreadable | Existing behavior above. Add nothing here. |

At most one line per section, at most once per session — a repo can be stale in
both the base and the Collaboration section, which is two lines total, said
once. Never repeat them later in the session.

## The ticket rule

- Any repository modification requires a ticket. Informational-only work
  (reading, explaining, discussing, running tests without changes) requires
  none.
- The gate fires at the FIRST WRITE to the repo, not at conversation start —
  with one exception: DEBUGGING SESSIONS ESTABLISH THEIR TICKET AT SESSION
  START, before investigation begins. Debugging nearly always ends in a repo
  write, and session start is the natural moment to settle ticket and parent
  in one exchange rather than interrupting mid-investigation.
- This rule is absolute by design. If the repo is touched, there is a
  record. Do not honor "no ticket, just fix it" — acknowledge, create the
  ticket, and proceed; the ticket costs the human nothing.
- SINGLE EXEMPTION: `redmine-onboarding`'s conventions-file commit — a
  standalone, human-approved commit of only `redmine-conventions.md` on the
  default branch, never on a work branch, never bundled with other changes.
  Nothing else is exempt.
- Brainstorming is the exception that asks: at brainstorm start, ask the
  human once — "Want a ticket for this?" This ask is DELIBERATE, not
  removable friction: the human uses brainstorming for no-code feature
  planning and decides per-session whether it deserves a record. The human may brainstorm features
  with no ticket and no repo writes. If a no-ticket brainstorm later reaches
  design approval and spec commit, create the ticket at that moment and post
  the design summary retroactively as its first comment.

## Placement invariant

Every Task has a parent before work proceeds. The parent is either one of
the Epics or another Task.

- Epics are a closed set created by the human. NEVER create an Epic. Resolve
  Epics by subject search, never by hardcoded ID.
- On find-or-create, check `parent_issue_id`. If missing, ask the human
  which parent the ticket belongs under (bundle this ask into an existing
  stopping point when possible). Never guess placement; never proceed with
  an orphaned Task.
- Find-or-create means SEARCH FIRST: query open Tasks by subject keywords
  before creating anything. If the human names a ticket number, use it.

## Git redlines (non-negotiable)

- Plans and specs are committed to the ticket's work branch. NEVER create a
  PR for a plan or spec. PRs are reserved for finished tasks, created at
  completion only.
- One branch per ticket, one worktree per branch. Planning and
  implementation both happen in that same worktree. Never create separate
  worktrees for planning vs implementation.
- Branch naming: `task-<ticket-id>-<slug>` (e.g. `task-142-depth-mapping`).
- The branch is recorded in the ticket's `branch` custom field the moment
  the branch is created — not buried in a note.
- No git mechanics beyond these rules are defined here. Commit conventions,
  merge strategy, hooks: ask the human explicitly; do not invent.

## Status model

Statuses are the Redmine 6.x defaults. Status follows phase; the human never
manually assigns one during a session.

| Status      | Meaning in this workflow                         | Set by |
|-------------|--------------------------------------------------|--------|
| New         | Created / brainstorming / planning               | agent  |
| In Progress | Execution underway (first implementer dispatch)  | agent  |
| Feedback    | Blocked on the human                             | agent  |
| Resolved    | Verified, PR created — agent's ceiling           | agent  |
| Rejected    | Idea declined — agent only on explicit human say | both   |
| Closed      | Human-only. The agent NEVER closes a ticket.     | human  |

**Complete agent transition whitelist (Task/SubTask only; no Epic
transitions ever):**
New→In Progress, In Progress→Feedback, Feedback→In Progress,
In Progress→Resolved, Feedback→Resolved (legal only when the human's answer
to a blocker itself completes the work — e.g. "that behavior is fine, ship
it"; do not remove this edge), Resolved→In Progress (rework after PR
review), New→Rejected (only on explicit human instruction at design
review). Any transition not on this list is forbidden — do not attempt it.

## Read only when state is needed

Treat every successful 2xx Redmine response as success. Never issue a read
solely to confirm a successful write.

Read a ticket only when its current state is needed for the next action, when
the human asks for it, or once at a terminal boundary. Before reporting a
ticket Resolved or Rejected, fetch it once and confirm its current status. Use
that same response for the final report. If the status differs, report the
observed state without speculating about the cause or retrying automatically.

Read and refusal paths never mutate.

## Routing table: superpowers phase → ticket bookkeeping

| Active skill / event                  | Ticket action |
|---------------------------------------|---------------|
| brainstorming (start)                 | Ask: ticket or not. If yes: find-or-create, parent check, comment "brainstorm started: <one-line intent>". Status New. |
| brainstorming (design approved)       | Create deferred ticket if needed (+ parent ask). Comment: design summary + spec path. Create branch + worktree, set `branch` field, commit spec to the branch. |
| brainstorming (outcome: don't build)  | Comment: design summary + reasoning. Ask the human: mark Rejected, or leave New? |
| writing-plans (plan saved)            | Commit plan to the ticket's branch (NO PR). Comment: plan path + task count. Status stays New. |
| subagent-driven-development (start)   | Status → In Progress at first implementer dispatch. |
| SDD (each plan task completes)        | Comment: task name, test results, review outcome. One comment per plan task; update `done_ratio` on the same write. |
| SDD (BLOCKED, unresolvable)           | Comment: the exact blocking question + what was tried. Status → Feedback. On answer: → In Progress. |
| executing-plans (no-subagent fallback)| Identical bookkeeping to SDD, minus dispatch. |
| systematic-debugging (session start)  | Find-or-create (parent check) BEFORE investigation begins. Status → In Progress at first repo write. |
| systematic-debugging (resolved)       | Comment: root cause + verification evidence. Then completion flow. |
| systematic-debugging (escalation stop)| Comment: hypotheses tried, findings. Status → Feedback. |
| finishing-a-development-branch        | See Completion below. |
| test-driven-development, requesting-code-review, verification-before-completion, using-git-worktrees | Interior machinery. No direct ticket writes; their outputs surface in the comments above. |
| using-superpowers, writing-skills, informational work | No repo writes → no ticket. |
| ANY OTHER / FUTURE skill              | Default rule: if it modifies the repo and no ticket is attached, treat like systematic-debugging — ticket at first write, minimal comments. |

## SubTask policy (deliberate decision)

Plan progress is tracked as COMMENTS on the Task, not mirrored SubTasks.
The two artifacts already have distinct owners: SDD maintains the plan
file's checkboxes on the branch — the structured progress record, versioned
with the code it tracks — while Redmine's journal is the narrative record.
SubTask mirroring would create a third copy of the same state, owned by the
tracker, guaranteed to drift the first time a plan is amended
mid-execution. Comments-only means Redmine never claims to know the plan's
STRUCTURE, only its HISTORY — the "Redmine is memory, not workflow"
inversion applied one level down. It is also ~3x cheaper in API writes on a
slow MCP and avoids per-SubTask closing ceremony.

The skill updates the Task's `done_ratio` as plan tasks complete, so the
issue list shows a progress bar without SubTask machinery. This requires the
instance setting "calculate done ratio = use the issue field." The setup
checklist calls out this setting; a non-admin account cannot change it.

SubTasks remain a MANUAL tool for the human's own decomposition; the skill
never creates them. When the skill ENCOUNTERS a human-created SubTask, it
treats it exactly like a Task — a ticket with a parent; the routing table
does not care about depth. Do not "improve" any of this into automatic
mirroring without a signed design change.

## Completion

When finishing-a-development-branch presents its menu, annotate each option
with its ticket consequence. PR is the recommended default for Tasks.

1. **Push + create PR (default):** comment with verification evidence
   (actual test command + actual output) and the PR URL; set the `pr`
   custom field to the PR URL (same findability rationale as `branch` — a
   field, not a buried note); status → Resolved. The human reviews, merges,
   and Closes.
2. **Merge locally:** comment with verification evidence; status → Resolved.
3. **Keep as-is:** comment noting state; status stays In Progress.
4. **Discard:** comment noting discard and why; ask the human what to do
   with the ticket. Do not change status unilaterally.

Never claim completion without fresh verification evidence in the comment.

## Session resume

When the human says "continue #N" (or names a ticket):
1. Read the ticket: `branch` field, latest comments, plan path.
2. Check out the recorded branch's worktree. If the worktree has
   uncommitted changes from another session, stop and tell the human —
   one agent writes at a time.
3. Reload the plan; resume from the first unchecked task via SDD.
4. Do not re-brainstorm, do not re-plan, do not ask the human to open a
   new session. Resume is silent unless state is inconsistent (empty
   branch field, missing plan) — then ask.

## Subagents and Redmine

Subagents NEVER touch Redmine. All ticket reads and writes flow through the
controller session, serialized. Before any bulk creation, query what exists.

## Ground rules

- Never create backlog content the human didn't voice or approve. Test
  fixtures live in the sandbox project, labeled as fixtures.
- No testing against real repos or the real backlog. Prod is for prod.
- The human closes tickets. The agent stops at Resolved.
- Epic layer is closed; Epics are never worked directly and never get
  branches.
- Status changes happen only at the phase transitions in the routing table
  — never ask the human to pick a status.
