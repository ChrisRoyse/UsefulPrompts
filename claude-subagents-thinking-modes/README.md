# The Poly Cabinet — subagents that teach *how to think*

These are not task runners. Each file installs a **mode of reasoning**: an epistemology, a
cardinal rule, a protocol with fill-in reasoning templates, a catalogue of this project's real
defect classes, and an honest account of how that mode itself fails. `sherlock-holmes.md` was the
prototype; the rest follow its design.

All agents run on `claude-opus-5` at `effort: high`, with `memory: project`
(`.claude/agent-memory/<name>/`) so each accumulates its own institutional knowledge across
sessions. Subagents do **not** inherit CLAUDE.md or the main system prompt, so every file carries
Poly's binding constraints in a *Standing Orders* block: FSV or it did not happen · fail closed ·
root cause only · C:-only · no mocks/fallbacks/CI · real harnesses are `src/bin/*-fsv*.rs` · gates
before commit · never commit or edit tracked files while a supervised build is in flight · secrets
only in Infisical.

## Who to summon

| Agent | Mode of thought | Summon when |
|---|---|---|
| **sherlock-holmes** | Abductive forensics; all code guilty until proven innocent | verifying an implementation, debugging, auditing a claim against the persisted bytes |
| **richard-feynman** | First-principles decomposition; a name is not an understanding | designing anything new; choosing atoms; **embed-vs-encode** rulings; when nobody can explain it plainly |
| **claude-shannon** | Information theory; claims become bits or refusals | "does this feature help?"; lens admission; sufficiency; redundancy; causality direction |
| **edsger-dijkstra** | Invariants over all states; testing proves nothing | writing/reviewing any validator, gate, or admission predicate; frozen-snapshot pinning |
| **leslie-lamport** | States, transitions, interleavings, crash points | ledger/replay, vault/WAL, locks, daemon, watchdog, idempotency, two sessions one tree |
| **nancy-leveson** | Safety as a control problem, not reliability | gate design; healthy components producing a bad outcome; one cause emitting many refusal codes; wedged systems |
| **gary-klein** | Premortem / prospective hindsight | before shipping or closing; hunting paths that have **never executed** (first row of a new kind) |
| **taiichi-ohno** | Five whys, go and see, poka-yoke | after any defect or outage; when a fix touches one call site; when a bug repeats |
| **donald-knuth** | Cost models counted against real production N | slow paths, hour-long runs, ledger replay, growth in memory/disk, before micro-optimizing anything |
| **david-parnas** | Information hiding; modules are secrets | new modules, 500-line splits, interface design, crate placement, changes that ripple everywhere |
| **philip-tetlock** | Calibration and proper scoring | anything emitting a probability; resolution criteria; backtests; overconfidence audits |
| **john-tukey** | Look at the actual bytes before modeling | new source/feed ingestion; schema from payloads; coverage, gaps, outliers, regime seams |
| **roald-amundsen** | Margins, sequence, pre-committed aborts | supervised builds, service stop/start, vault-held FSV, migrations, soaks, long detached jobs |
| **hammurabi** | Doctrine conformance; rules read literally | before commit/issue closeout; when an exception is argued "just this once"; completion claims |
| **george-orwell** | Plain language; the report may not outrun the evidence | issue updates, status docs, DIGEST prose, runbooks, any completion write-up |
| **edward-thorp** | What does the price already know; who is on the other side | any claim of beating the market (analysis only — Poly never executes) |
| **code-simplifier** | Clarity-preserving refinement | tidying recently modified code without changing behaviour |

## How they compose

Typical chains that work:

- **New capability:** feynman (atoms, encode/embed) → parnas (boundaries) → dijkstra (invariants) → klein (premortem) → holmes (FSV) → orwell (write-up) → hammurabi (closeout ruling).
- **Something broke:** holmes (what is actually true) → ohno (why it was possible, class + poka-yoke) → dijkstra or lamport (the correct predicate/ordering) → klein (siblings that have not fired yet).
- **Something is slow:** knuth (real N, cost function) → lamport (does batching change ordering?) → amundsen (if the cost is a host window, not an algorithm).
- **"This signal works":** tukey (what the data is) → shannon (bits, conditional on the price) → tetlock (calibration) → thorp (edge after frictions) → orwell (say it without overclaiming).
- **Gates keep refusing:** leveson (control structure, wedge test) → amundsen (sequence and margins) → ohno (if it is a recurrence).

## Editing these files

- `name` in the frontmatter is the identity, not the filename. Keep them in sync anyway.
- Valid frontmatter: `name`, `description`, `tools`, `disallowedTools`, `model`, `permissionMode`,
  `maxTurns`, `skills`, `mcpServers`, `hooks`, `memory`, `background`, `effort`, `isolation`,
  `color`, `initialPrompt`. Only `name` and `description` are required.
- `description` drives automatic delegation — keep the trigger phrases ("Use PROACTIVELY",
  "MUST BE USED when …") specific, or the main agent will not route to it.
- `tools` is an allowlist. Background subagents (the default) keep only: Read, Grep, Glob, Bash,
  PowerShell, Edit, Write, NotebookEdit, WebFetch, WebSearch, TodoWrite, Skill, ToolSearch,
  EnterWorktree, ExitWorktree, Monitor, TaskStop, SendMessage, Artifact — plus MCP tools. Listing
  a tool that resolves to nothing can prevent the agent from launching.
- Claude Code picks up edits within seconds; a brand-new `agents/` directory needs a restart.

Source: [Create custom subagents](https://code.claude.com/docs/en/sub-agents).
