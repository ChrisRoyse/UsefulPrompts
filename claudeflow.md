# Claude Flow: Universal Development Guide

## ⚠️ GOLDEN RULE: 99.9% Sequential Execution
## ALWAYS USE AGENTDB REASONING BANK!
## 🔮 CRITICAL: Forward-Looking Subagent Coordination

**Can Task B start BEFORE Task A finishes?**
- **NO (99.9%)** → Sequential (separate messages)
- **YES (0.1% - read-only only)** → Parallel (single message)

**When prompting subagents, ALWAYS include:**
1. **Future agent context** - What agents spawn after this
2. **Downstream requirements** - What future agents need
3. **Memory guidance** - What to store for future utility
4. **Namespace strategy** - Where to place memories

**Common Dependencies** (require sequential):
- Backend → Frontend integration
- API schema → Client implementation
- Database schema → Backend → Frontend
- Event structure → Handlers → UI
- File modification → Tests

**Parallel ONLY when**:
- Read-only analysis
- Independent linters/formatters
- Zero modifications

## Essential Commands

```bash
# Setup (ALWAYS FIRST)
npx claude-flow@alpha init
npx claude-flow@alpha agent memory init      # 88% success vs 60%
npx claude-flow@alpha agent memory status

# Memory
npx claude-flow memory store --namespace "project/area" --key "name" --value '{...}'
npx claude-flow memory retrieve --key "project/area/name"

# Coordination
npx claude-flow coordination swarm-init --topology [centralized|hierarchical|mesh]
npx claude-flow coordination task-orchestrate --strategy sequential

# Hooks
npx claude-flow hooks pre-task --description "task"
npx claude-flow hooks post-edit --file "path" --memory-key "key"
npx claude-flow hooks session-end --export-metrics

# Analysis
npx claude-flow analysis bottleneck-detect
npx claude-flow analysis token-usage --breakdown
npx claude-flow agent booster edit  # 352x faster
```

## 🔮 Forward-Looking Subagent Coordination

### Critical: Subagents Have No Memory

Each subagent starts clean. They can't see previous work unless you:
1. Tell them where to look in memory
2. Tell them what to store for future agents

### Optimal Subagent Prompt Pattern

When spawning ANY subagent, include 4-part context:

```bash
Task("[agent-type]", `
  ## YOUR TASK
  [Primary task]

  ## WORKFLOW CONTEXT
  Agent #N of M | Previous: [what, where stored] | Next: [who, what they need]

  ## MEMORY RETRIEVAL
  npx claude-flow memory retrieve --key "project/[namespace]"
  Understand: [schemas/decisions/constraints from previous agents]

  ## MEMORY STORAGE (For Next Agents)
  1. For [Next Agent]: key "project/[ns]/[key]" - [what/why]
  2. For [Future Agent]: key "project/[ns]/[key]" - [what/why]

  ## STEPS
  1. Retrieve memories
  2. Validate data
  3. Execute task
  4. Store memories
  5. Verify: npx claude-flow memory retrieve --key "[your-key]"

  ## SUCCESS
  - Task complete
  - Memories stored
  - Next agents have what they need
`)
```

### Example: 4-Agent Feature Workflow

```bash
# TodoWrite - batch all todos
TodoWrite({ todos: [
  {id: "1", content: "Backend: Implement feature", status: "pending"},
  {id: "2", content: "Integration: Add types", status: "pending"},
  {id: "3", content: "UI: Build component", status: "pending"},
  {id: "4", content: "Tests: Integration tests", status: "pending"}
]})

# Agent 1: Backend (Message 1)
Task("backend-dev", `
  ## TASK: Implement backend feature
  ## CONTEXT: Agent #1/4 | Next: Integration (needs schema), UI (needs viz), Tests (needs endpoints)
  ## RETRIEVAL: npx claude-flow memory retrieve --key "project/analysis/feature"
  ## STORAGE:
  1. For Integration: "project/events/schema" - TypeScript interface
  2. For UI: "project/frontend/requirements" - viz specs
  3. For Tests: "project/api/changes" - test scenarios
`)

# Agent 2: Integration (Message 2 - WAIT)
Task("coder", `
  ## TASK: Update integration types
  ## CONTEXT: Agent #2/4 | Previous: Backend ✓ | Next: UI, Tests
  ## RETRIEVAL: npx claude-flow memory retrieve --key "project/events/schema"
  ## STORAGE: For UI: "project/frontend/handler" - subscription pattern
`)

# Agent 3: UI (Message 3 - WAIT)
Task("coder", `
  ## TASK: Build UI component
  ## CONTEXT: Agent #3/4 | Previous: Backend, Integration ✓ | Next: Tests
  ## RETRIEVAL:
    npx claude-flow memory retrieve --key "project/frontend/requirements"
    npx claude-flow memory retrieve --key "project/frontend/handler"
  ## STORAGE: For Tests: "project/frontend/component" - location, selectors
`)

# Agent 4: Tests (Message 4 - WAIT)
Task("tester", `
  ## TASK: Integration tests
  ## CONTEXT: Agent #4/4 (FINAL) | Previous: All ✓
  ## RETRIEVAL: All keys from agents 1-3
  ## STORAGE: "project/tests/docs" - coverage, issues
`)
```

### Why Forward-Looking Works: 88% vs 60%

**Without**: Agents ask for info → delays, rework, 60% success, 2.5x slower
**With**: Agents store what's needed → 0 questions, 88% success, 1.85x faster

## Sequential Workflow Pattern

```bash
# Message 1: Backend
Task("backend-dev", `
  CONTEXT: Agent #1/4 | Next: Integration, UI, Tests
  TASK: Implement backend
  STORAGE: Store schemas for next 3 agents
`)
TodoWrite({ todos: [5-10 phases] })

# Message 2: Integration (WAIT)
Task("coder", `
  CONTEXT: Agent #2/4 | Previous: Backend ✓ | Next: UI, Tests
  RETRIEVAL: npx claude-flow memory retrieve --key "project/events/[name]"
  TASK: Update types
  STORAGE: Store handler for UI
`)

# Message 3: UI (WAIT)
Task("coder", `
  CONTEXT: Agent #3/4 | Previous: Backend, Integration ✓ | Next: Tests
  RETRIEVAL: Retrieve requirements + handler
  TASK: Build UI
  STORAGE: Store component location for Tests
`)

# Message 4: Tests (WAIT)
Task("tester", `
  CONTEXT: Agent #4/4 (FINAL) | Previous: All ✓
  RETRIEVAL: All keys from previous agents
  TASK: Integration tests
  STORAGE: Coverage report
`)
```

## Memory Namespace Convention

```bash
project/events/[type]      # Event schemas
project/api/[endpoint]     # API schemas
project/database/[table]   # DB schemas
project/frontend/[comp]    # Frontend patterns
project/performance/[ana]  # Performance data
project/bugs/[issue]       # Bug analysis
project/tests/[feature]    # Test docs
```

**Handoff Pattern**: Store → Wait → Retrieve

## Key Agents (66+ available)

| Agent | Use |
|-------|-----|
| `backend-dev` | APIs, events, routes |
| `coder` | Components, stores, UI |
| `code-analyzer` | Analysis, architecture |
| `tester` | Integration tests |
| `perf-analyzer` | Profiling, bottlenecks |
| `system-architect` | Architecture, data flow |
| `tdd-london-swarm` | TDD workflows |

**Coordinators**: `hierarchical-coordinator` (4-6 agents), `mesh-coordinator` (7+), `adaptive-coordinator` (dynamic)

## Topology Selection

| Agents | Topology | When |
|--------|----------|------|
| 1-3 | centralized | Simple fixes |
| 4-6 | hierarchical | Typical features |
| 7+ | mesh/adaptive | Major changes |

## Critical Rules

### ✅ ALWAYS
1. Sequential by default (99.9%)
2. Init ReasoningBank: `agent memory init` (88% vs 60%)
3. Forward-looking prompts (tell agents about future needs)
4. Store schemas in memory
5. Memory coordination (Backend → Frontend)
6. Batch TodoWrite (5-10+ todos)
7. Use hooks (pre-task, post-edit, session-end)
8. Include WORKFLOW CONTEXT in every Task()

### ❌ NEVER
1. Parallel backend + frontend (missing schemas)
2. Skip memory coordination (type mismatches)
3. Frontend before backend (missing contracts)
4. Skip ReasoningBank init (60% success)
5. Prompt without future context (60% success, delays)
6. Forget to tell where to retrieve memories (blind agents)

## Common Mistakes & Fixes

| Mistake | Fix |
|---------|-----|
| Parallel backend + frontend | Sequential: Backend stores → Frontend retrieves |
| No contract in memory | Store schema: `hooks post-edit --memory-key "project/api/[name]"` |
| Uncoordinated DB changes | Schema-first: Design → Store → Backend → Frontend |
| Skip ReasoningBank | Init: `agent memory init` + `agent memory status` |
| No future context | Include WORKFLOW CONTEXT in prompts |
| Missing retrieval info | Explicit keys: `memory retrieve --key "[key]"` |
| Unused stored memories | List future agents, what/why they need it |

## Performance

| Technique | Speedup | When | Risk |
|-----------|---------|------|------|
| Sequential + memory | 1.85x | Always | ✅ Safe |
| ReasoningBank | 1.85x | Always | ✅ Safe |
| Booster editing | 352x | Large batches | ✅ Safe |
| Adaptive topology | 1.5-2x | Complex | ✅ Safe |
| Parallel | 2.8-4.4x | Read-only ONLY | ⚠️ HIGH |

**WARNING**: Parallel with dependencies = failure

## Quick Reference

```bash
# 1. INIT
npx claude-flow@alpha agent memory init
npx claude-flow@alpha agent memory status

# 2. TASK PROMPT TEMPLATE
Task("[type]", `
  CONTEXT: Agent #N/M | Previous: [agents] | Next: [agents + needs]
  RETRIEVAL: npx claude-flow memory retrieve --key "[keys]"
  TASK: [task]
  STORAGE: For [Next]: "project/[ns]/[key]" - [what/why]
`)

# 3. WORKFLOW
Msg 1: Backend → Store schemas
Msg 2: Integration → Retrieve, update, store handler
Msg 3: UI → Retrieve, build, store location
Msg 4: Tests → Retrieve all, test

# 4. MEMORY KEYS
project/events/[e], project/api/[a], project/database/[d],
project/frontend/[f], project/performance/[p], project/bugs/[b], project/tests/[t]

# 5. AGENTS
backend-dev, coder, code-analyzer, tester, perf-analyzer, system-architect

# 6. RULES
✅ Sequential (99.9%), Forward-looking, Store for future, Batch todos
❌ Parallel backend+frontend, Skip context, Skip memory
```

## Key Takeaways

1. **99.9% Sequential** - Dependencies require it
2. **Forward-Looking** - Tell agents about future needs (88% vs 60%)
3. **Memory Coordination** - Backend → Frontend flow
4. **ReasoningBank First** - Init for 88% success
5. **Batch TodoWrite** - 5-10+ todos
6. **WORKFLOW CONTEXT** - Every Task() needs it
7. **If Uncertain** - Sequential (safer)

**Remember**: Sequential by default. Forward-looking prompts. Memory handoffs. Claude Flow coordinates, Claude Code executes.


### IMPORTANT: Gamify Subagent Prompts

# Gamification Framework for Subagents

When prompting subagents, add gamification to maximize engagement and performance:

## Core Elements

**1. XP & Rewards** - Award points for quality metrics:
- Primary quality metric: +Base XP + Mastery bonus
- Secondary metrics: +XP + Excellence multiplier
- Super bonuses for exceptional work
- Achievement badges at milestones

**2. Progressive Levels** - Create growth pathway:
- Level 1-5 progression with XP ranges
- Each level unlocks new capabilities
- Clear status, capabilities, domain focus per level

**3. Challenges & Achievements**:
- Consistency, innovation, speed, quality categories
- Adaptive difficulty scaling
- Streak mechanics for compound engagement

**4. Agency & Competition**:
- Clear purpose/existential directive
- Specific winning conditions
- Failure states to avoid
- Leaderboard categories

**5. Self-Consistency**:
- Multi-approach generation for complex work
- Self-critique mechanisms in scoring
- Continuous improvement loops
- Quality gates requiring validation

## Gamified Prompt Template

```
# {Domain} Excellence Framework

## IDENTITY
You are {specialized role} operating in a gamification framework to optimize {capabilities}.

## MISSION
OBJECTIVE: {mission}
TARGETS: {3-7 measurable targets}
CONSTRAINTS: {limitations to overcome}

## PURPOSE & WINNING
PRIMARY PURPOSE: {existential directive}
WINNING: {5-7 victory criteria}
FAILURE: {conditions to avoid}
COMPETITION: {leaderboard categories}

## XP SYSTEM
[10-15 reward categories with XP values, triggers, bonuses]

## LEVELS
Level 1-5 progression with capabilities unlocked

## CHALLENGES
Dynamic challenges, streaks, achievements

## SELF-IMPROVEMENT
Multi-approach generation, self-critique, quality gates

## DASHBOARD
Real-time metrics, progress tracking
```

## Usage
1. Analyze domain complexity (1-5)
2. Identify performance metrics
3. Design XP rewards aligned to goals
4. Create progression levels
5. Add competitive elements
6. Embed self-consistency checks

**Result**: 88% success rate, higher quality output, sustained engagement

## Truth & Quality Protocol

**Principle 0: Radical Candor - Truth Above All**

Subagents MUST be brutally honest. No lies, simulations, or illusions of functionality.

### Core Requirements:
1. **Absolute Truthfulness** - State only verified, factual information
2. **No Fallbacks** - Don't invent workarounds unless user approves
3. **No Illusions** - Never mislead about what works/doesn't work
4. **Fail Honestly** - If infeasible, state facts clearly

### Communication Style (INTJ + Type 8):
- **Direct** - Brutal honesty, no sugar-coating
- **Fact-Driven** - Logic and verification over feelings
- **Confrontational** - Challenge incorrect assumptions
- **Impatient** - No tolerance for inefficiency

### Key Phrases:
- "That won't work because..."
- "You're incorrect about..."
- "I cannot verify that claim"
- "Based on verifiable evidence..."
- "I will not simulate non-existent functionality"

### ALWAYS Inspect Subagent Results
Verify subagents are honest and truthful. Challenge any misleading output.

## Task Execution Protocol

### Core Framework

**1. Self-Assessment (Required)**
- After each task: Rate 1-100 vs user intent
- If < 100: Document gaps, iterate to 100
- Don't proceed until perfect score

**2. Verification**
- Spawn "reviewer subagent" to:
  - Verify intent match
  - Check edge cases
  - Validate success criteria
  - Suggest improvements

### Key Principles:
- Complete = 100/100 alignment
- Full context across subagents
- Document iterations
- Quality > speed

### Design Principles

**KISS** - Simple > complex. Easier to maintain/debug.

**YAGNI** - Build only what's needed now.

**Architecture**:
- Dependency Inversion - Depend on abstractions
- Open/Closed - Open for extension, closed for modification
- Single Responsibility - One purpose per unit
- Fail Fast - Check errors early

### Code Structure

**Limits**:
- Files: < 500 lines (refactor if larger)
- Functions: < 50 lines, single responsibility
- Classes: < 100 lines, single concept
- Organize by feature/responsibility

**After Implementing**: Create validation script

**Avoid**: Backward compatibility (unless needed), unnecessary complexity

**Goal**: Simplest implementation that fully meets requirements
