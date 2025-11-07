# Claude Flow: AI Usage Reference

## Core Execution Model

**Claude Code Task Tool** = Execution (spawns agents)
**Claude Flow MCP Tools** = Coordination (memory, monitoring)

```javascript
// Pattern: MCP coordinates, Task tool executes
mcp__claude-flow__swarm_init { topology: "mesh", maxAgents: 6 }
Task("Backend", "Build API", "backend-dev")
Task("Frontend", "Build UI", "coder")
Task("Tests", "Write tests", "tester")
```

## GOLDEN RULE: Batch Everything in Single Messages

```javascript
// ✅ CORRECT
[One Message]:
  Task("Agent1", "...", "coder")
  Task("Agent2", "...", "tester")
  TodoWrite({ todos: [...10 todos...] })
  Write("file1.js"); Write("file2.js")

// ❌ WRONG
Message 1: Task("Agent1")
Message 2: Task("Agent2")
```

## Commands

### Basic Swarm
```bash
npx claude-flow swarm "<objective>" [--strategy research|development|analysis|testing]
npx claude-flow swarm "<objective>" --mode centralized|hierarchical|mesh|adaptive
npx claude-flow swarm "<objective>" --claude  # Open in Claude Code
```

### Hive Mind (Advanced)
```bash
npx claude-flow hive-mind wizard           # Interactive setup
npx claude-flow hive-mind spawn "<task>"   # Spawn swarm
npx claude-flow hive-mind status           # Check status
npx claude-flow hive-mind metrics          # Performance
```

### Memory Operations
```bash
npx claude-flow memory store --namespace "project/decisions" --key "api-design" --value '{...}'
npx claude-flow memory retrieve --namespace "project/decisions" --key "api-design"
```

### Hooks (Automatic Coordination)
```bash
npx claude-flow hooks pre-task --description "<task>"
npx claude-flow hooks post-task --task-id "<id>"
npx claude-flow hooks post-edit --file "<file>" --format true --update-memory true
```

## Agent Types (64 total)

### Core Development
- **coder** - Implementation
- **reviewer** - Code review
- **tester** - Testing
- **researcher** - Research
- **planner** - Planning

### Coordination
- **hierarchical-coordinator** - Queen-led delegation
- **mesh-coordinator** - Peer-to-peer
- **adaptive-coordinator** - Dynamic topology
- **swarm-memory-manager** - Memory management

### Specialized
- **backend-dev** - Backend development
- **system-architect** - Architecture design
- **code-analyzer** - Analysis
- **tdd-london-swarm** - TDD workflow
- **github-modes** - GitHub automation

**See**: `.claude/agents/*` for all 64 agents

## Agent Selection Guide

| Task | Agent |
|------|-------|
| Simple code | coder |
| Architecture | system-architect |
| API design | backend-dev, api-docs |
| Testing | tester, tdd-london-swarm |
| Review | reviewer, code-review-swarm |
| Research | researcher |
| Coordination | hierarchical-coordinator, mesh-coordinator |

## Topologies

| Complexity | Agents | Topology |
|-----------|--------|----------|
| Simple | 1-3 | centralized |
| Medium | 4-6 | hierarchical |
| Complex | 7+ | mesh or adaptive |

## MCP Tools (Key Ones)

### Coordination
```javascript
mcp__claude-flow__swarm_init({ topology, maxAgents })
mcp__claude-flow__agent_spawn({ type })
mcp__claude-flow__task_orchestrate({ objective })
```

### Monitoring
```javascript
mcp__claude-flow__swarm_status()
mcp__claude-flow__agent_metrics()
mcp__claude-flow__task_results()
```

### Memory
```javascript
mcp__claude-flow__memory_usage({ action: "store|retrieve", namespace, key, value })
```

### GitHub
```javascript
mcp__claude-flow__repo_analyze({ owner, repo })
mcp__claude-flow__pr_enhance({ number })
mcp__claude-flow__code_review({ pr })
```

## Memory Namespaces

- `swarm/[agent]/[step]` - Agent-specific
- `project/decisions` - Project decisions
- `hive/collective` - Hive mind shared
- `coordination/status` - Coordination state

## SPARC Methodology

**S**pecification → **P**seudocode → **A**rchitecture → **R**efinement → **C**ompletion

```bash
npx claude-flow sparc tdd "<feature>"                    # Full TDD workflow
npx claude-flow sparc run architect "<task>"             # Architecture phase
npx claude-flow sparc batch spec,architect,tdd "<task>"  # Parallel execution
```

## Skills (25 total)

Invoked in Claude Code:
```
/skill swarm-orchestration
/skill github-code-review
/skill performance-analysis
/skill hive-mind-advanced
```

**See**: `.claude/skills/*` for all 25 skills

## File Organization

**NEVER save to root**. Use:
- `/src/` - Source code
- `/tests/` - Tests
- `/docs/` - Documentation
- `/config/` - Configuration

## Workflow Example

```javascript
// 1. Optional: Setup coordination
mcp__claude-flow__swarm_init({ topology: "hierarchical", maxAgents: 8 })

// 2. Spawn agents (ALL IN ONE MESSAGE)
Task("Researcher", "Analyze requirements for REST API with auth", "researcher")
Task("Architect", "Design microservices architecture", "system-architect")
Task("Backend Dev", "Implement API endpoints and auth", "backend-dev")
Task("DB Architect", "Design PostgreSQL schema", "code-analyzer")
Task("Tester", "Write Jest tests with 90% coverage", "tester")
Task("Reviewer", "Review code quality and security", "reviewer")

// 3. Batch all todos
TodoWrite({ todos: [
  {content: "Research API patterns", status: "in_progress", activeForm: "..."},
  {content: "Design architecture", status: "pending", activeForm: "..."},
  {content: "Implement endpoints", status: "pending", activeForm: "..."},
  {content: "Design database", status: "pending", activeForm: "..."},
  {content: "Write tests", status: "pending", activeForm: "..."},
  {content: "Code review", status: "pending", activeForm: "..."},
]})

// 4. Agents use hooks automatically for coordination
// 5. Agents share decisions via memory
// 6. Monitor via MCP tools
```

## Agent Coordination Lifecycle

Each agent automatically runs:

**Before work:**
```bash
npx claude-flow hooks pre-task --description "<task>"
npx claude-flow hooks session-restore --session-id "swarm-<id>"
```

**During work:**
```bash
npx claude-flow hooks post-edit --file "<file>" --memory-key "swarm/<agent>/<step>"
```

**After work:**
```bash
npx claude-flow hooks post-task --task-id "<task>"
npx claude-flow hooks session-end --export-metrics true
```

## Quick Reference

### When to Use Claude Flow

**Use for:**
- Multi-step complex tasks
- Need for specialized agents (testing, review, architecture)
- Coordination between multiple subsystems
- Persistent memory across operations
- GitHub automation
- SPARC methodology

**Don't use for:**
- Simple single-file edits
- Quick information queries
- Tasks with no coordination needs

### Optimization

- **Parallel execution** - Spawn all agents in one message (2.8-4.4x faster)
- **Memory sharing** - Store decisions for other agents to retrieve
- **Adaptive topology** - Let system choose optimal coordination
- **ReasoningBank** - Automatically stores learned patterns

### Status Checks

```bash
npx claude-flow swarm status     # Swarm health
npx claude-flow hive-mind status  # Hive mind state
npx claude-flow agent memory status  # Memory usage
```

## Directory Structure

After init:
```
.claude/
  ├── agents/        # 64 agent definitions
  ├── commands/      # Command docs
  ├── skills/        # 25 skill definitions
  └── settings.json  # Hooks and config
.swarm/
  └── memory.db      # ReasoningBank
.hive-mind/
  ├── hive.db        # Collective intelligence
  ├── sessions/      # Session history
  └── config.json    # Configuration
.mcp.json            # MCP servers (4 configured)
```

## Essential MCP Servers

- **ruv-swarm** ✅ - Enhanced coordination
- **flow-nexus** ✅ - Cloud features (optional, requires registration)
- **agentic-payments** ✅ - Payment auth (optional)
- **claude-flow@alpha** ❌ - May fail (use stable instead)

---

**Remember**: Claude Flow coordinates, Claude Code creates. Always batch operations in single messages.
