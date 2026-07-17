# Redmine × Superpowers — Design Document

Status: signed off section-by-section in planning conversation, 2026-07-17.
Companion files in this repo: `skills/redmine-tracking/SKILL.md` (the skill)
and `skills/redmine-onboarding/SKILL.md` (which subsumes what were once a
separate instance-setup checklist and build brief).

## 1. The inversion

A pipeline-first approach bolts superpowers onto a ticket system; this
design inverts that: **superpowers is the workflow; Redmine is its memory.** One skill
(`redmine-tracking`) observes which superpowers skill is active and does
ticket bookkeeping as a side effect — never as a gate. Status follows
phase; the human never manually assigns status in a session.

Design principle throughout: **the skill never creates its own stopping
points; it piggybacks on the ones superpowers already has** (brainstorm
opening exchange, design approval, spec review, execution handoff,
completion menu, BLOCKED escalation). Comments are free; questions are
expensive.

Source of truth: skill texts read in full from obra/superpowers `main` on
2026-07-17 — brainstorming, writing-plans, executing-plans,
subagent-driven-development, finishing-a-development-branch; plus
verification-before-completion, using-git-worktrees, requesting-code-review,
systematic-debugging (excerpts). Onboarding's first-run verification diffs
the locally installed versions against these.

## 2. Lifecycle map (signed)

| Phase | Ticket bookkeeping | Human stop? |
|---|---|---|
| Brainstorm start | Ask once: "Want a ticket?" (deliberate — human plans features with no code). Yes → find-or-create, parent check, "brainstorm started" comment, status New. No → zero Redmine writes. | Rides brainstorming's own opening exchange |
| Design approved | Create deferred ticket if needed (+ parent ask). Design summary comment + spec path. Create branch `task-<id>-<slug>` + worktree, set `branch` field, commit spec to the branch. | Superpowers' own design-approval stop |
| Design outcome "don't build" | Design summary + reasoning comment. Ask: Rejected or leave New. | Same stop |
| Plan saved (writing-plans) | Plan committed to ticket's branch — NO PR. Comment: plan path + task count. Status stays New. | Superpowers' spec-review / handoff stops |
| Execution (SDD; executing-plans as no-subagent fallback) | In Progress at first dispatch. One comment per completed plan task (name, tests, review) + `done_ratio` on the same write. BLOCKED → comment with exact question, status Feedback; answered → In Progress. | Only BLOCKED — the interruption is the point |
| Debugging (systematic-debugging) | Ticket established at SESSION START, before investigation. In Progress at first repo write. Resolution: root-cause + evidence comment. Escalation stop → Feedback. | Debug session start; escalation |
| Completion (finishing-a-development-branch) | 4-option menu kept, PR annotated as default. PR → evidence comment, `pr` field set, Resolved. Merge-local → evidence, Resolved. Keep-as-is → comment, stays In Progress. Discard → comment, ask human about ticket. | The skill's own menu |

Second entry point: bugs route to systematic-debugging, not brainstorming —
no spec/plan for most bugs; floor is one comment in, one comment out. All
other superpowers skills are interior machinery (TDD,
verification-before-completion, requesting-code-review, using-git-worktrees:
no direct ticket writes) or informational (no repo write → no ticket).
Default row for unknown/future skills: repo-modifying + no ticket attached
→ treat like debugging.

## 3. Rules (signed)

- **Ticket rule (absolute):** any repo modification requires a ticket;
  informational-only work requires none. Gate = first repo write (debugging
  excepted: ticket at session start). No "just fix it" escape hatch — if
  the repo is touched, there is a record.
- **Placement invariant:** every Task has a parent — one of the project's
  Epics (a closed set the human defines) or a parent Task. Epic set is
  closed; skill never creates Epics; resolves them by subject, never
  hardcoded ID. Missing parent → ask, never guess.
- **Git redlines:** plans/specs commit to the work branch, never a PR; PRs
  only for finished tasks, created at completion; one branch per ticket,
  one worktree per branch, planning + implementation in the same worktree;
  branch recorded in the `branch` field. No further git mechanics invented.
- **Session resume:** "continue #N" → read branch field + comments + plan
  path, resume SDD from first unchecked task. No re-brainstorm, no new
  sessions requested.
- **Subagent isolation:** subagents never touch Redmine; all writes flow
  through the controller, serialized. One agent writes at a time; resuming
  sessions check the worktree for another session's uncommitted state.

## 4. Schema / setup spec (signed)

- **Trackers:** Epic (container only; never worked, never branched, no
  agent transitions), Task (one design/plan/branch/worktree/PR; nests under
  Epic or Task), SubTask (exists for manual human decomposition only; when
  encountered, treated exactly like a Task).
- **Statuses:** the six 6.x defaults, zero additions. New / In Progress /
  Feedback / Resolved (agent ceiling) / Rejected / Closed (human-only).
- **Custom fields (2):** `branch` (text; Task+SubTask), `pr` (text, PR URL
  at completion; Task+SubTask). Both fields, not buried notes —
  findability.
- **Agent transition whitelist (complete):** New→In Progress,
  In Progress→Feedback, Feedback→In Progress, In Progress→Resolved,
  Feedback→Resolved (only when the blocker's answer completes the work),
  Resolved→In Progress (PR rework), New→Rejected (explicit human
  instruction only). Nothing else is wired for the Agent role; Closed is
  mechanically unreachable.
- **Roles:** human keeps full rights. Agent role: view/add/edit issues, add
  notes, manage subtasks/relations, set custom fields; no delete, no
  project admin. One non-admin API user.
- **Instance setting:** done ratio calculated from the issue field (not
  status), or `done_ratio` writes are silently ignored.

## 5. Decisions log (with durable rationale)

1. **Comments, not SubTask mirroring.** The two artifacts already have
   distinct owners: the plan file's checkboxes on the branch are the
   structured progress record, versioned with the code; Redmine's journal
   is the narrative record. Mirroring creates a third copy owned by the
   tracker, guaranteed to drift when plans are amended mid-execution.
   Comments-only means Redmine never claims to know the plan's structure,
   only its history — the memory-not-workflow inversion one level down.
   Also ~3x cheaper on a slow MCP. `done_ratio` gives the progress bar.
2. **Brainstorm asks ticket-or-not.** Deliberate, human-requested: the
   pipeline doubles as a no-code feature-planning tool.
3. **Debug tickets at session start.** Debugging nearly always writes;
   session start is the natural single exchange for ticket + parent,
   avoiding a mid-investigation interrupt.
4. **Scoped verification.** Read-back verify only for workflow-gated /
   silently-ignorable writes (status, custom fields, parent, done_ratio
   once); trust 2xx for comments. mcp-redmine round-trips are slow; don't
   tax the hottest loop.
5. **Completion menu kept, PR default.** finishing-a-development-branch's
   4-option stop is a natural stopping point; options annotated with
   ticket consequences rather than pinned to PR-only.
6. **Rejected:** one narrow agent transition (New→Rejected, explicit human
   say at design review); otherwise human-only like Closed.

## 6. Testing & promotion boundary

All testing runs against a sandbox project and a throwaway git repo;
fixtures labeled as fixtures. Nothing touches the real product repo or real
backlog until the human explicitly promotes the skill. Prod is for prod.
