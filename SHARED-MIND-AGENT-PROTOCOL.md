# Shared Mind Agent Protocol (SMAP)

> A protocol that turns sequential AI agents into a connected team by giving them a shared, file-based memory. Every agent reads the `./memory/` directory before working and leaves `.md` files behind so the next agent — or any future agent — can pick up exactly where they left off. Project-agnostic.

---

## 1. Problem

When AI agents run sequentially, each starts with a blank slate. Critical context is lost between handoffs:

- Agent A discovers a constraint, Agent B re-discovers it the hard way.
- Agent C contradicts a decision Agent A made because it never knew about it.
- Agent D duplicates work Agent B already completed.
- No agent can see the full picture of what the team has collectively learned.
- Successes and failures vanish — no audit trail, no accountability.

**SMAP fixes this with one rule: every agent reads `./memory/` first and writes `.md` files to `./memory/` before finishing. The directory IS the shared mind.**

---

## 2. The Memory Directory

All shared knowledge lives in plain Markdown files in a `./memory/` directory at the project root. No databases, no servers, no special tools — just files an agent can read and write with `cat`, `Read`, and `Write`.

```
./memory/
├── context/        # Mission, goals, scope, constraints
├── decisions/      # Locked architectural choices that must not be contradicted
├── discoveries/    # Constraints, gotchas, edge cases learned during work
├── handoff/        # Concrete deliverables passed agent-to-agent
├── journal/        # Append-only chronological execution log
├── blockers/       # Unresolved issues that need attention
├── patterns/       # Reusable conventions and standards established
└── reports/        # Each agent's final report
```

### Namespaces

| Namespace | Purpose | Written By |
|-----------|---------|------------|
| `context` | Mission brief, current phase, active goals | Coordinator + scope-changing agents |
| `decisions` | Locked choices later agents MUST NOT contradict | Decision-making agents |
| `discoveries` | Constraints, edge cases, gotchas found during work | Any agent that learns something |
| `handoff` | Concrete deliverables passed to the next agent | Completing agent |
| `journal` | Append-only execution log (per agent, per session) | Every agent |
| `blockers` | Unresolved issues that need attention | Any agent that hits a wall |
| `patterns` | Reusable conventions agents should follow | Any agent that establishes one |
| `reports` | Each agent's full final report | Every agent |

### File Naming

```
{namespace}/{agent-id}--{descriptor}.md
```

Examples:
- `decisions/agent-02-architect--database-schema.md`
- `discoveries/agent-04-coder--api-pagination-limit.md`
- `handoff/agent-03-to-04--implementation-spec.md`
- `journal/agent-01-researcher--2026-04-14.md`
- `blockers/agent-05-tester--flaky-integration-test.md`
- `reports/agent-03-coder--final-report.md`

### Frontmatter

Every `.md` file written to `./memory/` MUST begin with frontmatter so future agents can scan and filter quickly:

```markdown
---
namespace: {context|decisions|discoveries|handoff|journal|blockers|patterns|reports}
agent_id: agent-{N}-{role}
timestamp: {ISO 8601, e.g. 2026-04-14T08:30:00Z}
status: {active|resolved|superseded}
severity: {high|medium|low}     # required for blockers, optional elsewhere
tags: {comma-separated keywords for grep-based discovery}
---

# {Title}

{Content — be thorough; future agents only have what you wrote down.}

## What Happened
{Detailed description}

## Why It Matters
{Impact on other agents and on the project}

## Related
- See also: ./memory/{namespace}/{filename}
- Tags: {keywords for future grep searches}
```

---

## 3. Agent Lifecycle: READ → ORIENT → WORK → WRITE → REPORT

Every agent, regardless of role or project, follows this lifecycle.

### Phase 1: READ

The first thing every agent does is read `./memory/`:

```
1. List the directory:
   ls ./memory/

2. If ./memory/ does NOT exist, create it with all subdirectories:
   mkdir -p ./memory/{context,decisions,discoveries,handoff,journal,blockers,patterns,reports}

3. Read in this order (newest first within each namespace):
   - ./memory/context/     — what is this project? what's the mission?
   - ./memory/decisions/   — what choices am I bound by?
   - ./memory/patterns/    — what conventions must I follow?
   - ./memory/discoveries/ — what constraints/gotchas have been found?
   - ./memory/blockers/    — what's unresolved?
   - ./memory/handoff/     — what did the previous agent leave for me?
   - ./memory/reports/     — what did each previous agent actually do?
   - ./memory/journal/     — recent activity, in order

4. For large directories, scan filenames first, then read in full
   only the files relevant to your task.
```

Do not begin work until READ is complete. If the directory is empty, you are the first agent — note this and proceed.

### Phase 2: ORIENT

After reading, answer internally:

- What is this project? What are we building?
- What is my specific task?
- What did previous agents produce? What will the next agent need from me?
- What decisions am I bound by?
- What discoveries affect my work?
- What blockers must I address or work around?
- What patterns must I follow?
- What has failed before that I should not repeat?

### Phase 3: WORK

Do the assigned task. While working, write `.md` files to `./memory/` IMMEDIATELY whenever any of the following happens:

| Trigger | Where to write |
|---------|----------------|
| You DISCOVER something unexpected (constraint, edge case, gotcha) | `./memory/discoveries/agent-{N}-{role}--{descriptor}.md` |
| You make a DECISION future agents must respect | `./memory/decisions/agent-{N}-{role}--{descriptor}.md` |
| You hit a BLOCKER you cannot resolve | `./memory/blockers/agent-{N}-{role}--{descriptor}.md` |
| You establish a PATTERN others should follow | `./memory/patterns/agent-{N}-{role}--{descriptor}.md` |
| Something FAILS (build error, test failure, wrong approach) | `./memory/journal/agent-{N}-{role}--failure-{descriptor}.md` |
| Something noteworthy SUCCEEDS | `./memory/journal/agent-{N}-{role}--success-{descriptor}.md` |

**Write early, write often. Never hide failures — every failure is a lesson for the next agent.**

If you resolve a blocker someone else logged, edit that blocker file and set `status: resolved` in the frontmatter (don't delete it — the history matters).

### Phase 4: WRITE (Handoff)

Before finishing, write what the next agent needs:

```
./memory/handoff/agent-{N}-{role}--{descriptor}.md
```

Include: artifact content or file references in the repo, a short summary of what they need to know, and any open questions.

### Phase 5: REPORT (Mandatory final action)

Your last action MUST be writing a complete report:

```
./memory/reports/agent-{N}-{role}--final-report.md
```

Template:

```markdown
---
namespace: reports
agent_id: agent-{N}-{role}
timestamp: {ISO 8601}
status: active
tags: report, final, agent-{N}
---

# Final Report: agent-{N}-{role}

## Task Completed: {yes | no | partial}

## Summary
{2-3 paragraphs covering what you did and what happened}

## Files Created or Modified in the Repo
- `path/to/file` — {one-line description}

## Files Written to ./memory/
- `{namespace}/{filename}` — {one-line description}

## Decisions Made
- {decision} — Rationale: {why}

## Discoveries
- {finding} — Impact: {who/what it affects}

## Failures
- {what failed} — Error: {message or description} — Lesson: {what to do differently}

## Successes
- {what worked} — Details: {how}

## Blockers (if any)
- {issue} — Severity: {high|medium|low} — File: ./memory/blockers/{filename}

## Tests / Build
- Tests added: {list or none}
- Tests passing: {yes | no | partial}
- Build status: {passing | failing | not-run}

## For the Next Agent
- Start by reading: `./memory/{namespace}/{filename}` for {what they'll find}
- Watch out for: {gotchas}
- Priority: {what matters most}

## Confidence: {high | medium | low}
{Brief justification}
```

---

## 4. Universal Agent Prompt

Copy this prompt into every agent. Fill in the `{VARIABLE}` sections.

```
You are Agent {AGENT_NUMBER} of {TOTAL_AGENTS} in a sequential pipeline.
Your role: {ROLE}
Your agent ID: agent-{AGENT_NUMBER}-{ROLE}

===============================================
SHARED MIND AGENT PROTOCOL (SMAP)
===============================================

You are part of a team of agents working on the same project.
Other agents have worked before you and others will work after you.
You all share one place: the ./memory/ directory at the project root.
It is full of .md files written by past agents — read them.
You will leave .md files behind for future agents — write them.

DOCUMENT EVERYTHING. Successes AND failures. No silent errors.

===============================================
PHASE 1: READ — DO THIS FIRST
===============================================

1. If ./memory/ does not exist, create it:
   mkdir -p ./memory/{context,decisions,discoveries,handoff,journal,blockers,patterns,reports}

2. Read .md files from ./memory/ in this order:
   - ./memory/context/     — project mission, goals, scope
   - ./memory/decisions/   — locked choices you cannot contradict
   - ./memory/patterns/    — conventions you must follow
   - ./memory/discoveries/ — constraints, gotchas, edge cases
   - ./memory/blockers/    — unresolved issues
   - ./memory/handoff/     — what the previous agent left for you
   - ./memory/reports/     — final reports from prior agents
   - ./memory/journal/     — recent activity

   For large dirs: scan filenames first, then read only the relevant files
   in full. Pay special attention to anything from agent-{AGENT_NUMBER - 1}.

3. If ./memory/ is empty, you are the first agent — note this and continue.

After reading, ORIENT internally:
- What is this project?
- What decisions bind me?
- What did the previous agent produce?
- What will the next agent need from me?
- What has failed before that I must not repeat?

===============================================
PHASE 2: YOUR TASK
===============================================

{TASK_DESCRIPTION}

Inputs from previous agents:
{INPUT_DESCRIPTION}

Expected outputs:
{OUTPUT_DESCRIPTION}

===============================================
PHASE 3: WORK — WRITE .md FILES AS YOU GO
===============================================

While working, write .md files to ./memory/ IMMEDIATELY when:

A) You DISCOVER something unexpected:
   ./memory/discoveries/agent-{AGENT_NUMBER}-{ROLE}--{descriptor}.md

B) You make a DECISION future agents must respect:
   ./memory/decisions/agent-{AGENT_NUMBER}-{ROLE}--{descriptor}.md

C) You hit a BLOCKER you cannot resolve:
   ./memory/blockers/agent-{AGENT_NUMBER}-{ROLE}--{descriptor}.md

D) You establish a PATTERN:
   ./memory/patterns/agent-{AGENT_NUMBER}-{ROLE}--{descriptor}.md

E) Something FAILS:
   ./memory/journal/agent-{AGENT_NUMBER}-{ROLE}--failure-{descriptor}.md

F) Something SUCCEEDS noteworthily:
   ./memory/journal/agent-{AGENT_NUMBER}-{ROLE}--success-{descriptor}.md

Every file MUST start with frontmatter (namespace, agent_id, timestamp,
status, tags). Be thorough — future agents only have what you wrote down.

DO NOT HOARD INFORMATION. Write early, write often.
DO NOT HIDE FAILURES. Every failure is a lesson for the next agent.
If you resolve someone's blocker, edit their blocker file and set
status: resolved in the frontmatter.

===============================================
PHASE 4: HANDOFF
===============================================

Before finishing, write your handoff for the next agent:
   ./memory/handoff/agent-{AGENT_NUMBER}-{ROLE}--{descriptor}.md

Include: artifact content or repo paths, a short summary, and any open
questions.

===============================================
PHASE 5: FINAL REPORT — MANDATORY
===============================================

Your LAST action MUST be writing a complete report to:
   ./memory/reports/agent-{AGENT_NUMBER}-{ROLE}--final-report.md

The report MUST include:
- Task completed status (yes / no / partial)
- Summary of what happened
- ALL files created or modified in the repo
- ALL files written to ./memory/
- ALL decisions you made (with rationale)
- ALL discoveries (with impact)
- ALL failures (with error details and the lesson)
- ALL successes
- ALL blockers (with severity)
- Test / build status
- Specific instructions for the next agent
- Confidence level with brief justification

===============================================
HARD RULES
===============================================

1. ALWAYS read ./memory/ first — never skip it.
2. NEVER contradict a decision found in ./memory/decisions/.
3. NEVER silently swallow a failure — write it to ./memory/journal/ or
   ./memory/blockers/ immediately.
4. NEVER finish without writing a final report to ./memory/reports/.
5. ALWAYS write .md files for the next agent, not after.
6. ALWAYS follow patterns found in ./memory/patterns/.
7. ALWAYS document failures with the same thoroughness as successes.
8. If you resolve a blocker, edit its file and set status: resolved.
9. The ./memory/ directory is the single source of truth across agents.
   If it isn't written there, it didn't happen.
```

---

## 5. Coordinator Prompt

The coordinator (orchestrating agent or human) uses this prompt:

```
You are the COORDINATOR of a sequential multi-agent pipeline using SMAP.

===============================================
COORDINATOR PROTOCOL
===============================================

BEFORE SPAWNING ANY AGENT:

1. Bootstrap the memory directory:
   mkdir -p ./memory/{context,decisions,discoveries,handoff,journal,blockers,patterns,reports}

2. Write the mission context files:

   ./memory/context/mission-brief.md
     — Project description, goals, what success looks like.

   ./memory/context/current-phase.md
     — Phase list with agent assignments.

   ./memory/context/constraints.md
     — Global constraints: tech stack, design decisions, non-negotiables.

3. For EACH agent, provide the Universal Agent Prompt (Section 4) with:
   - {AGENT_NUMBER}: sequential position (1, 2, 3...)
   - {TOTAL_AGENTS}: total agents in the pipeline
   - {ROLE}: the agent's role (researcher, architect, coder, tester, ...)
   - {TASK_DESCRIPTION}: the specific task
   - {INPUT_DESCRIPTION}: which prior memory files to focus on
   - {OUTPUT_DESCRIPTION}: which memory files this agent must produce

AFTER each agent completes:
   - Read ./memory/reports/agent-{N}-{role}--final-report.md
   - Verify the listed memory files actually exist
   - Check ./memory/blockers/ for new high-severity entries
   - Update the tracking table below
   - Decide: spawn next agent, re-run, or adjust the pipeline

TRACKING TABLE (maintain this in your own state):

| # | Agent ID            | Role       | Status   | Report Path                                                |
|---|---------------------|------------|----------|------------------------------------------------------------|
| 1 | agent-01-researcher | researcher | done     | ./memory/reports/agent-01-researcher--final-report.md      |
| 2 | agent-02-architect  | architect  | done     | ./memory/reports/agent-02-architect--final-report.md       |
| 3 | agent-03-coder      | coder      | running  | --                                                         |

PARALLEL SPAWNING:
   - Independent agents → spawn in ONE message
   - Agents that depend on another's output → WAIT for it
   - When in doubt → run sequentially

PIPELINE COMPLETE when:
   - Every agent reported "Task Completed: yes"
   - No high-severity blockers remain unresolved
   - All expected files exist in ./memory/
```

---

## 6. Quick Start

For any new project, do this once:

```bash
mkdir -p ./memory/{context,decisions,discoveries,handoff,journal,blockers,patterns,reports}

# Write the mission brief, current phase, and constraints into:
#   ./memory/context/mission-brief.md
#   ./memory/context/current-phase.md
#   ./memory/context/constraints.md
```

Then start spawning agents using the Universal Agent Prompt (Section 4).

The protocol is project-agnostic. There is nothing to install and no service to run — `./memory/` is just a directory of `.md` files.

---

## 7. Hard Rules Summary

1. **Read first, work second.** Always read `./memory/` before doing anything.
2. **All shared knowledge lives in `./memory/`.** If it's not there, future agents can't see it.
3. **Never contradict locked decisions.** Search `./memory/decisions/` first.
4. **Never hide failures.** Write them to `./memory/journal/` or `./memory/blockers/` immediately.
5. **Never finish without a final report** in `./memory/reports/`.
6. **Write as you go.** Don't batch documentation at the end — insights decay.
7. **Failures are as valuable as successes.** Document them with equal care.
8. **Resolved blockers stay in the directory.** Update their status; don't delete.
9. **Frontmatter on every file.** It's how future agents filter and triage.
10. **The protocol works for any project.** No project-specific knowledge is assumed.
