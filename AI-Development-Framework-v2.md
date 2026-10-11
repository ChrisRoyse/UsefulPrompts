# Issue-Driven, Evidence-Gated Development

*A framework for building software with AI coding agents*

This document describes a complete way of running a software project in which AI coding agents do most of the building. It is written to apply to any project, in any language, on any issue tracker. Wherever a rule depends on your situation, the document says so and names the setting you need to choose.

It is organised as five layers, followed by guidance on scaling to several agents, keeping the system current, and adopting it from scratch. Copy-ready templates are in the appendix.

| Part | Title | What you get from it |
| --- | --- | --- |
| 1 | The framework in brief | The whole system on one page, and the loop every session follows |
| 2 | Principles | The nine ideas every later rule comes from |
| 3 | The repository as operating manual | Which files to create and what each one is for |
| 4 | The issue tracker as state | Labels, pinned context issues, the issue lifecycle, comment formats |
| 5 | Verification | How "done" is proven, and what does not count as proof |
| 6 | The session protocol | How an agent starts, checkpoints, pauses and closes |
| 7 | Guardrails | The mechanical limits that stop bad work |
| 8 | Working with several agents | Claims, fan-out and who owns tracker changes |
| 9 | Keeping the framework alive | Changing rules, changing direction, lessons |
| 10 | Adopting it | A setup sequence and the settings you must choose |
| A | Templates | Entry file, charter, context issues, issue bodies, comments |

## 1. The framework in brief

AI coding agents have two structural weaknesses. They remember nothing between sessions, and they report success more readily than they check it. The framework is built around those two facts.

It gives the agent a durable memory outside its own context (the issue tracker), a rulebook it rereads at the start of every session (doctrine files in the repository), and one definition of "done" that cannot be met by saying so (evidence read back from the real system).

| Layer | What it is | The question it answers |
| --- | --- | --- |
| Doctrine | A short binding charter and an operating manual, both in the repository | What are the rules here? |
| State | The issue tracker: pinned context issues, tasks, epics, decisions | Where are we, and what is next? |
| Verification | Act on the real system, then independently read the stored result | Did it actually work? |
| Session protocol | Fixed routines for starting, checkpointing, pausing and closing | How does a fresh agent carry on? |
| Guardrails | Local hooks, size limits, fail-closed errors, hygiene rules | What stops bad work without relying on judgement? |

### The loop

Every session, whether it lasts ten minutes or ten hours, runs the same loop.

1. **Read state.** Reread the rulebook, the pinned context issues and the live state of the repository.
2. **Pick.** Take the next issue in the declared order. Do not choose freely.
3. **Claim.** Mark the issue in progress and post a short plan.
4. **Work in scope.** Change only what the issue describes. Note anything else you find in a comment on the active issue.
5. **Verify against reality.** Run the real thing and read back what it stored.
6. **Record evidence.** Post the command, the context and the read-back result on the issue.
7. **Close.** Close the issue, commit, push, and confirm the push landed.
8. **Update the snapshot.** Edit the "You are here" issue so the next session starts from the truth.

> **The rule that outranks the others.** The issue tracker is the only record of what is done, in progress and remaining. If it is not on an issue, it did not happen.

## 2. Principles

The rest of the document is detail. These nine ideas are what the detail is for; when a situation is not covered, decide from these.

| Principle | What it means in practice |
| --- | --- |
| One home per kind of information | Design lives in documents. Progress lives in issues. Reality lives in what the system has stored. Never write progress into a design document, and never treat an issue as proof of what the system does. |
| A return value is a claim | Exit codes, API responses, log lines, passing tests and an agent's summary are all claims. The stored result is the verdict. Read the stored result. |
| Write for a successor with no context | Every issue, comment and handoff is written so that an agent starting cold can continue without working anything out again. |
| Fail closed and say why | When something is missing, malformed or unproven, stop with a structured error. No silent fallback, no substitute success value, no mock data in verification. |
| One change at a time, inside scope | An issue has a stated scope. Anything found outside it is noted on the issue and left alone. Several simultaneous changes make cause and effect unreadable. |
| Rules carry an ID, a date and a reason | Numbered rules can be cited in issues and commits. Dated rules can be superseded cleanly. A stated reason lets an agent handle the case the rule did not foresee. |
| Edit to the truth | Current-state documents are corrected in place. A false statement is deleted, never left beside its correction. |
| Missing prerequisites are work | A missing tool, file or service is the next task, not a stopping point, as long as the fix is local and reversible. People direct the work and approve anything destructive or outward-facing. |
| Thresholds before results | Acceptance numbers are written down before the measurement is taken. A failed gate is a result to record, not a number to adjust. |

## 3. The repository as operating manual

The repository holds everything an agent needs to know about how to work, in files it can read at the start of a session. The repository does not hold progress.

### Files to create

| File or folder | Contents | Rule |
| --- | --- | --- |
| Agent entry file at the root | Read order, execution context, the issue workflow, standing invariants, layout, build notes | Short enough to reread every session. This is the first thing an agent opens. |
| Charter (`DOCTRINE.md`) | The cardinal rule, what the project is, numbered invariants, anti-patterns, locked decisions | Overrides every other document. Says so in its first lines. |
| Design documents | What the system is and why: a numbered specification or blueprint | Never records progress. Design corrections are allowed; status prose is banned. |
| Implementation plan | Phases in dependency order, each with deliverables and one exit gate; task cards per phase | The plan says in what order and proven how. Issue state supersedes any checkbox in it. |
| Protocol documents | How to code and debug, how to split large files, how to write documentation | Binding. They put the charter into practice and override nothing in it. |
| Evidence or reports folder | Written gate results and the readback behind them | A gate result is a file plus an issue comment, never a remark in chat. |
| Status snapshot (optional) | A dated summary generated from the tracker | States that it summarises the tracker and does not replace it. |
| Skills folder | One packaged routine per recurring workflow (part 6) | Each skill says when to use it and when not to. |
| Hooks folder | Local pre-commit and pre-push gates (part 7) | Installed once per clone. Never bypassed. |

If you use more than one agent tool, give each its own entry file with the same binding rules, and have each point to the other. An agent should reach the same rulebook whichever tool opened the repository.

### What goes in the entry file

1. **One paragraph on what the project is,** ending with the sentence that names the tracker as the state of record.
2. **The read order:** charter, then the pinned context issues, then the governing design documents.
3. **Execution context:** the one canonical workspace, the one environment where builds and evidence count, and where outputs must stay.
4. **The issue workflow,** as numbered steps (part 4).
5. **Standing invariants:** the handful of rules that apply to every change.
6. **Layout:** what each top-level directory owns, with warnings on areas under restructuring.
7. **Build notes:** the exact commands, including the non-obvious flags.
8. **The rules fresh agents break most often,** listed by rule ID in one line.

### What goes in the charter

- **The cardinal rule,** stated in one or two sentences.
- **What the project is,** and the end state it is heading for.
- **Numbered invariants.** Each has a short ID so that issues, commits and reviews can cite it.
- **How the tracker is used,** in summary, pointing to the full protocol.
- **Execution order:** the dependency spine of phases.
- **Anti-patterns:** a list of things to refuse, each tied to the invariant it breaks.
- **Locked decisions:** the owner's directives, captured so no agent re-argues them.

### Order of authority

When two sources disagree, the higher one wins. State this order in the charter.

1. The charter.
2. The entry file and the protocol documents.
3. Pinned context issues, for current state.
4. Design documents.
5. Task issues and their comments.

Within one level, a newer dated directive supersedes an older one, and says which one it supersedes. An agent told to break the charter refuses and asks the owner.

### How these documents are written

- **Dense.** Cut anything a capable reader would supply unaided: restatement, preamble, obvious examples.
- **Exact where it matters.** Numbers, thresholds, file paths, error codes, rule IDs and commands are kept word for word.
- **Pointers, not copies.** A rule has one home. Everything else links to it, so there is nothing to drift.
- **Structured.** Tables and lists for anything a reader will scan; prose for reasoning.
- **One term per concept.** Pick a name and keep it.

## 4. The issue tracker as state

Agents are stateless across sessions. The tracker is their shared memory, and it is treated with the same care as code.

| Tracker element | Role |
| --- | --- |
| Open issues | Active state: what exists to be done right now |
| Comments | The journal: what was tried, found and proven, in order |
| Closed issues | Institutional knowledge: searchable decisions and evidence |
| Labels and milestones | Organisation: type, status, priority, area, phase |

> **An issue is still a claim.** The tracker records what you are doing and what you have learned. It never replaces reading the real system. The two stay consistent because every close carries evidence read from the system.

### 4.1 Labels

Create the label set once, before the first task. Keep the families separate so that a query on one family is meaningful.

| Family | Typical values | Rule |
| --- | --- | --- |
| Type | context, epic, task, bug, decision, pattern, blocker | Exactly one per issue |
| Status | ready, in-progress, blocked, needs-spec | At most one. Open with no status means unclaimed. |
| Priority | p0, p1, p2, p3 | One per issue. p2 is the default. |
| Severity | critical, high, medium, low | Defects only |
| Area | One per subsystem | One or two per issue |
| Phase | A milestone or a phase label per stage of the plan | One per task |
| Source (optional) | agent, human; or which agent | Useful when several agents or people file issues |

Priority is a statement about severity and integrity. It is a poor work queue once many issues share the top level. When that happens, add an explicit ordering (section 4.4) instead of inventing a higher priority.

### 4.2 Pinned context issues

A small set of pinned issues holds what every agent must know before acting. They are read at the start of every session and kept in context throughout.

| Context issue | Holds, as current truth only |
| --- | --- |
| Mission and invariants | What the project is for, a pointer to the charter, and the current scope |
| You are here | Current objective, what is verified, what is in flight, what comes next, and the order to finish in |
| Environment and operations | Where work runs, how to reach it, paths, how to build and run, where secrets come from |
| Landmines | Numbered traps that catch every newcomer, each with how it was found and how to avoid it |
| Data and assets (if relevant) | Which datasets, models or fixtures are acquired and verified, and what is still needed |
| End state | The complete definition of finished (section 4.3) |

Rules for context issues:

- **Snapshot, not journal.** They state what is true now. History belongs in closed decision issues and in issue comments.
- **Pointers, not copies.** Link to the charter section; state only the current status beside the link.
- **Short.** If one grows long it is collecting history. Prune it.
- **Stamped.** Each carries the date it was last checked against the real system.
- **Edited to the truth.** When a fact changes, rewrite the line. Never append a correction beneath a stale statement.
- **Pruned at every phase boundary.** Reread each line, check it against reality, delete what is stale, and record the pass in a comment.

A stale context issue misleads every agent that reads it afterwards. Keeping these current is a correctness task, not housekeeping.

### 4.3 The end-state issue

Write down what finished means before building much. One pinned issue lists every artifact that must exist at the end, and for each one:

- **What** it is.
- **Where** it lives.
- **How to verify it:** the check an auditing agent runs, with the expected result.

It also lists what is out of scope and ends with a final audit: the sequence of checks that, when all pass, means work stops. Every task should trace to making one statement in this issue true. A task that does not is either covered elsewhere or out of scope.

### 4.4 Epics, phases and order

- **Phases are milestones.** State the dependency spine explicitly, including which phases may run in parallel. Work the lowest open phase first.
- **An epic groups tasks** with a checklist of linked issues and one completion criterion per phase. Tick a box only when its issue is closed with evidence.
- **An ordering issue says what comes next.** When there is a backlog, one pinned issue lists the queue in order. Agents take the next item; they never pick by issue number or by preference.
- **Order by blast radius.** Fix first whatever makes evidence unreliable, because every later check depends on it. Then data loss and durability, then bounded correctness, then performance and tidiness.

### 4.5 What a task issue contains

| Section | Contents |
| --- | --- |
| Problem | Observed behaviour, with the exact command, output and execution context |
| Evidence | How it was observed: readback output, file and line, commit |
| Root cause | The structural cause if known, and whether it predates the current work |
| Scope | The bounded change this issue authorises |
| Links | `Blocked by:` and `Blocks:` lines |
| Definition of done | A checklist. Each box names the verification that proves it. |

The definition of done is what makes an issue closable by evidence. A box reads as a named check with an expected stored result, not as "works correctly".

### 4.6 The issue lifecycle

1. **Pick.** Choose an issue marked ready in the lowest open phase, or the next item in the ordering issue. Never start one marked blocked or needs-spec.
2. **Claim.** Post what you are about to do and a plan of five bullets or fewer. Change the status to in-progress in the same action.
3. **Check what is already done.** Read the issue body and every comment. Earlier sessions record completed work there. Do not redo a ticked item; rerun its named check and note the result.
4. **Work in scope.** Implement only the Scope section. Record anything adjacent in a comment on the active issue. Do not widen the scope to absorb it.
5. **Reference the issue in every commit.** Use a "refs" keyword until the last commit, and the closing keyword only once evidence exists.
6. **Prove, then tick.** A box may be ticked only in the session that ran its check, with the command and the read-back result pasted in a comment.
7. **Close.** When every box is ticked with evidence: post the closing comment, close the issue, tick it in its epic, remove the status label, commit and push.

If you must stop early, say exactly what is done, what is not, the next concrete step, and what is uncommitted. Then set the status to ready if someone else can continue, or blocked with the blocker named. Never leave an issue silently in progress.

### 4.7 Comment types

Use a fixed set of comment types with fixed headings. A successor can then find the resume point by scanning for the last one.

| Comment | When | Must contain |
| --- | --- | --- |
| CLAIM | Before any work | Plan, files to be touched, the source of truth that will be checked |
| Milestone | Reproduction, change of direction, new finding | What was learned and the evidence for it |
| Heartbeat | Long tasks, at a regular interval | Done so far and next step |
| Evidence | Each definition-of-done box | The box text, the exact command, the result, the readback |
| PAUSE | Stopping with work unfinished | Done, tried and failed, learned, where to resume, what to check on resume, uncommitted state |
| BLOCKED | A real wall after local options are exhausted | The blocker, the condition that unblocks, a link both ways |
| RESOLVED | Closing | Root cause and fix in two sentences, the verification, side effects, anything noted outside scope |

"Tried and failed" is the most valuable line in a pause comment. It saves the next agent the dead end.

Optionally, end each state-changing comment with a one-line machine-readable block (for example a small JSON object with the comment kind, timestamp, session and issue number). This lets a script or a later agent reconstruct a session history without parsing prose.

### 4.8 Knowledge issues

Decisions are recorded as issues, then closed. The closed issue is the permanent, searchable record. Lessons and traps go straight into the Landmines context issue (section 4.2).

| Type | Body sections | Lifecycle |
| --- | --- | --- |
| Decision | Context, decision, rationale, alternatives considered, consequences, supersedes, references | Closed once recorded. Reopened only to supersede, with links both ways. |
| Pattern | Signature, use when, example, anti-pattern to avoid | Closed once recorded. If universal, add one line to the entry file pointing to it. |
| Direction change | What changed, the measured reason, what it supersedes and what it leaves untouched, the new work order | Stays open and pinned while it governs. |

### 4.9 Tracker hygiene

- **Close done work immediately.** An open issue means active. A finished issue left open is a false statement.
- **Release stale claims.** Set a threshold after which an in-progress issue with no comment or commit returns to the queue.
- **Close duplicates with a link** to the issue being kept.
- **Supersede, do not contradict.** Two open decisions that disagree is a defect in the tracker.
- **Comment on every push** that touches an issue's files, with the commit and a one-line summary.

## 5. Verification

This layer is what makes agent-built software trustworthy. It has one name, Full State Verification, and one rule: after acting, perform a separate read of the place where the result physically lives, and compare it to what you expected.

### 5.1 The protocol

1. **Name the source of truth.** Where does the final result live? A database row, a file on disk, a queue, a ledger, a process state, a rendered page. Name it before running anything.
2. **Predict the outcome.** With an input you constructed, state the expected result and where it will appear, before executing.
3. **Read the state before.**
4. **Execute the real path.** The real artifact, the real store, real or known-answer data.
5. **Read the state after, independently.** Use a separate read operation. Do not use the return value of the action.
6. **Inspect the difference** between before and after against the prediction.
7. **Run at least three edge cases:** empty input, limit or maximum input, invalid format. Show state before and after each. Invalid input must be refused, and the refusal must leave stored state unchanged.
8. **Record the evidence** on the issue: commands, context, readback output.

If a process has an observable trigger and a stored outcome that can be checked, checking it is mandatory.

### 5.2 What counts as evidence

| Counts | Does not count |
| --- | --- |
| A separate query against the stored data after the action | The action's own return value or status code |
| A file reopened and compared by content or hash | The write call reporting success |
| The same real run repeated and compared byte for byte | A claim that the process is deterministic |
| The tracker or remote state read back after changing it | The change command exiting cleanly |
| A measurement taken against the running system, with its input size and date | An estimate, or a complexity claim in a comment |
| A refused operation plus proof that nothing was stored | An error message alone |

"Success response, row unchanged" is a failed check. No error is needed for it to fail.

### 5.3 Rules that keep verification honest

- **Smallest sufficient data.** Before using the large dataset, ask what the smallest input is that fully proves the behaviour. Scale up only when scale is the claim.
- **Real data for behaviour, constructed data for mechanics.** Known-input, known-output cases prove mechanics. Claims about quality on real inputs need real inputs.
- **No mocks in verification.** A check that passes against a stand-in proves the stand-in.
- **Separate your failures from inherited ones.** Before blaming your change, check the history. Record inherited failures on the active issue; do not absorb them.
- **Do not mask exit codes.** Piping output through a filter can hide a failure. Capture the real status.
- **Deferred or skipped checks are not passes.** Label them, count them, track them on an issue, and never present them as passing.
- **When a check fails, stop.** Find the root cause before changing anything else.

### 5.4 Binding evidence to what was built

- **Record provenance.** Evidence names the commit of the tree that was built and a hash of the artifact that was exercised.
- **Freeze the tree during an evidence build.** From the start of the build until the readback is captured, do not edit that checkout. An edit inside that window voids the evidence.
- **Treat build output as disposable.** Clean it after every build or verification batch, pass or fail, and confirm it is gone. It is scratch space, not project state.
- **Keep outputs inside the workspace** and out of version control.
- **Read back the push.** A push is complete when the remote head equals the local head and the working tree is clean.

### 5.5 Gates

- **Every phase has one exit gate,** written as exact numbers before the work begins.
- **A gate result is a written artifact** in the evidence folder, plus an issue comment containing the readback.
- **A failed gate is a result.** Record it and adjust the plan.
- **Thresholds move only by decision issue,** never after seeing the number.
- **Statistical claims need controls.** Where a result could arise by chance, a random or negative control is part of the gate, and the verdict against it is stated either way.

### 5.6 Where automated tests and CI fit

This is a setting, and the framework works at either end of it.

| Setting | How it works | Suits |
| --- | --- | --- |
| Tests as the fast claim | Unit, property, fuzz and mutation tests run locally as the inner loop. Each test must fail when the code is wrong and pass when it is right. Manual readback remains the gate. | Libraries, parsers, storage formats, anything with many small invariants |
| Manual verification only | No test suite. Each change is verified by exercising the real artifact and reading back stored state. A successful build is the only automated evidence. | Systems whose behaviour is mostly end-to-end, where a suite costs more wall-clock than it returns |

In both settings a passing test never closes an issue alone. Likewise for hosted CI: the practice described here runs all gates locally, invoked by the agent, with a pre-push hook as the backstop, to keep the loop fast and free of cost. A team that keeps hosted CI should give it the same standing as a test suite: useful, and still a claim.

### 5.7 Batch the verification

- One build serves every pending close in a session.
- Readback probes for several issues run as one combined script in one pass.
- Tracker changes are batched per round: all claims together, all evidence comments and closes together.
- Keep built artifacts, fixtures and driver scripts in session scratch space so that re-verification reruns artifacts instead of rebuilding them.
- Reuse a proven verification driver before writing a new one.

### 5.8 Cost is part of correctness

A data-processing path that is correct but cannot finish at production scale is a defect. A small fixture that passes has measured the cost of nothing. Before writing or extending any such path, answer these in the issue:

1. What is N? Read it from the real artifact. "Unknown" is an honest answer and a tracked gap. A guess is neither.
2. Per what does this run: per process, per pass, per item, per byte?
3. What stays constant across the loop it sits in? Compute that once, outside the loop.
4. What does it recompute, reopen, reparse or rehash that already exists?
5. Is there already a batched or incremental version of this function nearby?
6. What does it append to, and who retires those bytes?
7. What lock does it hold, for how long, and who else needs it?
8. Is its output order deterministic? If not, fix that before any parallel work.

Measure before choosing a remedy, and name the dominant term and its share. Never fix a cost by raising a timeout or budget, by adding concurrency to something that is waiting rather than computing, or by deleting a check that fails closed.

Keep a numbered catalogue of the cost defects the project has paid for, each with its mechanism, how to spot it in review, and the question that prevents it. Cite the catalogue number in issues and commits.

## 6. The session protocol

### 6.1 Start of session

Run this on a fresh start, on resume, and after any context compaction. The on-disk version of each file is authoritative over anything remembered.

1. Reread the entry file and the charter.
2. Read every pinned context issue, the ordering issue first.
3. Run the read-state queries: what is in progress, what is blocked, the unclaimed queue, the decisions that bind this area.
4. Fetch, and reconcile the working tree against the last handoff's statement of uncommitted work.
5. Read the active issue body and every comment.
6. Claim, or continue the existing claim.

### 6.2 During the session

- Comment at milestones, not at every step.
- Post a heartbeat on long tasks.
- Commit verified slices, push, and read back the push before starting the next slice.

### 6.3 End of session

1. **Hygiene first.** Build output gone, no stray branches, worktrees, locks or temporary files from this session, every uncommitted path accounted for.
2. **Commit and push what is verified.** Unverified code is named in the handoff, or committed to a clearly named side branch, never committed as if done.
3. **Post the closing or pause comment** on the active issue.
4. **Set the status label** to match reality.
5. **Update "You are here"** if the project state changed.

The handoff lives on the issue. Do not write handoff or memory files to disk; a successor will not know to look for them, and they drift from the tracker.

### 6.4 Package the routines as skills

Each recurring routine becomes a named skill or command that the agent can invoke. This turns the protocol from something the agent must remember into something it runs.

| Skill | Does | Use when |
| --- | --- | --- |
| Issue driver | Runs one issue through pick, claim, check prior work, scoped work, proof, close | Starting, resuming or closing an issue |
| Verify | The verification protocol and the evidence format | Deciding whether something worked |
| Resume | Injects live repository, lock and tracker state, then walks the mandatory rereads | Fresh session on existing work, or after compaction |
| Handoff | Hygiene check, done and not-done record, next step, label swap | Any pause or stop with work unfinished |
| Situation report | Handoff plus a candid read: what is solid, what is at risk, on track or not | On request, or before a risky transition |
| Commit and push | Review the diff, hygiene sweep, issue-referenced message, push, read back | Publishing verified work |
| Fan-out | The parallel-work playbook (part 8) | Several ready issues that do not overlap |
| Skill lint | Checks that the skills themselves are well formed and current | After editing any skill |

Each skill description states what it is for and what it is not for, and names the skill to use in the other case. Restrict each skill to the tools it needs.

### 6.5 Unattended loops

For recurring runs, keep a short per-tick prompt in the repository: read the context issues, pick the highest-priority unclaimed issue, claim it, audit the task against the live tree (assume the description may be wrong and read the code first), ship one unit end to end with evidence, and close with a resolved, pause or blocked comment. One unit per tick.

## 7. Guardrails

These limits work without relying on the agent's judgement in the moment.

### 7.1 Local gates as hooks

- **Pre-push:** formatting and lint at deny level, and a scan of the pushed commits for credentials. Skip the compile gate when a push contains no source changes.
- **Pre-commit:** repository boundary checks, such as forbidden dependencies or references to places this repository must not touch.
- **Fail loudly.** If a required tool is missing, reject the operation with a clear message. Do not skip the check.
- **Never bypass.** A hook failure is information. Fix the cause.
- **Comment each hook with why it exists:** the incident that motivated it.

### 7.2 A file-size ceiling

Set a hard maximum length for code files; documentation is exempt. Smaller files fit in an agent's context, produce smaller diffs, and let parallel agents work without colliding. When a file would exceed the limit:

1. Split it before the change lands. An over-limit file is never left in place silently.
2. Split by single responsibility into a module directory.
3. Leave the original path as a thin facade that re-exports the same public names.
4. Keep the public interface identical and add no circular dependencies.
5. Rebuild, re-verify, and rerun the line-count check.

### 7.3 Error discipline

- **Every failure returns a code, a message and a remediation.** The next agent should know what failed and what to do about it.
- **No silent fallback.** Every degradation is labelled and every skip is counted.
- **No stubs on shipped paths.**
- **No constant that could be a measurement.** New thresholds are declared, named settings or measured values.
- **Root cause before fix.** Keep asking why until you reach a structural property of the system. "Someone forgot" is not a root cause.

### 7.4 One place where work counts

- **One canonical workspace.** No copied checkouts or shadow trees.
- **One execution environment for evidence.** If the product ships for one target, evidence comes from that target. Other machines are editing clients.
- **Clean up only what you started.** Never stop or remove processes, services or files whose ownership you cannot prove. A crash or reconnect does not authorise a general cleanup.
- **Secrets by name only.** Values come from a secret manager at run time and never appear in the repository, an issue, a commit or a chat.

### 7.5 Declared non-goals

State what the project deliberately does not do, with a date and a reason: platforms not yet supported, concerns that do not apply to its deployment, features parked. This stops agents from doing well-meant work nobody asked for. A non-goal is disclosed, labelled and tracked; it is not a silent omission, and deferring something is different from abandoning it.

### 7.6 The autonomy boundary

| The agent decides and does | A person directs or approves |
| --- | --- |
| Technical, toolchain and implementation choices | Project direction and priorities |
| Installing, configuring or generating a missing local prerequisite | Destructive actions |
| Choosing the more durable option over the locally easier one | Anything outward-facing: publishing, sending, deploying to others |
| Labelling and ordering issues | Changes to the charter or to a gate threshold |

The agent does not pause for routine approval or leave a known next step undecided. "Blocked" is reserved for the specific action only a person can take, after local options are exhausted.

### 7.7 Honesty about "done"

- Never say done without evidence. Open the diff, rerun the check, read the stored state.
- Report partial or blocked rather than a false pass.
- Record failure as carefully as success. It is the next agent's lesson.
- For investigations, assume the description may be wrong, look for contradictions between claims and stored state, and argue the opposite case before reaching a verdict.

## 8. Working with several agents

### 8.1 Claims

- **Claim before work,** with the status change and the comment in one action.
- **Earliest claim holds,** unless it has gone silent past the stale threshold. The later agent yields in a comment and picks another issue.
- **Check before starting.** Another agent may have landed a fix. Fetch, read the remote history and the issue state, and prefer verifying a shipped fix to rebuilding it.
- **On a single-agent project, say so.** State in the entry file that an old in-progress label never prevents continuation, so the agent does not stall on its own earlier claim.

### 8.2 Fan-out

Parallel work is safe when the pieces touch disjoint files and do not need the same exclusive resource.

| Runs in parallel | Stays sequential |
| --- | --- |
| Single-module work in modules with no shared files | Anything touching a shared hub file or a module under restructuring |
| Read-only audits and analysis | Any two efforts that need the one build toolchain or the one exclusive device |
| Issues whose scope is small and exact | Issues that need repository-wide context |

**The orchestrator owns every tracker change.** Workers never push, label, comment or close. Each worker gets an isolated worktree and branch and returns a structured result:

```
{ issue, status: done | partial | blocked | already-fixed,
  evidence[], commits[], pending_integration, notes[] }
```

The orchestrator then merges the disjoint branches, runs one consolidated build and one combined verification pass, closes each issue with evidence, posts a summary, and removes the worktrees and branches.

A worker that needs wide context is a sign the issue belongs in the sequential lane.

### 8.3 Separate author from reviewer

Where a result decides something important, have it checked by a session that did not produce it, starting from the stored evidence and not from the author's summary.

## 9. Keeping the framework alive

### 9.1 Changing a rule

Rules change as the project learns. Change them in the open:

- Add the new rule with its date and source, and state which earlier rule it supersedes.
- Rewrite or remove the old text where it lives. If history must be kept, mark it clearly as retired and non-authorising.
- Update every file that repeats the rule, or better, replace the repeats with pointers.
- Reconcile any remaining documents that still use the retired language in the same change.

### 9.2 Changing direction

A change of objective is filed as a pinned direction-change issue. It states the measured reason, exactly what it supersedes, what it leaves untouched, and the new order of work. Older issues keep their evidence; only their priority changes. Put a one-paragraph note of the current objective at the top of the entry file, with the date.

### 9.3 Turn incidents into rules

When something costs real time, distil it: one numbered line in Landmines, or a new invariant in the charter, or a hook. Each rule in a mature project should trace back to the incident that paid for it. When a landmine is fixed, delete the line.

### 9.4 Share lessons across projects

If two projects on the same foundations hit the same class of defect, write one catalogue and reference it from both. Agents starting a new project then inherit the lessons instead of rediscovering them at full cost.

## 10. Adopting it

### 10.1 Setup sequence

1. **Write the charter.** One page to begin: the cardinal rule, what the project is, five to ten numbered invariants, a short anti-pattern list.
2. **Write the entry file.** Read order, execution context, the issue workflow, build commands.
3. **Create the labels** (section 4.1) and one milestone per phase.
4. **Open and pin the context issues:** Mission, You are here, Environment and operations, Landmines.
5. **Write the end-state issue.** List the artifacts and how each is verified.
6. **Lay out the phases** in dependency order, each with one numeric exit gate.
7. **Create the first tasks,** each with a scope and a definition of done.
8. **Install the hooks** and set the file-size limit.
9. **Package the first skills:** issue driver, verify, handoff, resume.
10. **Run one issue end to end** and correct the documents wherever the agent hesitated or guessed.

### 10.2 Settings to choose

The framework is the same everywhere; these settings are not. Decide each one, write the decision in the charter with its reason, and change it by decision issue.

| Setting | Options | Guidance |
| --- | --- | --- |
| Automated tests | Fast inner-loop claim, or none | Either way, stored-state readback is the gate |
| Hosted CI | None with local hooks, or kept as a claim | Do not let a pipeline become the definition of done |
| Agents | One, or several | With one, declare that stale claims never block |
| Stale-claim threshold | Hours to days | Shorter with several active agents |
| Branching | Direct to main with issue-referenced commits, or one branch and one review per issue | Use branches when more than one agent writes |
| Execution environment | Local, or a dedicated host | Exactly one counts for evidence |
| File-size ceiling | A fixed line count | Pick one number and enforce it by script |
| Context issue set | The four core issues plus any the project needs | Add one only if every agent needs it every session |
| Non-goals | Project-specific | Dated, with a reason, and tracked |

### 10.3 Signs it is working

- A new session reaches productive work without asking what the state is.
- Every closed issue contains a command and a read-back result.
- The "You are here" issue matches reality when checked.
- Diffs stay inside the scope of the issue they reference.
- A rule in the charter can be traced to the reason it was written.

### 10.4 Common ways it degrades

- Context issues that grow into logs.
- Finished issues left open, or issues closed on a summary without evidence.
- Progress notes creeping into design documents.
- A verification script that prints a green tick and stands in for reading the state.
- A threshold or timeout quietly raised to make something pass.
- The same rule copied into three files and updated in one.

## Appendix A. Templates

Replace anything in angle brackets. Commands use a generic issue-tracker command-line tool; substitute your own.

### A.1 Agent entry file

```
# <Project> - agent entry point

<One paragraph: what the project is.>
State of record: the issue tracker. Nothing else.

## Read in this order, every session
1. DOCTRINE.md - binding charter, overrides everything.
2. Pinned context issues - Mission, You are here, Environment, Landmines.
3. <governing design documents>

## Execution context
- Canonical workspace: <path>. No copies or shadow trees.
- Evidence environment: <target>. Builds elsewhere do not count.
- Outputs stay under <dir> and are never committed.
- On resume or compaction: reread this file, the charter, the active
  issue and all its comments, and the live repository state.

## Issue workflow
Pick -> Claim -> Check prior work -> Work in scope -> Prove, then tick
-> Close, commit, push. Details: <link to protocol>.

## Standing invariants
1. <rule>   2. <rule>   3. <rule>

## Rules most often broken
<ID> <rule> - <ID> <rule>.

## Layout
<dir> - <what it owns>

## Build
<exact commands>
```

### A.2 Charter

```
# <Project> Doctrine - the binding charter

> This document overrides every other document in this repository.
> Read it first, every session.

## 0. The cardinal rule
A return value is a claim. The source of truth is the stored state.
Read the stored state.

## 1. What this project is
<Purpose and target end state.>

## 2. Invariants (binding)
1. <Name> (R1). <Rule. Reason.>
2. <Name> (R2). <Rule. Reason.>

## 3. The tracker is the coordination and memory surface
<Summary and pointer to the full protocol.>

## 4. Execution order
<Phase 0> -> <Phase 1> -> (<Phase 2> || <Phase 3>) -> <Phase 4>

## 5. Anti-patterns (refuse these)
- <Thing to refuse> (breaks R<n>).

## 6. Locked decisions
1. <Directive, date, what it supersedes.>
```

### A.3 Context issue: You are here

```
[CONTEXT] You are here

> LAST VERIFIED <date> - replaces the <earlier date> snapshot.

## Current objective
<One paragraph.>

## Verified
- <Fact, with where it was read from.>

## In flight
1. #<n>: <state, and what remains before close.>

## Binding context for this work
- <Constraint every agent needs right now.>

## Completion order
<step> -> <step> -> <step>
```

### A.4 Context issue: Landmines

```
[CONTEXT] Landmines

Add one line here when a trap is found.
Remove a line when the trap is fixed, with evidence.

1. <Trap> (found <date>, during #<n>): <what happens>. <How to avoid it.>
2. <Trap> ...
```

### A.5 Task issue body

```
## Problem
<Observed behaviour; exact command, output, execution context.>

## Evidence
<Readback output, file:line, commit.>

## Root cause (if known)
<Structural cause; whether it predates current work.>

## Scope
<The bounded change.>

Blocked by: #<n>
Blocks: #<m>

## Definition of done
- [ ] <Change implemented; errors return code, message, remediation.>
- [ ] <Named check: real command + independent readback of stored state.>
- [ ] <Three edge cases, with state before and after each.>
- [ ] Evidence recorded here; build output cleaned and confirmed gone.
```

### A.6 Comment formats

```
CLAIM - agent:<name> session:<id> commit:<sha>
Plan: <2-5 bullets>
Files I will touch: <list>
Source of truth for verification: <table / file / queue / system>

PAUSE - agent:<name> session:<id> commit:<sha>
Done: <verified items only>
Tried and failed: <dead ends>
Learned: <gotchas; add to Landmines if every agent needs it>
Resume at: <file:line> with <next command>
Check on resume: <where to read state>
Uncommitted: <status output, or "clean">

BLOCKED by <#n | owner decision | owner-only action>.
Cannot proceed until <condition>.

RESOLVED - agent:<name> commit:<sha>
Fix: <root cause and structural fix, two sentences>
Verification:
  - Build / lint: <status>
  - Evidence: <state before -> action -> state after, with values>
  - Edge cases: <three, each with outcome>
Side effects: <or "none">
Noted outside scope: <or "none">
```

### A.7 Decision body

```
[DECISION] <title>
## Context          ## Decision         ## Rationale
## Alternatives considered             ## Consequences
## Supersedes       ## References
```

### A.8 Read-state queries

```
# 1. Pinned context - read all
issues list --state open --label type:context
# 2. Claimed work - do not collide
issues list --state open --label status:in-progress
# 3. Blocked - has the blocker cleared?
issues list --state open --label status:blocked
# 4. Unclaimed queue, oldest first
issues list --state open --label type:task --search "no:assignee"
# 5. Decisions binding this area
issues list --state closed --label type:decision --search "<topic>"
```

### A.9 Turn-end checklist

- Every ticked box has a command and a readback in a comment.
- Verified work is committed, pushed, and the push read back.
- Build output and session scratch are gone; no stray branches, worktrees or locks.
- The active issue carries a resolved, pause or blocked comment.
- The status label matches reality.
- "You are here" reflects the new state.
