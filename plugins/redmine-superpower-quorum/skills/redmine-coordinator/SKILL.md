---
name: redmine-coordinator
description: 'Use when the human starts a collaborative ticket session as the coordinator, in a repo whose conventions/redmine.md has a Collaboration section — e.g. "work #N with reviewers", "coordinate ticket 42", "run 42 with the reviewer models". Coordinator side only; the reviewer side of the same session uses redmine-reviewer.'
---

# Redmine Coordinator

**Release:** 2 (conventions schema version)

## Core principle

Run superpowers normally; re-route the human's conversational seat to the
owning ticket's journal; enforce the human's authorization gates; keep
the session alive with a standing watcher.

Read `collab-protocol.md` in full before anything else. It sits in
`shared/` alongside the skill folders — `../shared/collab-protocol.md`
from this skill's own directory. Installed as part of the
`redmine-superpower-quorum` plugin, `shared/` ships with the skills
automatically. If this skill folder is installed via symlink, resolve the
folder's real path first (`readlink -f`) and find `../shared/` from there;
if installed by copying, `shared/` must have been copied alongside the
skill folders.

Where this skill touches shared rules it cites `collab-protocol.md` §N;
on any divergence the protocol file is authoritative.

## Preconditions

- The conventions file exists with a Collaboration section — look at
  `conventions/redmine.md`, then `redmine-conventions.md` at the repo root,
  and take the first that resolves. If neither resolves, or the section is
  missing, stop and point the human at `redmine-collab-onboarding`; never
  limp along on guessed conventions.
- Reviewer sessions are expected to be starting — the human's kickoff
  names or implies a roster.

## Session setup

Do these in order; each step depends on the one before it.

1. **Read the rulebook and the conventions.** `collab-protocol.md` in
   full, then the Collaboration section of `conventions/redmine.md`:
   `coordinator_account`, `reviewer_accounts`, `human_account`,
   `signature_format`, `persona_defaults`, `gate_defaults`,
   `authorization_channels`, `round_budget`, `notification_map`,
   `recognized_commenters`, `authoritative_files`.
2. **Tracker orientation.** Read the owning ticket verbatim, its
   relations, and the project ticket list including recently closed
   tickets — resolved tickets carry decisions an open-only view misses
   (`collab-protocol.md` §4, ownership search).
3. **Post the contract journal.** If the ticket already carries a
   contract journal, this session is a resume, not a kickoff — follow
   `collab-protocol.md` §10 in place of steps 3–5; never post a second
   contract. Otherwise, copy the template from
   `collab-protocol.md` §2 and fill it in: defaults from the conventions
   fields (`gate_defaults`, `authorization_channels`, `round_budget`,
   `notification_map`), overrides from the human's kickoff words.
4. **Read back your own post and arm the standing watcher** with that
   journal ID as baseline, per the recipe in `collab-protocol.md` §6,
   using your harness's recurring scheduler (e.g. Claude Code:
   CronCreate).
5. **Wait for every rostered reviewer's confirmation** before posting
   any design question. A silent reviewer is the stall path of
   `collab-protocol.md` §6: nudge on the ticket, then escalate to the
   human through the §9 channels — never start the dialogue into an
   empty room (`collab-protocol.md` §2).

## The rewire rule

Wherever an active superpowers skill would ask the human a non-gate
question — brainstorming's clarifying questions, plan review,
code-review requests — post it to the ticket for the reviewers instead,
with evidence and a recommendation. Gate stops and scope/product
decisions still go to the human, even with every gate waived
(`collab-protocol.md` §3). The routing test for every outbound question:
"is this a decision the contract reserves to the human?" If yes, human;
if no, ticket. Chat stays the human's channel; the ticket is the shared
record (`collab-protocol.md` §1).

## Phase flow

| Phase | Superpowers skill | Ticket behavior |
|---|---|---|
| Design dialogue | brainstorming | questions → ticket; dialogue to per-reviewer agreement; Gate A |
| Spec | (spec authoring) | commit spec, post SHA; dialogue to agreement |
| Plan | writing-plans | commit plan, post SHA; dialogue to agreement; Gate B |
| Execution | subagent-driven-development | stage evidence posts; slow cadence (§6) |
| PR/review | finishing-a-development-branch | PR opened; review chain (§5); Gate C |

**Phase-boundary recaps are a coordinator duty.** At every phase
transition, post the compact current-state recap of
`collab-protocol.md` §10 before moving on. `redmine-tracking`
bookkeeping rides along untouched: keep performing it as a side effect
of each phase, exactly as that skill directs.

## Finding handling

Respond to every reviewer finding individually — never in bulk — using
exactly this vocabulary: **accepted / accepted-with-modification /
rejected-with-evidence / needs-clarification**. Verify each finding
against source before accepting it (`collab-protocol.md` §4;
superpowers:receiving-code-review applies — agreement is earned by
evidence, not deference). Findings from `recognized_commenters` accounts
get the same individual disposition but carry no agreement standing
(`collab-protocol.md` §8).

## Gate conduct

`collab-protocol.md` §3, operationalized:

- **Quotability check.** Before crossing any gate, quote the human's
  authorizing words and cite their journal ID, or cite the waiver
  recorded in the contract — whether from conventions defaults or a
  mid-session grant (`collab-protocol.md` §3). If you can do neither,
  you do not have the authorization.
- **SHA-bound authorization.** The authorization names the SHA it
  covers; new commits on the branch under review void reviewer
  agreement. At Gate C, immediately before any merge, check both
  parents — HEAD equals the authorized head, and the target branch tip
  still matches the tip recorded at agreement time — per
  `collab-protocol.md` §3.
- **Halt and hand over.** On agreement at a gated phase: post the halt
  ("halting at Gate X, awaiting human"), notify the human through the
  conventions `notification_map` channels (`collab-protocol.md` §9),
  verify the standing watcher is live (`collab-protocol.md` §6), then
  end the turn. While halted, a watcher firing that reveals only peer
  journals never crosses the gate; a journal authored by the rostered
  human account, or a human chat message, does.
- **Corrections record.** Record your own errors in the artifact with
  their cause, as a running corrections note, so the record shows what
  changed and why.

**Human interrupts.** The human's words always win, at any moment, over
any protocol state. Record the instruction on the ticket, quoted, before
acting on it, so reviewers see the same authority state you do.
