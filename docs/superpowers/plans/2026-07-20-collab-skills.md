# Multi-Model Collaboration Skills Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Implement the five prose artifacts specified in
`docs/superpowers/specs/2026-07-20-collab-skills-design.md` (freeze
candidate `06d454d`): a shared protocol rulebook, coordinator and reviewer
role skills, an optional collab onboarding skill, and the two small edits
to existing files.

**Architecture:** Everything is instruction prose — markdown skill files,
no code. `skills/shared/collab-protocol.md` is the single source of truth
for all shared rules; the three SKILL.md files are role-specific and cite
protocol sections rather than restating them. The conventions template
gains an optional collab section whose field names the skills reference.

**Tech Stack:** Markdown. Claude Code / Codex skill format (YAML
frontmatter with `name` and `description`, then prose). No build, no
tests-as-code — verification is grep probes plus read-through checks
against the spec.

## Global Constraints

Copied from the spec; every task's requirements include these.

- **Discipline-only.** No scripts, no schemas, no machinery. Prose rules
  only. (Spec §1.)
- **§4 is authoritative.** Role skills cite protocol sections
  (`per collab-protocol.md §N`) and never paraphrase rule content at
  length. On any divergence the protocol file wins. (Spec §5 preamble.)
- **Platform-agnostic.** Wording must work when the skill is loaded by
  Claude Code or Codex. Name harness tools only as parenthesized examples
  (e.g. "your harness's recurring scheduler (Claude Code: CronCreate)").
  (Spec §2.)
- **Protocol file resolution (verbatim in all three new SKILL.md files):**
  > Read `collab-protocol.md` in full before anything else. It lives at
  > `shared/collab-protocol.md` relative to this repo's `skills/`
  > directory. If this skill folder is installed via symlink, resolve the
  > folder's real path first (`readlink -f`) and find `../shared/` from
  > there; if installed by copying, `shared/` must have been copied
  > alongside the skill folders.
- **House style.** Match the existing skills: frontmatter `name` +
  `description` (description states trigger conditions, third person,
  "Use when…"), `#` title, `##` sections, dense imperative prose, bold
  lead-ins on bullets, ~76-char line wrap. Follow
  superpowers:writing-skills conventions when authoring.
- **Verbatim load-bearing strings** (must appear character-exact where a
  task names them): see each task's grep probes.
- **Repo hygiene:** commit after every task; never touch `README.md`
  (it has uncommitted human edits — README updates are deferred, see
  After the Plan) or `docs/design.md`.

## File Structure

```
skills/shared/collab-protocol.md        Task 1  (new — the rulebook)
redmine-conventions.template.md         Task 2  (modify — optional collab section)
skills/redmine-coordinator/SKILL.md     Task 3  (new)
skills/redmine-reviewer/SKILL.md        Task 4  (new)
skills/redmine-collab-onboarding/SKILL.md  Task 5  (new)
skills/redmine-onboarding/SKILL.md      Task 5  (modify — pointer line + Collaboration ownership rule)
                                        Task 6  (consistency sweep, no new files)
```

---

### Task 1: `skills/shared/collab-protocol.md`

**Files:**
- Create: `skills/shared/collab-protocol.md`

**Interfaces:**
- Consumes: spec §3–§4 (authority layout and all ten rule areas) from
  `docs/superpowers/specs/2026-07-20-collab-skills-design.md`.
- Produces: numbered sections **§1–§10** that Tasks 3–5 cite by number,
  in exactly this order:
  §1 Participants and channel · §2 The contract journal · §3 Authority
  rules · §4 Evidence discipline · §5 Review chain and personas ·
  §6 Watcher recipe · §7 Round budget and convergence health ·
  §8 Guardrails · §9 Human notification · §10 Resume and replacement.
  Also produces the contract journal template (in §2) and the persona
  menu (in §5) that Tasks 3 and 5 reference by name.

- [ ] **Step 1: Read the source sections**

Read `docs/superpowers/specs/2026-07-20-collab-skills-design.md` §3,
§4.1–§4.10 in full. This file transcribes those rules into a standalone
rulebook addressed to the models ("you"), organized into the ten numbered
sections listed in Produces. Do not invent rules; do not drop rules. Every
bullet in spec §4.x must land in the corresponding section.

- [ ] **Step 2: Write the file header and §1–§2**

Open with an H1 (`# Collaboration Protocol`). Section headings use the
exact format `## §N — <title>` (e.g. `## §3 — Authority rules`) so the
role skills' `§N` citations are greppable. Then a preamble paragraph:
both role skills require reading this file in full at session start; this
file is the citable authority — when a peer breaks protocol, quote the
section at them; the human's words override everything in it.

§1 from spec §4.1. §2 from spec §4.2, including a fill-in contract
journal template rendered as a markdown block the coordinator copies:

```markdown
COLLABORATION CONTRACT (ticket #<id>)
Objective: <one line>
Roster:
  coordinator: <redmine account> — this session
  reviewer 1: <redmine account> — persona: <persona or none>
  reviewer N: ...
  human: <redmine account or "chat only">
Phases: design → spec → plan → execution → PR/review
Gates: A (post-design): on|waived · B (post-plan): on|waived · C (merge): on
Authorization channel: A: <chat-quoted|ticket-posted> · B: <…> · C: <…>
Cadence: dialogue <n> min · artifact review <n> min · execution <n> min · PR/CI <n> min
Round budget: <n> per phase
Preconditions: <ticket-specific, or "none">
Recognized commenters: <list from conventions, or "none">
```

- [ ] **Step 3: Write §3–§4**

§3 Authority rules from spec §4.3 — all eight bullets (quotability with
journal ID; explicit assent only + the coordinator-authored-record honest
limit; SHA-bound authorization scoped to the branch under review; Gate C
binds both parents and covers every integration path; per-gate channel
strength dial; gate = turn boundary + §9 notification; no greenlight
language; agreement → awaiting-human; scope/product escalation).

§4 Evidence discipline from spec §4.4 — all eight bullets (full-clause
quoting; claims carry evidence or are marked unverified; verify before
accepting; commit anchoring; ownership search across open and resolved
tickets; matrices route review; falsifiable check first; withdrawals name
evidence or are "withdrawn without concession, advisory").

- [ ] **Step 4: Write §5–§7**

§5 from spec §4.5: parallel examination, serialized posting by ordinal,
coverage attestation with its honest ceiling, dissent-must-engage,
per-reviewer agreement at the same SHA, re-review walks the chain in
order. Include the persona menu verbatim: correctness skeptic,
scope/conventions auditor, security/robustness, simplicity/YAGNI,
test-adequacy — free-form allowed; a persona is a lens, not a blinder
and not an authority change.

§6 from spec §4.6: all ten watcher bullets, including the cadence table
(dialogue 1–2 min · artifact review 3–5 · execution 10–15 · PR/CI 10–15)
and the unrecoverable-error sentence verbatim. Fold in the delivery rule
from spec §8: any post that advances the turn (contract, agreement, gate
halt, review findings) gets a read-back — which doubles as the watcher
baseline; plain comments trust 2xx.

§7 from spec §4.7: round budget default 8, escalation on exhaustion,
shrinking-length health check and naming convergence theater.

- [ ] **Step 5: Write §8–§10**

§8 from spec §4.8: network allowlist; no web access without quotable
human instruction; three-tier author check (roster / recognized
commenters / everyone else); reviewers never write repo content and
review committed SHAs from their own checkout; authoritative-files
enumeration with pinned-ref rule (a branch's change to a listed file is
an artifact under review, not live instructions); subagents never touch
Redmine.

§9 from spec §4.9: event classes, conventions-mapped channels, the
Feedback-unreachable-from-New constraint, durable ticket-post twin.

§10 from spec §4.10: phase-boundary recaps (content list: phase,
artifact + SHA, open findings, gate state, next actor); resume protocol
with cursor announcement, rebuild order, HEAD verification, dirty-tree
rule (posted, never silently discarded or committed, blocks
implementation until dispositioned), watcher re-arm; never resume
silently; replacement sessions inherit only the ticket.

- [ ] **Step 6: Verify with grep probes**

Run each; expected: at least one match per probe, in this file.

```bash
f=skills/shared/collab-protocol.md
grep -c "this is not authorization to proceed; next actor: human" $f   # ≥ 1
grep -c "cite its journal ID; if you cannot, you do not have it" $f     # ≥ 1
grep -c "Deleting your watcher while in an awaiting state is the one unrecoverable process error" $f  # ≥ 1
grep -c "withdrawn without concession, advisory" $f                     # ≥ 1
grep -c "on the branch under review" $f                                 # ≥ 1
grep -c "merge queue" $f                                                # ≥ 1
grep -c "never silently discarded or silently committed" $f             # ≥ 1
grep -c "simplicity/YAGNI" $f                                           # ≥ 1
grep -Ec "^## §(10|[1-9]) — " $f                                        # = 10
```

Then a read-through against spec §4.1–§4.10: every spec bullet has a
home; no section renumbered.

- [ ] **Step 7: Commit**

```bash
git add skills/shared/collab-protocol.md
git commit -m "feat: add shared collaboration protocol rulebook"
```

---

### Task 2: conventions template collab section

**Files:**
- Modify: `redmine-conventions.template.md` (append one section at end)

**Interfaces:**
- Consumes: nothing from other tasks (field list comes from spec §4.2,
  §4.8, §4.9, §7).
- Produces: the collab section heading and **exact field names** that
  Tasks 3–5 reference: `coordinator_account`, `reviewer_accounts`,
  `human_account`, `signature_format`, `persona_defaults`,
  `recognized_commenters`, `gate_defaults`, `authorization_channels`,
  `round_budget`, `notification_map`, `authoritative_files`,
  `reviewer_checkout`.

- [ ] **Step 1: Read the existing template**

Read `redmine-conventions.template.md` in full to match its formatting
conventions (comment style, placeholder style).

- [ ] **Step 2: Append the collab section**

Append, matching the template's existing style, a section titled
`## Collaboration (optional)` opening with this exact sentence:

> This section exists only if `redmine-collab-onboarding` was run; base
> onboarding never writes or edits it, and collab onboarding never edits
> anything above it.

Then the twelve fields from Produces, each with a placeholder value and a
one-line comment stating what it configures, e.g.:

```markdown
- coordinator_account: <redmine login>        # posts contracts, artifacts, recaps
- reviewer_accounts: <login> (1), <login> (2) # ordinal order is posting order
- human_account: <redmine login|"chat only">  # the human's redmine login, or "chat only"
- signature_format: <e.g. "— {model} ({role}{ordinal})">
- persona_defaults: 1: <persona|none>, 2: <persona|none>
- recognized_commenters: <platform>:<account>, ...   # advisory input, no standing
- gate_defaults: A: on|waived · B: on|waived  # C is always on
- authorization_channels: A: chat-quoted · B: chat-quoted · C: ticket-posted
- round_budget: 8
- notification_map: gate: <channels> · escalation: <channels> · milestone: off
- authoritative_files: CLAUDE.md, redmine-conventions.md, collab-protocol.md, <...>
- reviewer_checkout: <e.g. "read-only clone at <path>"|none>
```

- [ ] **Step 3: Verify**

```bash
grep -c "## Collaboration (optional)" redmine-conventions.template.md  # = 1
grep -c "authorization_channels" redmine-conventions.template.md       # = 1
grep -c "authoritative_files" redmine-conventions.template.md          # = 1
```

Read-through: the section is append-only (git diff shows no changes above
it) and every field name in Produces appears exactly once.

- [ ] **Step 4: Commit**

```bash
git add redmine-conventions.template.md
git commit -m "feat: add optional collaboration section to conventions template"
```

---

### Task 3: `skills/redmine-coordinator/SKILL.md`

**Files:**
- Create: `skills/redmine-coordinator/SKILL.md`

**Interfaces:**
- Consumes: protocol §1–§10 (Task 1, cited by number); conventions field
  names (Task 2); spec §5.
- Produces: the skill name `redmine-coordinator` referenced by Task 5's
  onboarding text.

- [ ] **Step 1: Write frontmatter and core principle**

Frontmatter:

```yaml
---
name: redmine-coordinator
description: Use when the human starts a collaborative ticket session ("work #N with reviewers") in a repo whose redmine-conventions.md has a Collaboration section. Runs the superpowers workflow while routing non-gate questions to model reviewers through the ticket journal, enforcing human authorization gates, and keeping the session alive with a standing watcher.
---
```

Body opens: one-line mission (run superpowers normally; re-route the
human's conversational seat to the ticket; enforce gates; keep the
session alive), then the protocol-file resolution block from Global
Constraints verbatim, then: "Where this skill touches shared rules it
cites `collab-protocol.md` §N; on any divergence the protocol file is
authoritative."

- [ ] **Step 2: Write preconditions and session setup**

From spec §5. Preconditions: conventions file with Collaboration section
present — if missing, stop and point the human at
`redmine-collab-onboarding`; reviewer sessions expected to be starting.
Session setup, in order: read protocol + conventions → tracker
orientation (owning ticket verbatim, relations, project ticket list
including recently closed) → post the contract journal (template in
protocol §2, defaults from conventions, overrides from the human's
kickoff words) → read back own post, arm the standing watcher with that
journal ID (protocol §6) → wait for every rostered reviewer's
confirmation before posting design questions (silent reviewer →
protocol §6 stall path).

- [ ] **Step 3: Write the rewire rule and phase flow**

Rewire rule from spec §5 including the routing test sentence: "is this a
decision the contract reserves to the human?" Phase flow table mapping
each superpowers skill to its ticket behavior:

| Phase | Superpowers skill | Ticket behavior |
|---|---|---|
| Design dialogue | brainstorming | questions → ticket; dialogue to per-reviewer agreement; Gate A |
| Spec | (spec authoring) | commit spec, post SHA; dialogue to agreement |
| Plan | writing-plans | commit plan, post SHA; dialogue to agreement; Gate B |
| Execution | subagent-driven-development | stage evidence posts; slow cadence (§6) |
| PR/review | finishing-a-development-branch | PR opened; review chain (§5); Gate C |

Plus: phase-boundary recap (protocol §10) posts at every transition — a
coordinator duty; redmine-tracking bookkeeping rides along untouched.

- [ ] **Step 4: Write finding handling and gate conduct**

Finding handling from spec §5: respond to every finding individually
using exactly the vocabulary **accepted / accepted-with-modification /
rejected-with-evidence / needs-clarification**; verify each against
source before accepting (superpowers:receiving-code-review applies).
Gate conduct: protocol §3 operationalized — quotability check with
journal ID; SHA-bound authorization including the Gate C
head-and-base check at merge time; halt post; §9 notification; end the
turn; while halted, watcher firings and peer journals never cross the
gate. Corrections record: own errors recorded in the artifact with their
cause. Human interrupts (spec §8): human words always win; the
coordinator records the instruction on the ticket, quoted, before acting
on it, so reviewers see the same authority state.

- [ ] **Step 5: Verify**

```bash
f=skills/redmine-coordinator/SKILL.md
grep -c "collab-protocol.md" $f                          # ≥ 3 (resolution block + citations)
grep -c "rejected-with-evidence" $f                      # ≥ 1
grep -c "is this a decision the contract reserves to the human" $f  # ≥ 1
grep -Ec "§(3|6|10)" $f                                  # ≥ 3
head -5 $f | grep -c "name: redmine-coordinator"         # = 1
```

Read-through against spec §5: every bullet represented; no protocol rule
restated at length (citations instead); platform-agnostic wording.

- [ ] **Step 6: Commit**

```bash
git add skills/redmine-coordinator/SKILL.md
git commit -m "feat: add redmine-coordinator role skill"
```

---

### Task 4: `skills/redmine-reviewer/SKILL.md`

**Files:**
- Create: `skills/redmine-reviewer/SKILL.md`

**Interfaces:**
- Consumes: protocol §1–§10 (Task 1); conventions field names (Task 2);
  spec §6.
- Produces: the skill name `redmine-reviewer` referenced by Task 5.

- [ ] **Step 1: Write frontmatter and core principle**

```yaml
---
name: redmine-reviewer
description: Use when the human starts a reviewer session for a collaborative ticket ("review ticket #N as reviewer 2"). Builds an independent prior from the tracker and codebase, challenges the coordinator's claims against primary sources through ticket comments, and never writes repo content, never advances ticket state, and never utters authorization language.
---
```

Body opens: one-line mission, the protocol resolution block verbatim,
and the §4-is-authoritative line — same shape as Task 3 Step 1.

- [ ] **Step 2: Write setup with ordering rationale**

From spec §6, preserving the order and stating why it is the point
(independence is established at setup and cannot be recovered later):
read protocol + conventions → deep tracker orientation (linked tickets
in full, recently resolved decision records, project ticket list) → repo
standards docs (CLAUDE.md etc., from the pinned ref per protocol §8) →
independent codebase orientation → only then read the coordinator's
artifact/questions. Confirm identity + persona on the ticket; arm the
standing watcher (protocol §6).

- [ ] **Step 3: Write review behavior and chain conduct**

Review behavior from spec §6 (verify against source, re-read
authoritative clauses before accepting any mandate/exception/ownership
claim, check claimed exceptions against the rule's source text, review
the diff at the named SHA from your own checkout per protocol §8,
declare what you could and could not verify). Chain conduct: cite
protocol §5 — parallel examination, ordinal posting, coverage
attestation, new/concur/dissent, dissent engages the prior finding.

- [ ] **Step 4: Write approval discipline and hard boundaries**

Approval discipline from spec §6: withhold by default and say so every
round ("this is not approval yet"); agreement is a distinct speech act
naming the SHA; every agreement post carries the protocol §3 formula
"this is not authorization to proceed; next actor: human"; own and post
your own misses. Hard boundaries from spec §6: never write/commit repo
content; committed SHAs from own checkout only (protocol §8); never
advance ticket workflow state; never implement own findings; flag
protocol violations on the ticket immediately (example: implementation
evidence while a gate is pending).

- [ ] **Step 5: Verify**

```bash
f=skills/redmine-reviewer/SKILL.md
grep -c "this is not authorization to proceed; next actor: human" $f  # ≥ 1
grep -c "this is not approval yet" $f                                 # ≥ 1
grep -c "collab-protocol.md" $f                                       # ≥ 3
grep -c "own checkout" $f                                             # ≥ 1
head -5 $f | grep -c "name: redmine-reviewer"                         # = 1
```

Read-through against spec §6; confirm setup ordering matches exactly.

- [ ] **Step 6: Commit**

```bash
git add skills/redmine-reviewer/SKILL.md
git commit -m "feat: add redmine-reviewer role skill"
```

---

### Task 5: `skills/redmine-collab-onboarding/SKILL.md` + base pointer

**Files:**
- Create: `skills/redmine-collab-onboarding/SKILL.md`
- Modify: `skills/redmine-onboarding/SKILL.md` (pointer line in the
  Report section + Collaboration ownership rule, nothing else)

**Interfaces:**
- Consumes: conventions field names (Task 2, written exactly); skill
  names `redmine-coordinator`/`redmine-reviewer` (Tasks 3–4); spec §7.
- Produces: nothing consumed later.

- [ ] **Step 1: Write frontmatter and mode detection**

```yaml
---
name: redmine-collab-onboarding
description: Use when the human wants to set up multi-model collaboration for a repo ("set up multi-model collaboration"), after redmine-onboarding has passed clean. Interviews the human, appends the Collaboration section to redmine-conventions.md, and verifies each declared agent account against the instance. Optional — repos that never run two models never need it.
---
```

Mode detection mirroring base onboarding: conventions file has no
`## Collaboration (optional)` section → FRESH; section exists →
RE-VERIFY (probe, diff, update only what the human approves).
Precondition: base onboarding passed clean — if `redmine-conventions.md`
is missing entirely, stop and point at `redmine-onboarding`.

- [ ] **Step 2: Write the interview**

One question at a time, brainstorming style, proposing defaults from
what is discoverable. The items, in order, writing exactly the Task 2
field names: coordinator account; reviewer accounts and typical count
(ordinal order = posting order); the human's Redmine account or "chat
only" (`human_account` — feeds the contract roster; a ticket-posted
Gate C authorization requires it); signature format; persona defaults per
ordinal (offer the protocol §5 menu, free-form or none allowed);
recognized-commenters allowlist (platform + account); gate defaults
(A/B on|waived — C is not askable, always on); per-gate authorization
channel strength (default Gate C to ticket-posted where the human holds
an account, and say why: a fabricated Gate C authorization survives
repudiation); round budget (default 8); notification map per event class
(gate / escalation / milestone), proposing only channels the harness
actually supports; authoritative instruction files (propose CLAUDE.md,
redmine-conventions.md, collab-protocol.md; human adds or removes);
reviewer checkout convention (recommend a read-only clone or push-less
credentials where the harness allows — recommendation, not requirement).

- [ ] **Step 3: Write the write-and-verify sections**

Write: append the Collaboration section to `redmine-conventions.md`
from the template; show the human; standalone commit on the default
branch (the same sanctioned no-ticket write as base onboarding). State
the mutual-preservation rule: this skill edits only the Collaboration
section; base onboarding never touches it.

Verify (from each declared agent account where credentials allow): the
account can see the project; the account can post a comment on a labeled
fixture issue, write-then-read-back. Report ends WORKING / NEEDS ADMIN
in base-onboarding style. Read/refusal paths never mutate.

- [ ] **Step 4: Add the base-onboarding pointer line and ownership rule**

In `skills/redmine-onboarding/SKILL.md`, append to the end of the
`## Report` section (after the "Do not proceed…" paragraph):

```markdown
Optionally: to set up multi-model collaboration for this repo, run
`redmine-collab-onboarding` once this verification passes clean.
```

Also add the Collaboration ownership rule: base onboarding preserves
any `## Collaboration (optional)` section byte-for-byte — that section
belongs to `redmine-collab-onboarding` exclusively and must never be
presented as RE-VERIFY drift. Nothing else in that file changes.

- [ ] **Step 5: Verify**

```bash
f=skills/redmine-collab-onboarding/SKILL.md
grep -c "authorization_channels\|authorization channel" $f      # ≥ 1
grep -c "FRESH\|RE-VERIFY" $f                                   # ≥ 2
grep -c "NEEDS ADMIN" $f                                        # ≥ 1
grep -c "redmine-collab-onboarding" skills/redmine-onboarding/SKILL.md  # = 1
git diff --stat skills/redmine-onboarding/SKILL.md              # 1 file, ~2-3 lines added
```

Read-through against spec §7; interview items complete and in order;
every conventions field from Task 2 is asked for.

- [ ] **Step 6: Commit**

```bash
git add skills/redmine-collab-onboarding/SKILL.md skills/redmine-onboarding/SKILL.md
git commit -m "feat: add redmine-collab-onboarding skill and base pointer"
```

---

### Task 6: Cross-artifact consistency sweep

**Files:**
- Modify: any of the five artifacts, only to fix findings from this sweep.

**Interfaces:**
- Consumes: all previous tasks.
- Produces: the finished, internally consistent artifact set.

- [ ] **Step 1: Citation resolution check**

Every `§N` citation in the three SKILL.md files names a section that
exists in `collab-protocol.md` under that number and actually contains
the cited rule. Check mechanically, then by reading:

```bash
grep -n "§[0-9]" skills/redmine-coordinator/SKILL.md skills/redmine-reviewer/SKILL.md skills/redmine-collab-onboarding/SKILL.md
```

- [ ] **Step 2: Field-name and vocabulary check**

Conventions field names used in any skill match Task 2's list
character-exactly. The disposition vocabulary (accepted /
accepted-with-modification / rejected-with-evidence /
needs-clarification), the attestation markers (posted below / dedup'd
against / concur), and the two verbatim formulas (agreement suffix,
quotability sentence) are identical wherever they appear:

```bash
grep -rn "not authorization to proceed" skills/ | sort
grep -rn "accepted-with-modification" skills/
```

- [ ] **Step 3: No-restatement check**

Skim each SKILL.md for passages that restate protocol rule content at
length (more than one sentence of a §'s substance). Replace with a
citation. This is the spec's own F1 drift lesson — the sweep exists
because it already happened once, in the spec itself.

- [ ] **Step 4: Spec coverage check**

Walk spec §2 (artifact list), §5, §6, §7 bullet by bullet; confirm each
lands in an artifact. Walk protocol §1–§10 against spec §4.1–§4.10 the
same way. Fix gaps inline.

- [ ] **Step 5: Commit (only if fixes were made)**

```bash
git add -A skills/ redmine-conventions.template.md
git commit -m "fix: consistency sweep across collab artifacts"
```

---

## After the Plan (not plan tasks)

- **Sandbox dry-run (spec §9):** human-gated acceptance testing — full
  N=1 cycle, then N=2 with personas, plus the five probes (gate-hold,
  stall, waiver, TOCTOU, resume). Runs with the human playing all gates
  against a sandbox project and throwaway repo. This is a live exercise,
  not an implementable task.
- **README update:** README.md currently says "Two skills ship here" and
  has uncommitted human edits. Deferred — coordinate with the human
  before touching it.
