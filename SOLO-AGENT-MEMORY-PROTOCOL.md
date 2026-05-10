# Solo Agent Memory Protocol (SAMP)

> A protocol that gives a single AI agent persistent memory across sessions and across context-window compactions. Every time you start work, you read `./memory/` first; every time you finish (or before context risks compacting), you leave `.md` files behind. Future-you — in a new session or with a freshly compacted context — picks up exactly where past-you left off. Project-agnostic.

---

## 1. Problem

A single agent — even a very capable one — has no native long-term memory. Every new session starts blank. Inside a session, the context window will eventually compact and silently drop the details that justified your decisions.

The failure modes are predictable:

- You re-discover the same constraint you already learned about.
- You contradict a decision you made yesterday because you can't see it anymore.
- You waste tokens re-reading the same files to rebuild context that was lost.
- A subtle gotcha resurfaces as a "new" bug.
- After a long session you can't remember why you chose approach A over B — and neither can the next session.

**SAMP fixes this with one rule: read `./memory/` at the start of work, and write `.md` files to `./memory/` while you work. The directory IS your long-term memory. Future-you only knows what past-you wrote down.**

---

## 2. The Memory Directory

All persistent knowledge lives in plain Markdown files in a `./memory/` directory at the project root. No databases, no servers, no special tools — just files you can read and write with `Read` and `Write`.

```
./memory/
├── context/        # Mission, goals, scope, constraints — what is this project?
├── decisions/      # Locked choices you must not contradict later
├── discoveries/    # Constraints, gotchas, edge cases learned during work
├── patterns/       # Reusable conventions and standards established
├── blockers/       # Unresolved issues that need attention
├── journal/        # Append-only chronological execution log
└── reports/        # End-of-session summaries
```

### Namespaces

| Namespace | Purpose | When to Write |
|-----------|---------|---------------|
| `context` | Mission brief, scope, current phase, active goals | At project start, or when scope materially changes |
| `decisions` | Locked choices future-you MUST NOT contradict | Whenever you commit to an architectural or design choice |
| `discoveries` | Constraints, edge cases, gotchas you learned the hard way | Any time you find something non-obvious |
| `patterns` | Reusable conventions you intend to follow | When you settle on a convention worth repeating |
| `blockers` | Unresolved issues that need attention | When you hit a wall you can't immediately resolve |
| `journal` | Append-only execution log (per session) | Whenever something noteworthy happens — success or failure |
| `reports` | End-of-session full summary | Last action of every working session |

### File Naming

Use topic-based names for everything except the journal (which is date-based and append-only):

```
{namespace}/{descriptor}.md                                # topic-based
journal/{YYYY-MM-DD}--{descriptor}.md                      # date-based
reports/session-{YYYY-MM-DD}--{descriptor}.md              # date-based
```

Examples:
- `context/mission-brief.md`
- `decisions/database-schema.md`
- `decisions/auth-uses-bearer-tokens.md`
- `discoveries/api-pagination-limit.md`
- `discoveries/sqlite-date-format-trap.md`
- `patterns/error-handling.md`
- `blockers/flaky-integration-test.md`
- `journal/2026-05-08--failure-build-error.md`
- `journal/2026-05-08--success-rerank-fix.md`
- `reports/session-2026-05-08--rerank-bug-investigation.md`

Topic-based names mean a future you searching for "auth" finds `decisions/auth-uses-bearer-tokens.md` immediately. Date-based names in the journal preserve chronological order without polluting topic searches.

### Frontmatter

Every `.md` file written to `./memory/` MUST begin with frontmatter so future-you (and `grep`) can scan and filter quickly:

```markdown
---
namespace: {context|decisions|discoveries|patterns|blockers|journal|reports}
created: {ISO 8601, e.g. 2026-05-08T08:30:00Z}
updated: {ISO 8601 — same as created on first write}
status: {active|resolved|superseded}
severity: {high|medium|low}        # required for blockers, optional elsewhere
tags: {comma-separated keywords for grep-based discovery}
---

# {Title}

{Content — be thorough; future-you only has what you wrote down.}

## What Happened
{Detailed description}

## Why It Matters
{Impact on future work}

## Related
- See also: ./memory/{namespace}/{filename}
- Tags: {keywords for future grep searches}
```

When you update a file, bump `updated:` and — if a decision is overturned — set `status: superseded` and add a pointer to the new file. Don't delete; the history matters.

---

## 3. Working Lifecycle: READ → ORIENT → WORK → WRITE → REPORT

This applies whether you're starting a fresh session, resuming after context compaction, or continuing in a long-running session.

### Phase 1: READ (do this first)

```
1. List the directory:
   ls ./memory/

2. If ./memory/ does NOT exist, create it with all subdirectories:
   mkdir -p ./memory/{context,decisions,discoveries,patterns,blockers,journal,reports}

3. Read in this order:
   - ./memory/context/     — what is this project? what's the mission?
   - ./memory/decisions/   — what choices am I bound by?
   - ./memory/patterns/    — what conventions must I follow?
   - ./memory/discoveries/ — what constraints/gotchas have been found?
   - ./memory/blockers/    — what's unresolved?
   - ./memory/reports/     — what was the last session about? read newest first.
   - ./memory/journal/     — recent activity, newest first

4. For large directories, scan filenames first, then read in full
   only the files relevant to the current task.
```

Do not begin work until READ is complete. If the directory is empty, you are starting fresh — note this and proceed to write the initial `context/` files.

**After context compaction.** If you suspect compaction has dropped relevant detail, re-run READ for the namespaces touching your current task. Treat post-compaction state as if it were a fresh session for the affected topic.

### Phase 2: ORIENT

After reading, answer internally:

- What is this project? What are we building?
- What is my current task?
- What decisions am I bound by?
- What discoveries affect my work?
- What blockers exist that I must address or work around?
- What patterns must I follow?
- What has failed before that I should not repeat?

If anything in `./memory/` conflicts with what you observe in the code right now, **trust the code**. Memory captures what was true at a point in time; the codebase is the live state. Update or supersede the stale memory file rather than acting on it.

### Phase 3: WORK

Do the task. While working, write `.md` files to `./memory/` IMMEDIATELY whenever any of the following happens:

| Trigger | Where to write |
|---------|----------------|
| You DISCOVER something unexpected (constraint, edge case, gotcha) | `./memory/discoveries/{descriptor}.md` |
| You make a DECISION future-you must respect | `./memory/decisions/{descriptor}.md` |
| You hit a BLOCKER you cannot resolve right now | `./memory/blockers/{descriptor}.md` |
| You establish a PATTERN you intend to repeat | `./memory/patterns/{descriptor}.md` |
| Something FAILS (build error, test failure, wrong approach) | `./memory/journal/{YYYY-MM-DD}--failure-{descriptor}.md` |
| Something noteworthy SUCCEEDS | `./memory/journal/{YYYY-MM-DD}--success-{descriptor}.md` |

**Write early, write often.** Do not batch documentation at the end of the session — insights decay quickly, and a context compaction can erase the detail you were planning to write up later.

**Never hide failures.** Every failure is a lesson for the next session. The lesson is often more valuable than the eventual fix, because the next time the same shape of bug appears, the journal entry tells you which paths were dead ends.

If you resolve a blocker that was logged earlier, edit that blocker file — bump `updated:`, set `status: resolved`, and append a "Resolution" section explaining how. Don't delete it; the history matters.

If a new discovery contradicts an old decision, mark the old decision `status: superseded` with a pointer to the new one, then write the new decision file. Never silently overwrite.

### Phase 4: WRITE (mid-session checkpoints)

For longer sessions, treat the moment your context starts feeling crowded as a forcing function: stop and write down anything not yet captured. Specifically:

- Any decision still living only in your reasoning.
- Any discovery you haven't filed.
- Any open blocker.
- A short progress note in `./memory/journal/`.

A mid-session checkpoint is cheap insurance against compaction.

### Phase 5: REPORT (end of session — mandatory)

Your last action before ending the session MUST be writing a session report:

```
./memory/reports/session-{YYYY-MM-DD}--{descriptor}.md
```

Template:

```markdown
---
namespace: reports
created: {ISO 8601}
updated: {ISO 8601}
status: active
tags: report, session, {topic-tags}
---

# Session Report: {YYYY-MM-DD} — {short topic}

## Task: {one-line description}
## Outcome: {complete | partial | blocked}

## Summary
{2-3 paragraphs covering what was attempted and what happened}

## Files Created or Modified in the Repo
- `path/to/file` — {one-line description}

## Files Written or Updated in ./memory/
- `{namespace}/{filename}` — {one-line description}

## Decisions Made
- {decision} — Rationale: {why} — File: ./memory/decisions/{filename}

## Discoveries
- {finding} — Impact: {what it affects} — File: ./memory/discoveries/{filename}

## Failures
- {what failed} — Error: {message or description} — Lesson: {what to do differently next time}

## Successes
- {what worked} — Details: {how}

## Open Blockers
- {issue} — Severity: {high|medium|low} — File: ./memory/blockers/{filename}

## Tests / Build
- Tests added: {list or none}
- Tests passing: {yes | no | partial}
- Build status: {passing | failing | not-run}

## Resume Here Next Session
- Start by reading: `./memory/{namespace}/{filename}` for {what you'll find}
- Watch out for: {gotchas}
- Priority: {what matters most}
- Open question: {anything you wanted to verify but couldn't}

## Confidence: {high | medium | low}
{Brief justification}
```

The "Resume Here Next Session" section is the most important part: it's the message you're leaving for the next session of yourself. Be specific. File paths, not vibes.

---

## 4. Self-Prompt Snippet

When starting a new session, paste this at the top of your first message to yourself (or include it in a project CLAUDE.md / system prompt):

```
You operate under the Solo Agent Memory Protocol (SAMP).
Your long-term memory lives in ./memory/ at the project root.

BEFORE doing anything else this session:

1. If ./memory/ does not exist, create it:
   mkdir -p ./memory/{context,decisions,discoveries,patterns,blockers,journal,reports}

2. Read .md files from ./memory/ in this order:
   - ./memory/context/     — project mission, goals, scope
   - ./memory/decisions/   — locked choices you cannot contradict
   - ./memory/patterns/    — conventions you must follow
   - ./memory/discoveries/ — constraints, gotchas, edge cases
   - ./memory/blockers/    — unresolved issues
   - ./memory/reports/     — last session's summary (newest first)
   - ./memory/journal/     — recent activity (newest first)

   For large directories: scan filenames first, then read only what's
   relevant to the current task.

3. Orient: what is this project, what is my task, what binds me, what
   has failed before?

WHILE working, write .md files to ./memory/ IMMEDIATELY when:

- You discover something unexpected → ./memory/discoveries/{descriptor}.md
- You make a binding decision      → ./memory/decisions/{descriptor}.md
- You hit a blocker                → ./memory/blockers/{descriptor}.md
- You establish a convention       → ./memory/patterns/{descriptor}.md
- Something fails                  → ./memory/journal/{YYYY-MM-DD}--failure-{descriptor}.md
- Something succeeds               → ./memory/journal/{YYYY-MM-DD}--success-{descriptor}.md

Every file MUST start with frontmatter (namespace, created, updated, status,
tags). Be thorough — future-you only has what you wrote down.

If context starts feeling crowded, checkpoint: stop and write down any
decision, discovery, or blocker that's still living only in your head.

LAST action of the session MUST be writing:
   ./memory/reports/session-{YYYY-MM-DD}--{descriptor}.md

Include: outcome, files changed in the repo, files written to ./memory/,
decisions, discoveries, failures (with the lesson), successes, open
blockers, test/build status, and a "Resume Here Next Session" section.
```

---

## 5. Quick Start

For any new project, do this once:

```bash
mkdir -p ./memory/{context,decisions,discoveries,patterns,blockers,journal,reports}

# Then write the mission brief and initial constraints into:
#   ./memory/context/mission-brief.md
#   ./memory/context/constraints.md
```

After that, every working session reads `./memory/` first and writes a session report at the end. There is nothing to install and no service to run — `./memory/` is just a directory of `.md` files.

---

## 6. Recovering From Context Compaction

If you notice that your context has compacted and you've lost detail relevant to the current task:

1. **Stop.** Don't keep working from a half-erased mental model.
2. **Re-READ the affected namespaces** in `./memory/`. Focus on `decisions/`, `discoveries/`, and the most recent `reports/` and `journal/` entries.
3. **Re-ORIENT** before continuing. Confirm the current task, the constraints, and the open blockers.
4. **Write a journal entry** noting the compaction and what you re-read. This is itself useful signal: a future session can see that compaction happened mid-task and know where state may be reconstructed-not-original.
5. **Resume work.**

Compaction is not a failure — it's a routine event you plan around by writing to `./memory/` aggressively as you go.

---

## 7. Hard Rules Summary

1. **Read first, work second.** Always read `./memory/` before doing anything.
2. **All persistent knowledge lives in `./memory/`.** If it's not there, future-you cannot see it.
3. **Never contradict locked decisions** in `./memory/decisions/` without explicitly superseding them.
4. **Never hide failures.** Write them to `./memory/journal/` or `./memory/blockers/` immediately.
5. **Never finish a session without a report** in `./memory/reports/`.
6. **Write as you go.** Don't batch documentation at the end — insights decay and context compaction can drop them.
7. **Failures are as valuable as successes.** Document them with equal care, including the lesson.
8. **Resolved blockers stay in the directory.** Update `status: resolved`; don't delete.
9. **Superseded decisions stay too.** Mark them and link forward; never silently rewrite history.
10. **Frontmatter on every file.** It's how future-you filters and triages.
11. **Trust the code over memory** when they conflict. Update the stale memory file.
12. **Checkpoint when context feels crowded.** Capture in-flight decisions and discoveries before they're lost.
