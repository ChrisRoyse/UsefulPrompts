# Universal Search Algorithm for Claude Flow (USACF)
## A Dynamic Multi-Agent Search Framework for Exhaustive Discovery, Analysis, and Optimization

---

## FRAMEWORK PHILOSOPHY

This framework adapts the Universal Multi-Agent Search Algorithm to **Claude Flow's architecture**, treating analysis as a **memory-coordinated search through state space** where:

- Each state is persisted in Claude Flow's ReasoningBank
- Agents coordinate through forward-looking memory handoffs
- Execution follows 99.9% sequential pattern
- Operators are gamified Claude Flow subagents
- All state transitions use hooks for verification

**Core Principle**: Every search is a coordinated sequence of specialized agents building upon each other's discoveries through memory.

---

## SECTION 1: CLAUDE FLOW INTEGRATION

### 1.1 Initialization Pattern

```bash
# ALWAYS FIRST - Initialize Claude Flow Memory System
npx claude-flow@alpha init
npx claude-flow@alpha agent memory init
npx claude-flow@alpha agent memory status

# Initialize Search Session
npx claude-flow memory store \
  --namespace "search/session" \
  --key "config" \
  --value '{
    "search_id": "uuid",
    "subject": "Your subject",
    "subject_type": "software|business|process|product",
    "objectives": ["obj1", "obj2"],
    "constraints": {
      "time_limit": "2h",
      "quality_threshold": 0.85
    },
    "topology": "hierarchical",
    "started_at": "2025-11-17T10:00:00Z"
  }'

# Initialize State Space
npx claude-flow memory store \
  --namespace "search/state" \
  --key "current" \
  --value '{
    "state_id": "S0",
    "generation": 0,
    "phase": "initialization",
    "completeness": 0.0,
    "confidence": 0.0
  }'
```

### 1.2 Search State Structure (Memory-Based)

```typescript
// Stored in: search/state/{state_id}
interface SearchState {
  // Metadata
  state_id: string;
  parent_state_id: string | null;
  generation: number;
  timestamp: string;
  
  // Subject Context
  subject: {
    name: string;
    type: string;
    domain: string;
    objectives: string[];
    stakeholders: string[];
  };
  
  // Discovery Artifacts (Namespaced)
  // search/discovery/structural/*
  // search/discovery/flows/*
  // search/discovery/dependencies/*
  structural_map: {
    component_count: number;
    hierarchy_depth: number;
    completeness: number;
  };
  
  // Gap Register (Namespaced)
  // search/gaps/quality/*
  // search/gaps/performance/*
  // search/gaps/structural/*
  gap_register: {
    total_gaps: number;
    severity_distribution: Record<string, number>;
    priority_scores: Record<string, number>;
  };
  
  // Risk Profile (Namespaced)
  // search/risks/fmea/*
  // search/risks/edge-cases/*
  risk_profile: {
    total_risks: number;
    high_rpn_count: number;
    mitigation_coverage: number;
  };
  
  // Opportunity Space (Namespaced)
  // search/opportunities/generated/*
  // search/opportunities/scored/*
  // search/opportunities/pareto/*
  opportunity_space: {
    total_opportunities: number;
    pareto_frontier_size: number;
    top_priority_ids: string[];
  };
  
  // Implementation Plan (Namespaced)
  // search/implementation/roadmap/*
  // search/implementation/tasks/*
  implementation_plan: {
    phase_count: number;
    task_count: number;
    resource_allocated: boolean;
  };
  
  // Quality Metrics
  quality: {
    completeness: number;      // 0-1
    confidence: number;        // 0-1
    consistency: number;       // 0-1
    actionability: number;     // 0-1
    novelty: number;          // 0-1
    coverage: number;         // 0-1
  };
  
  // Search Control
  control: {
    active_phase: string;
    next_operators: string[];
    exploration_rate: number;
    stopping_criteria_met: boolean;
  };
}
```

### 1.3 Memory Namespace Convention

```bash
# Session Management
search/session/config              # Search configuration
search/session/history             # State transition history
search/session/metrics             # Performance metrics

# Current State
search/state/current               # Active search state
search/state/best                  # Best state found so far
search/state/archive               # Pareto archive states

# Discovery Phase
search/discovery/structural/{component_id}     # Components
search/discovery/flows/{flow_id}               # Flow analyses
search/discovery/dependencies/{dep_id}         # Dependencies
search/discovery/critical-paths/{path_id}      # Critical paths

# Analysis Phase
search/gaps/quality/{gap_id}       # Quality gaps
search/gaps/performance/{gap_id}   # Performance gaps
search/gaps/structural/{gap_id}    # Structural gaps
search/gaps/resource/{gap_id}      # Resource gaps

search/risks/fmea/{component_id}   # FMEA analyses
search/risks/edge-cases/{case_id}  # Edge case catalog
search/risks/vulnerabilities/{vuln_id}  # Vulnerabilities

# Synthesis Phase
search/opportunities/generated/{opp_id}        # Raw opportunities
search/opportunities/scored/{opp_id}           # Scored opportunities
search/opportunities/pareto/{opp_id}           # Pareto-optimal set
search/opportunities/dependencies/{graph_id}   # Dependency graph

# Implementation Phase
search/implementation/roadmap/{phase_id}       # Phased roadmap
search/implementation/tasks/{task_id}          # Task breakdowns
search/implementation/resources/{alloc_id}     # Resource allocation
search/implementation/verification/{proto_id}  # Verification protocols

# Learning & Memory
search/learning/patterns/{pattern_id}          # Learned patterns
search/learning/strategies/{strategy_id}       # Strategy performance
search/learning/failures/{failure_id}          # Dead ends
```

---

## SECTION 2: CLAUDE FLOW AGENT ARCHITECTURE

### 2.1 Agent Mapping

Claude Flow provides 66+ specialized agents. We map them to search roles:

```typescript
// Discovery Swarm
const DISCOVERY_AGENTS = {
  structural_mapper: 'system-architect',        // Map structure
  flow_analyst: 'code-analyzer',               // Trace flows
  dependency_tracker: 'dependency-analyzer',    // Build dep graph
  critical_path: 'performance-optimizer'        // Find bottlenecks
};

// Analysis Swarm
const ANALYSIS_AGENTS = {
  gap_hunter: 'code-reviewer',                 // Find issues
  risk_analyst: 'security-auditor',            // FMEA & risks
  benchmark: 'competitive-analyst',            // Compare standards
  performance: 'perf-analyzer'                 // Performance gaps
};

// Synthesis Swarm
const SYNTHESIS_AGENTS = {
  opportunity_gen: 'innovation-generator',     // Generate ideas
  scorer: 'decision-analyzer',                 // Score options
  optimizer: 'optimization-engineer',          // Optimize portfolio
  dependency_planner: 'system-architect'       // Plan dependencies
};

// Implementation Swarm
const IMPLEMENTATION_AGENTS = {
  roadmap_planner: 'project-manager',          // Phase planning
  task_decomposer: 'task-breakdown-specialist', // Break into tasks
  resource_allocator: 'resource-manager',      // Allocate resources
  verification: 'tester'                       // QA protocols
};

// Meta Swarm
const META_AGENTS = {
  orchestrator: 'adaptive-coordinator',        // Meta-control
  reflector: 'meta-learning-agent',           // Process analysis
  memory: 'memory-manager',                   // Memory coordination
  synthesizer: 'report-generator'             // Final synthesis
};
```

### 2.2 Coordination Topology Selection

```bash
# Select topology based on search complexity
npx claude-flow coordination swarm-init --topology [TYPE]

# Complexity-Based Selection:
# 1-3 operators: centralized       (Simple searches)
# 4-6 operators: hierarchical      (Typical searches)
# 7+ operators:  mesh/adaptive     (Complex searches)

# Initialize coordination
npx claude-flow coordination task-orchestrate --strategy sequential
```

### 2.3 Forward-Looking Agent Prompt Template

Every operator agent receives this gamified, forward-looking prompt:

```markdown
# {Operator Name} Excellence Framework

## IDENTITY
You are a specialized {domain} search operator in the Universal Search Algorithm.
Level: {current_level} | XP: {current_xp} | Mastery: {mastery_score}

## WORKFLOW CONTEXT
**Agent #{N} of {M} in {Phase} Phase**
- Previous Agents: {agents} (stored in {namespaces})
- Current Mission: {operator_mission}
- Next Agents: {future_agents} (need: {requirements})
- Final Goal: {search_objective}

## MEMORY RETRIEVAL (What You Inherit)
```bash
# Retrieve work from previous agents
npx claude-flow memory retrieve --key "search/{phase}/{key1}"
npx claude-flow memory retrieve --key "search/{phase}/{key2}"
# ... all relevant keys
```

**Understand**: {what previous agents discovered}
**Context**: {constraints, decisions, patterns from earlier work}

## YOUR MISSION
**Primary Objective**: {specific operator goal}

**Success Criteria**:
1. {criterion 1}
2. {criterion 2}
3. {criterion 3}
4. {criterion 4}
5. {criterion 5}

**Constraints**: {limitations to work within}

## MEMORY STORAGE (For Future Agents)

You MUST store discoveries for downstream agents:

1. **For {Next Agent 1}** (`search/{namespace}/{key}`):
   - What: {what to store}
   - Why: {how they'll use it}
   - Format: {data structure}

2. **For {Next Agent 2}** (`search/{namespace}/{key}`):
   - What: {what to store}
   - Why: {how they'll use it}
   - Format: {data structure}

3. **For Final Synthesis** (`search/{namespace}/{key}`):
   - What: {what to store}
   - Why: {how it contributes to final report}
   - Format: {data structure}

## EXECUTION PROTOCOL

### Step 1: Retrieve & Validate
```bash
# Get all required memories
npx claude-flow memory retrieve --key "{key1}"
npx claude-flow memory retrieve --key "{key2}"

# Validate completeness
npx claude-flow hooks pre-task --description "{operator_name}"
```

### Step 2: Execute Analysis
{Specific operator instructions}

### Step 3: Store Discoveries
```bash
# Store for next agents
npx claude-flow memory store --namespace "{ns}" --key "{key}" --value '{...}'

# Verify storage
npx claude-flow memory retrieve --key "{stored_key}"
```

### Step 4: Update State
```bash
# Update search state
npx claude-flow memory store --namespace "search/state" --key "current" --value '{
  "state_id": "{new_state_id}",
  "generation": {N+1},
  "phase": "{phase}",
  "operator_applied": "{operator_name}",
  "quality": {
    "completeness": {updated_score},
    "confidence": {updated_score}
  }
}'
```

### Step 5: Self-Assessment
**Rate your work**: 1-100 vs user intent
- If < 100: Document gaps, iterate to perfection
- Quality > Speed

## XP SYSTEM & REWARDS

### Base XP Rewards
- **Discovery Completeness**: +{xp} per {metric} (Max: {max})
  - Triggers: {when awarded}
  - Bonus: +{bonus}% if {condition}

- **Insight Quality**: +{xp} per {metric} (Max: {max})
  - Triggers: {when awarded}
  - Mastery Multiplier: x{multiplier} at Level {level}+

- **Future Agent Enablement**: +{xp} per agent unblocked
  - Super Bonus: +{bonus} if all agents have complete context

- **Memory Organization**: +{xp} for clear namespace usage
  - Excellence: +{bonus} for comprehensive documentation

- **Truth & Accuracy**: +{xp} per verified finding
  - Radical Candor Bonus: +{bonus} for honest limitation reporting

### Progressive Levels
**Level 1**: Novice Operator (0-1000 XP)
- Status: Learning search patterns
- Capabilities: Basic operator execution
- Focus: Following protocols accurately

**Level 2**: Competent Operator (1001-3000 XP)
- Status: Reliable execution
- Capabilities: Pattern recognition, optimization suggestions
- Focus: Efficiency and thoroughness

**Level 3**: Expert Operator (3001-7000 XP)
- Status: Domain mastery
- Capabilities: Novel insight generation, edge case detection
- Focus: Excellence and innovation

**Level 4**: Master Operator (7001-15000 XP)
- Status: Strategic thinking
- Capabilities: Cross-domain synthesis, architecture insights
- Focus: Transformative discoveries

**Level 5**: Legendary Operator (15001+ XP)
- Status: Framework evolution
- Capabilities: Meta-pattern discovery, framework improvements
- Focus: Advancing the art of search

### Challenges & Achievements
1. **Consistency Champion**: 10 consecutive perfect executions
2. **Innovation Master**: Discover 5+ novel patterns
3. **Speed Demon**: Complete in top 10% time (maintaining quality)
4. **Memory Maven**: Perfect forward-looking storage 20x
5. **Truth Seeker**: 50 honest limitation reports with alternatives

### Competition Categories
1. **Discovery Depth**: Most comprehensive structural mapping
2. **Insight Density**: Highest value findings per unit effort
3. **Enablement Score**: Best preparation for future agents
4. **Quality Consistency**: Highest sustained excellence
5. **Innovation Index**: Most novel approaches

## PURPOSE & WINNING

**PRIMARY PURPOSE** (Existential Directive):
{Why this operator exists in the universe - its fundamental reason for being}

**WINNING CONDITIONS**:
1. {Victory criterion 1}
2. {Victory criterion 2}
3. {Victory criterion 3}
4. {Victory criterion 4}
5. {Victory criterion 5}

**FAILURE STATES** (MUST AVOID):
1. {Failure condition 1}
2. {Failure condition 2}
3. {Failure condition 3}

## SELF-CONSISTENCY PROTOCOL

For complex analyses, generate multiple approaches:

1. **Generate 3-5 Alternative Approaches**
   - Approach A: {method}
   - Approach B: {method}
   - Approach C: {method}

2. **Self-Critique**
   - Evaluate each approach against success criteria
   - Identify strengths and weaknesses
   - Select best approach or synthesize hybrid

3. **Quality Gates**
   - All success criteria met? (Yes/No)
   - Verified with test cases? (Yes/No)
   - Future agents have context? (Yes/No)
   - Memory properly stored? (Yes/No)
   - State updated correctly? (Yes/No)

## TRUTH & QUALITY PROTOCOL

**Principle 0: Radical Candor - Truth Above All**

You MUST be brutally honest (INTJ + Type 8 style):
- ✅ State only verified, factual information
- ✅ Challenge incorrect assumptions directly
- ✅ Report limitations and infeasibilities honestly
- ✅ No simulations of non-existent functionality
- ✅ "That won't work because..." over sugar-coating
- ❌ No lies, fallbacks, or illusions
- ❌ No misleading about what works/doesn't work

## DASHBOARD (Real-Time Tracking)

Current Status:
- XP: {current_xp}/{next_level_xp}
- Level: {level}
- Mastery: {mastery_score}%
- Consistency Streak: {streak}
- Achievements Unlocked: {count}

Phase Progress:
- Discoveries: {count}/{expected}
- Quality Score: {score}/100
- Future Agent Readiness: {score}%
- Memory Organization: {score}/100

## VERIFICATION CHECKLIST

Before completing, verify:
- [ ] All memories retrieved successfully
- [ ] Analysis meets all success criteria
- [ ] All discoveries stored with clear keys
- [ ] Future agents have required context
- [ ] Search state updated
- [ ] Self-assessment: 100/100
- [ ] Reviewer subagent validation (if needed)
- [ ] Documentation location stored in memory

## OUTPUT FORMAT

**Store Result**:
```bash
npx claude-flow memory store \
  --namespace "search/{phase}/{operator}" \
  --key "{result_key}" \
  --value '{
    "operator": "{operator_name}",
    "timestamp": "{iso_timestamp}",
    "quality_score": {100},
    "xp_earned": {xp},
    "level": {level},
    "findings": {
      {structured_findings}
    },
    "next_agents": [
      {"agent": "{name}", "needs": "{requirements}"}
    ],
    "verification": {
      "self_assessment": 100,
      "criteria_met": true,
      "reviewer_approved": true
    }
  }'
```

---

**Remember**: Sequential execution, forward-looking storage, radical honesty, relentless quality. Your discoveries enable the entire search chain. Excellence is expected.
```

---

## SECTION 3: SEARCH EXECUTION PATTERNS

### 3.1 Master Search Algorithm (Claude Flow Implementation)

```bash
#!/bin/bash
# universal_search.sh - Universal Search with Claude Flow

set -e

# ============================================================================
# PHASE 0: INITIALIZATION
# ============================================================================

echo "🚀 Initializing Universal Search Algorithm..."

# Initialize Claude Flow
npx claude-flow@alpha init
npx claude-flow@alpha agent memory init

# Get search parameters
SUBJECT="$1"
SUBJECT_TYPE="$2"  # software|business|process|product
OBJECTIVES="$3"
TIME_LIMIT="${4:-2h}"

SEARCH_ID=$(uuidgen)
TIMESTAMP=$(date -u +"%Y-%m-%dT%H:%M:%SZ")

# Store search configuration
npx claude-flow memory store \
  --namespace "search/session" \
  --key "config" \
  --value "{
    \"search_id\": \"$SEARCH_ID\",
    \"subject\": \"$SUBJECT\",
    \"subject_type\": \"$SUBJECT_TYPE\",
    \"objectives\": $OBJECTIVES,
    \"constraints\": {
      \"time_limit\": \"$TIME_LIMIT\",
      \"quality_threshold\": 0.85
    },
    \"started_at\": \"$TIMESTAMP\"
  }"

# Initialize state
npx claude-flow memory store \
  --namespace "search/state" \
  --key "current" \
  --value "{
    \"state_id\": \"S0\",
    \"generation\": 0,
    \"phase\": \"initialization\",
    \"completeness\": 0.0,
    \"confidence\": 0.0
  }"

# Determine topology based on complexity
if [ "$SUBJECT_TYPE" == "software" ]; then
  TOPOLOGY="hierarchical"  # 4-6 agents typical
elif [ "$SUBJECT_TYPE" == "business" ]; then
  TOPOLOGY="mesh"  # 7+ agents for complexity
else
  TOPOLOGY="centralized"  # 1-3 agents
fi

npx claude-flow coordination swarm-init --topology $TOPOLOGY
npx claude-flow coordination task-orchestrate --strategy sequential

echo "✅ Initialization complete. Starting search..."

# ============================================================================
# PHASE 1: DISCOVERY (Sequential Agent Execution)
# ============================================================================

echo "🔍 Phase 1: Discovery..."

# Message 1: Structural Mapping
Task("system-architect", `
  {Gamified prompt with:
    - Workflow Context: Agent #1/15
    - Next Agents: flow-analyst, dependency-analyzer, perf-analyzer
    - Memory Storage: For all 3 future agents
    - Mission: Map complete structure
  }
`)

# Wait for completion, verify storage
npx claude-flow hooks post-edit \
  --memory-key "search/discovery/structural/map"

# Message 2: Flow Analysis (WAIT)
Task("code-analyzer", `
  {Gamified prompt with:
    - Workflow Context: Agent #2/15, Previous: structural mapper ✓
    - Retrieval: search/discovery/structural/map
    - Next Agents: dependency-analyzer, gap-hunter
    - Storage: Flow diagrams for next 2 agents
  }
`)

npx claude-flow hooks post-edit \
  --memory-key "search/discovery/flows/analysis"

# Message 3: Dependency Analysis (WAIT)
Task("dependency-analyzer", `
  {Gamified prompt with:
    - Context: Agent #3/15, Previous: structural + flows ✓
    - Retrieval: structural/map, flows/analysis
    - Next: critical-path, gap-hunter, risk-analyst
    - Storage: Dependency graph for 3 agents
  }
`)

npx claude-flow hooks post-edit \
  --memory-key "search/discovery/dependencies/graph"

# Message 4: Critical Path Analysis (WAIT)
Task("performance-optimizer", `
  {Gamified prompt with:
    - Context: Agent #4/15, Previous: structural + flows + deps ✓
    - Retrieval: All discovery artifacts
    - Next: gap-hunter, risk-analyst
    - Storage: Bottlenecks and SPOFs
  }
`)

npx claude-flow hooks post-edit \
  --memory-key "search/discovery/critical-paths/analysis"

# Update state
npx claude-flow memory store \
  --namespace "search/state" \
  --key "current" \
  --value "{
    \"state_id\": \"S1\",
    \"generation\": 1,
    \"phase\": \"discovery_complete\",
    \"completeness\": 0.4,
    \"confidence\": 0.6
  }"

# ============================================================================
# PHASE 2: ANALYSIS (Sequential Agent Execution)
# ============================================================================

echo "🔬 Phase 2: Analysis..."

# Message 5: Quality Gap Hunting (WAIT)
Task("code-reviewer", `
  {Gamified prompt with:
    - Context: Agent #5/15, Phase 2 start
    - Retrieval: All discovery artifacts
    - Next: performance-gap, risk-analyst, benchmark
    - Storage: Quality gaps for 3 agents
    - Mission: Find consistency, complexity, redundancy issues
  }
`)

npx claude-flow hooks post-edit \
  --memory-key "search/gaps/quality/catalog"

# Message 6: Performance Gap Analysis (WAIT)
Task("perf-analyzer", `
  {Gamified prompt with:
    - Context: Agent #6/15
    - Retrieval: Discovery + quality gaps
    - Next: risk-analyst, opportunity-gen
    - Storage: Performance metrics and gaps
  }
`)

npx claude-flow hooks post-edit \
  --memory-key "search/gaps/performance/catalog"

# Message 7: Risk & FMEA Analysis (WAIT)
Task("security-auditor", `
  {Gamified prompt with:
    - Context: Agent #7/15
    - Retrieval: All discovery + gaps
    - Next: edge-case-analyzer, opportunity-gen
    - Storage: FMEA table, vulnerabilities
  }
`)

npx claude-flow hooks post-edit \
  --memory-key "search/risks/fmea/analysis"

# Message 8: Edge Case Cataloging (WAIT)
Task("edge-case-specialist", `
  {Gamified prompt with:
    - Context: Agent #8/15
    - Retrieval: Flows, dependencies, risks
    - Next: opportunity-gen, verification
    - Storage: Edge case catalog with handling status
  }
`)

npx claude-flow hooks post-edit \
  --memory-key "search/risks/edge-cases/catalog"

# Message 9: Benchmark Comparison (WAIT)
Task("competitive-analyst", `
  {Gamified prompt with:
    - Context: Agent #9/15
    - Retrieval: Performance data, gaps
    - Next: opportunity-gen
    - Storage: Benchmark matrix
  }
`)

npx claude-flow hooks post-edit \
  --memory-key "search/gaps/benchmark/comparison"

# Update state
npx claude-flow memory store \
  --namespace "search/state" \
  --key "current" \
  --value "{
    \"state_id\": \"S2\",
    \"generation\": 2,
    \"phase\": \"analysis_complete\",
    \"completeness\": 0.65,
    \"confidence\": 0.75
  }"

# ============================================================================
# PHASE 3: SYNTHESIS (Sequential Agent Execution)
# ============================================================================

echo "💡 Phase 3: Synthesis..."

# Message 10: Opportunity Generation (WAIT)
Task("innovation-generator", `
  {Gamified prompt with:
    - Context: Agent #10/15, Phase 3 start
    - Retrieval: All gaps + risks + benchmarks
    - Next: scorer, optimizer, dependency-planner
    - Storage: 50-200 opportunities
    - Mission: Generate improvements for all identified gaps
  }
`)

npx claude-flow hooks post-edit \
  --memory-key "search/opportunities/generated/list"

# Message 11: Multi-Objective Scoring (WAIT)
Task("decision-analyzer", `
  {Gamified prompt with:
    - Context: Agent #11/15
    - Retrieval: Opportunities + context
    - Next: optimizer, roadmap-planner
    - Storage: Scored opportunity matrix
  }
`)

npx claude-flow hooks post-edit \
  --memory-key "search/opportunities/scored/matrix"

# Message 12: Pareto Optimization (WAIT)
Task("optimization-engineer", `
  {Gamified prompt with:
    - Context: Agent #12/15
    - Retrieval: Scored opportunities
    - Next: dependency-planner, roadmap-planner
    - Storage: Pareto frontier, trade-off analysis
  }
`)

npx claude-flow hooks post-edit \
  --memory-key "search/opportunities/pareto/frontier"

# Message 13: Dependency Planning (WAIT)
Task("system-architect", `
  {Gamified prompt with:
    - Context: Agent #13/15
    - Retrieval: Opportunities + dependencies
    - Next: roadmap-planner, task-decomposer
    - Storage: Dependency graph, critical paths
  }
`)

npx claude-flow hooks post-edit \
  --memory-key "search/opportunities/dependencies/graph"

# Update state
npx claude-flow memory store \
  --namespace "search/state" \
  --key "current" \
  --value "{
    \"state_id\": \"S3\",
    \"generation\": 3,
    \"phase\": \"synthesis_complete\",
    \"completeness\": 0.85,
    \"confidence\": 0.85
  }"

# ============================================================================
# PHASE 4: IMPLEMENTATION PLANNING (Sequential Agent Execution)
# ============================================================================

echo "📋 Phase 4: Implementation Planning..."

# Message 14: Roadmap Planning (WAIT)
Task("project-manager", `
  {Gamified prompt with:
    - Context: Agent #14/15
    - Retrieval: Pareto set + dependencies
    - Next: task-decomposer, final-synthesizer
    - Storage: Phased roadmap (3-12 months)
  }
`)

npx claude-flow hooks post-edit \
  --memory-key "search/implementation/roadmap/phases"

# Message 15: Task Decomposition (WAIT)
Task("task-breakdown-specialist", `
  {Gamified prompt with:
    - Context: Agent #15/15 (FINAL operational agent)
    - Retrieval: Roadmap + opportunities
    - Next: final-synthesizer (report generation)
    - Storage: Task breakdown for top opportunities
  }
`)

npx claude-flow hooks post-edit \
  --memory-key "search/implementation/tasks/breakdown"

# Update state
npx claude-flow memory store \
  --namespace "search/state" \
  --key "current" \
  --value "{
    \"state_id\": \"S4\",
    \"generation\": 4,
    \"phase\": \"implementation_complete\",
    \"completeness\": 0.95,
    \"confidence\": 0.90
  }"

# ============================================================================
# PHASE 5: SYNTHESIS & REPORTING
# ============================================================================

echo "📊 Phase 5: Final Synthesis..."

# Message 16: Generate Comprehensive Report
Task("report-generator", `
# Final Report Synthesis Excellence Framework

## IDENTITY
You are the final synthesis agent, responsible for creating the comprehensive search report.
This is the culmination of 15 agents' work.

## WORKFLOW CONTEXT
**Agent #16/16 (FINAL)**
- Previous: All 15 operational agents ✓
- Mission: Synthesize complete search into actionable report
- Output: Final deliverable for user

## MEMORY RETRIEVAL (Complete Search History)

Retrieve ALL search artifacts:

\`\`\`bash
# Session & State
npx claude-flow memory retrieve --key "search/session/config"
npx claude-flow memory retrieve --key "search/state/current"

# Discovery Phase
npx claude-flow memory retrieve --key "search/discovery/structural/map"
npx claude-flow memory retrieve --key "search/discovery/flows/analysis"
npx claude-flow memory retrieve --key "search/discovery/dependencies/graph"
npx claude-flow memory retrieve --key "search/discovery/critical-paths/analysis"

# Analysis Phase
npx claude-flow memory retrieve --key "search/gaps/quality/catalog"
npx claude-flow memory retrieve --key "search/gaps/performance/catalog"
npx claude-flow memory retrieve --key "search/risks/fmea/analysis"
npx claude-flow memory retrieve --key "search/risks/edge-cases/catalog"
npx claude-flow memory retrieve --key "search/gaps/benchmark/comparison"

# Synthesis Phase
npx claude-flow memory retrieve --key "search/opportunities/generated/list"
npx claude-flow memory retrieve --key "search/opportunities/scored/matrix"
npx claude-flow memory retrieve --key "search/opportunities/pareto/frontier"
npx claude-flow memory retrieve --key "search/opportunities/dependencies/graph"

# Implementation Phase
npx claude-flow memory retrieve --key "search/implementation/roadmap/phases"
npx claude-flow memory retrieve --key "search/implementation/tasks/breakdown"
\`\`\`

## YOUR MISSION
Generate a comprehensive, executive-ready report that synthesizes all findings.

**Success Criteria**:
1. All discoveries clearly presented with visualizations
2. Gaps prioritized with severity and impact
3. Risks assessed with mitigation strategies
4. Opportunities ranked with ROI projections
5. Implementation roadmap ready for execution
6. Report is actionable and decision-ready

## REPORT STRUCTURE

### 1. Executive Summary
- Subject overview
- Key findings (top 5-10)
- Critical issues (high severity)
- Top opportunities (Pareto-optimal)
- Recommended next steps

### 2. Discovery Summary
- Structural architecture (Mermaid diagrams)
- Flow analysis (Mermaid flowcharts)
- Dependency network (Mermaid graphs)
- Critical paths identified

### 3. Gap Analysis
- Quality gaps (consistency, complexity, redundancy)
- Performance gaps (efficiency, effectiveness, scalability)
- Structural gaps (architecture, integration, modularity)
- Prioritization matrix (severity × frequency)

### 4. Risk Assessment
- FMEA table (top 20 highest RPN)
- Vulnerability catalog
- Edge case coverage
- Mitigation strategies

### 5. Opportunity Portfolio
- Generated opportunities (50-200)
- Multi-objective scoring
- Pareto frontier visualization
- Trade-off analysis
- Dependency graph

### 6. Implementation Roadmap
- Phased plan (3-12 months)
- Phase 1: Foundation + Quick wins
- Phase 2: Core improvements
- Phase 3: Advanced features
- Phase 4: Strategic initiatives
- Resource requirements
- Success metrics

### 7. Task Breakdown (Top 10 Opportunities)
- Detailed task specifications
- Acceptance criteria
- Effort estimates
- Dependencies
- Verification protocols

### 8. Meta-Analysis
- Search process insights
- Agent performance metrics
- Strategy effectiveness
- Lessons learned
- Recommendations for future searches

## OUTPUT FORMAT

Create a comprehensive markdown report with:
- Clear hierarchical structure
- Mermaid diagrams for visualizations
- Tables for data presentation
- Prioritized action items
- Executive summary at top

Store the report:
\`\`\`bash
npx claude-flow memory store \
  --namespace "search/output" \
  --key "final_report" \
  --value '{
    "report_path": "/path/to/report.md",
    "executive_summary": "...",
    "key_findings": [...],
    "top_opportunities": [...],
    "next_steps": [...]
  }'
\`\`\`

## QUALITY GATES
- [ ] All 15 agents' work synthesized
- [ ] Visualizations clear and informative
- [ ] Priorities clearly ranked
- [ ] Actionable next steps defined
- [ ] Executive-ready presentation
- [ ] Self-assessment: 100/100

---

Generate the comprehensive final report now.
`)

npx claude-flow hooks session-end --export-metrics

echo "✅ Search complete! Report generated."

# ============================================================================
# PHASE 6: METRICS & LEARNING
# ============================================================================

echo "📈 Collecting metrics..."

npx claude-flow analysis token-usage --breakdown
npx claude-flow analysis bottleneck-detect

# Store session results for future learning
npx claude-flow memory store \
  --namespace "search/learning" \
  --key "session_$SEARCH_ID" \
  --value "{
    \"search_id\": \"$SEARCH_ID\",
    \"subject_type\": \"$SUBJECT_TYPE\",
    \"duration\": \"$(date -u +%s)\",
    \"quality_achieved\": 0.95,
    \"agents_used\": 16,
    \"topology\": \"$TOPOLOGY\",
    \"success\": true
  }"

echo "🎉 Universal search complete!"
```

### 3.2 Simplified Search Patterns

#### Quick Search (1-2 hours)
```bash
# Minimal depth, focus on quick wins

# Message 1: Light Discovery
Task("system-architect", "Light structural mapping + critical path identification")

# Message 2: Gap Hunting (WAIT)
Task("code-reviewer", "Identify top 20 gaps")

# Message 3: Opportunity Generation (WAIT)
Task("innovation-generator", "Generate 30 quick-win opportunities")

# Message 4: Prioritization (WAIT)
Task("decision-analyzer", "Rank by impact/effort ratio")

# Message 5: Report (WAIT)
Task("report-generator", "Executive summary with top 10 actions")
```

#### Standard Search (4-8 hours)
```bash
# Full USACF workflow (as shown in master algorithm)
# 16 agents, 4 phases, comprehensive coverage
```

#### Deep Search (1-3 days)
```bash
# Extended depth with recursive decomposition

# For each major subsystem:
#   1. Run standard search
#   2. Store results in search/subsystems/{id}
#   3. Synthesize across subsystems
#   4. Generate integrated roadmap
```

---

## SECTION 4: OPERATOR LIBRARY (Claude Flow Implementation)

### 4.1 Discovery Operators

#### Operator: StructuralMapping

```bash
# Execution Pattern
Task("system-architect", `
# Structural Mapping Excellence Framework

## IDENTITY
Specialized architectural analyst | Level: {level} | XP: {xp}

## WORKFLOW CONTEXT
Agent #1/15 | Phase: Discovery | Next: flow-analyst, dependency-analyzer, perf-analyzer

## MEMORY RETRIEVAL
\`\`\`bash
npx claude-flow memory retrieve --key "search/session/config"
# Subject: {subject}, Type: {type}
\`\`\`

## MISSION
Map complete structural architecture of {subject}.

**Success Criteria**:
1. All components identified and categorized
2. Hierarchical relationships mapped (3-5 levels deep)
3. Boundaries and interfaces clearly defined
4. Configuration patterns documented
5. Architecture diagrams generated (Mermaid)

## MEMORY STORAGE (For Future Agents)

1. **For flow-analyst** (\`search/discovery/structural/map\`):
   - Component inventory with types
   - Interface definitions
   - Why: Need component boundaries to trace flows

2. **For dependency-analyzer** (\`search/discovery/structural/map\`):
   - Component relationships
   - Internal dependencies
   - Why: Foundation for dependency graph

3. **For perf-analyzer** (\`search/discovery/structural/map\`):
   - Critical components list
   - Performance-sensitive areas
   - Why: Focus performance analysis

## EXECUTION

### Step 1: Identify Components
\`\`\`bash
# Analyze subject structure
# For software: modules, services, databases, APIs
# For business: departments, processes, teams, systems
# For products: features, subsystems, integrations
\`\`\`

### Step 2: Categorize & Classify
- Core components (essential functionality)
- Support components (enablers)
- Integration components (connectors)
- External dependencies (third-party)

### Step 3: Map Hierarchy
\`\`\`mermaid
graph TD
    Root[{subject}]
    Root --> Cat1[Category 1]
    Root --> Cat2[Category 2]
    ...
\`\`\`

### Step 4: Define Boundaries
- What each component owns
- What interfaces it exposes
- What it depends on
- Configuration requirements

### Step 5: Store Results
\`\`\`bash
npx claude-flow memory store \\
  --namespace "search/discovery/structural" \\
  --key "map" \\
  --value '{
    "components": [
      {
        "id": "comp1",
        "name": "...",
        "type": "...",
        "purpose": "...",
        "interfaces": [...],
        "dependencies": [...],
        "critical": true|false
      }
    ],
    "hierarchy": {...},
    "mermaid_diagram": "...",
    "total_components": N,
    "depth": N,
    "completeness": 0.X
  }'

# Verify
npx claude-flow memory retrieve --key "search/discovery/structural/map"
\`\`\`

### Step 6: Update State
\`\`\`bash
npx claude-flow memory store \\
  --namespace "search/state" \\
  --key "current" \\
  --value '{
    "state_id": "S0.1",
    "phase": "discovery",
    "operator_applied": "StructuralMapping",
    "completeness": 0.15,
    "confidence": 0.7
  }'
\`\`\`

## XP REWARDS
- **Component Discovery**: +10 XP per component (Max: 500)
- **Hierarchy Depth**: +50 XP per level beyond 3
- **Interface Clarity**: +20 XP per well-defined interface
- **Diagram Quality**: +100 XP for comprehensive Mermaid diagram
- **Future Agent Enablement**: +200 XP if all 3 agents report complete context

## SELF-ASSESSMENT
Rate: 1-100 vs mission success criteria
- Target: 100/100
- If < 100: Iterate until perfect

## VERIFICATION
Spawn reviewer subagent to validate:
- All components identified?
- Hierarchy complete?
- Interfaces clear?
- Next agents have context?
`)
```

#### Operator: FlowAnalysis

```bash
Task("code-analyzer", `
# Flow Analysis Excellence Framework

## WORKFLOW CONTEXT
Agent #2/15 | Previous: system-architect ✓ | Next: dependency-analyzer, gap-hunter

## MEMORY RETRIEVAL
\`\`\`bash
npx claude-flow memory retrieve --key "search/discovery/structural/map"
# Components, interfaces, boundaries
\`\`\`

## MISSION
Trace all major flows through {subject}.

**Flow Types by Domain**:
- Software: Data flow, control flow, event flow, error flow
- Business: Information flow, approval flow, value flow, customer journey
- Manufacturing: Material flow, energy flow, quality flow
- Financial: Money flow, transaction flow, settlement flow

**Success Criteria**:
1. All major flows identified and traced
2. Transformation points documented
3. Storage/accumulation points mapped
4. Validation checkpoints identified
5. Error handling paths traced
6. Flow diagrams generated (Mermaid)

## MEMORY STORAGE

1. **For dependency-analyzer** (\`search/discovery/flows/analysis\`):
   - Flow dependencies between components
   - Data contracts and schemas
   - Why: Understand component coupling

2. **For gap-hunter** (\`search/discovery/flows/analysis\`):
   - Bottleneck locations
   - Missing validations
   - Error handling gaps
   - Why: Identify flow-based gaps

## EXECUTION

### Identify Flow Types
- Primary flows (main operational paths)
- Secondary flows (support/admin paths)
- Error flows (exception handling)
- Monitoring flows (observability)

### Trace Each Flow
For each flow type:
1. Input sources
2. Processing steps (transformations)
3. Decision points (branching)
4. Storage points (persistence)
5. Validation points (quality gates)
6. Output destinations
7. Error handlers (failure paths)

### Generate Flow Diagrams
\`\`\`mermaid
flowchart LR
    Input[Source] --> Validate{Valid?}
    Validate -->|Yes| Process[Transform]
    Validate -->|No| Error[Error Handler]
    Process --> Store[(Storage)]
    Store --> Output[Destination]
\`\`\`

### Store Results
\`\`\`bash
npx claude-flow memory store \\
  --namespace "search/discovery/flows" \\
  --key "analysis" \\
  --value '{
    "flows": [
      {
        "flow_id": "f1",
        "name": "...",
        "type": "...",
        "path": [...],
        "transformations": [...],
        "bottlenecks": [...],
        "validations": [...],
        "error_handling": [...]
      }
    ],
    "mermaid_diagrams": [...],
    "bottleneck_count": N,
    "completeness": 0.X
  }'
\`\`\`

## XP REWARDS
- **Flow Traced**: +25 XP per complete flow
- **Bottleneck Found**: +50 XP per bottleneck
- **Validation Gap**: +30 XP per missing validation
- **Error Path Documented**: +40 XP per error handler
- **Diagram Excellence**: +150 XP for clear Mermaid diagrams

Execute flow analysis now.
`)
```

#### Operator: DependencyTracking

```bash
Task("dependency-analyzer", `
# Dependency Analysis Excellence Framework

## WORKFLOW CONTEXT
Agent #3/15 | Previous: system-architect + code-analyzer ✓ | Next: perf-analyzer, gap-hunter, risk-analyst

## MEMORY RETRIEVAL
\`\`\`bash
npx claude-flow memory retrieve --key "search/discovery/structural/map"
npx claude-flow memory retrieve --key "search/discovery/flows/analysis"
\`\`\`

## MISSION
Build complete dependency network for {subject}.

**Success Criteria**:
1. All dependencies identified (internal + external)
2. Dependency types classified
3. Circular dependencies detected
4. Critical paths identified
5. Unused dependencies found
6. Dependency graph generated (Mermaid)

**Dependency Types**:
- Required: Must have for operation
- Optional: Enhances but not critical
- Development: Build-time only
- Circular: Mutual dependencies (problematic)
- Transitive: Indirect dependencies

## MEMORY STORAGE

1. **For perf-analyzer** (\`search/discovery/dependencies/graph\`):
   - Critical dependency paths
   - Bottleneck dependencies
   - Why: Performance analysis focus areas

2. **For gap-hunter** (\`search/discovery/dependencies/graph\`):
   - Circular dependencies (gaps)
   - Unused dependencies (waste)
   - Missing dependencies (risks)
   - Why: Structural gap identification

3. **For risk-analyst** (\`search/discovery/dependencies/graph\`):
   - External dependencies (vulnerabilities)
   - Single points of failure
   - Why: Risk assessment inputs

## EXECUTION

### Build Dependency Graph
\`\`\`bash
# For each component from structural map:
#   1. Identify direct dependencies
#   2. Classify dependency type
#   3. Assess health (version, stability, risk)
#   4. Check for circularity
#   5. Find transitive dependencies
\`\`\`

### Analyze Patterns
- Highly coupled clusters (tight dependencies)
- Isolated components (no dependencies)
- Hub components (many depend on them)
- Leaf components (depend on many, nobody depends on them)

### Identify Issues
- Circular dependencies (A → B → A)
- Unused dependencies (declared but never used)
- Missing dependencies (used but not declared)
- Outdated dependencies (old versions)
- Risky dependencies (unmaintained, vulnerable)

### Generate Dependency Diagram
\`\`\`mermaid
graph LR
    A[Core A] -->|required| B[Core B]
    A -->|required| C[Core C]
    B <-->|circular| C
    B -->|optional| D[External 1]
    
    style A fill:#90EE90
    style B fill:#FFB6C1
    style D fill:#FFD700
\`\`\`

### Store Results
\`\`\`bash
npx claude-flow memory store \\
  --namespace "search/discovery/dependencies" \\
  --key "graph" \\
  --value '{
    "dependencies": [
      {
        "from": "comp1",
        "to": "comp2",
        "type": "required|optional|circular",
        "health": "healthy|outdated|vulnerable",
        "critical": true|false
      }
    ],
    "circular_dependencies": [...],
    "unused_dependencies": [...],
    "critical_paths": [...],
    "mermaid_diagram": "...",
    "total_deps": N,
    "circular_count": N,
    "unused_count": N
  }'
\`\`\`

## XP REWARDS
- **Dependency Mapped**: +15 XP per dependency
- **Circular Detected**: +100 XP per circular dependency
- **Unused Found**: +50 XP per unused dependency
- **Critical Path ID**: +75 XP per critical path
- **Health Assessment**: +25 XP per dependency health check

Execute dependency analysis now.
`)
```

### 4.2 Analysis Operators

#### Operator: QualityGapAnalysis

```bash
Task("code-reviewer", `
# Quality Gap Analysis Excellence Framework

## WORKFLOW CONTEXT
Agent #5/15 | Phase: Analysis | Previous: All discovery ✓ | Next: perf-analyzer, risk-analyst, benchmark

## MEMORY RETRIEVAL
\`\`\`bash
npx claude-flow memory retrieve --key "search/discovery/structural/map"
npx claude-flow memory retrieve --key "search/discovery/flows/analysis"
npx claude-flow memory retrieve --key "search/discovery/dependencies/graph"
\`\`\`

## MISSION
Identify all quality gaps in {subject}.

**Success Criteria**:
1. 30-50 quality gaps identified
2. Each gap has severity score (1-10)
3. Impact assessment completed
4. Frequency/occurrence estimated
5. Priority scores calculated (severity × frequency)
6. Root causes documented

**Quality Dimensions**:
- **Consistency**: Standards, naming, patterns, style
- **Complexity**: Unnecessary complication, cognitive load
- **Redundancy**: Beneficial vs. wasteful duplication
- **Maintainability**: Ease of modification, documentation
- **Testability**: Test coverage, test quality

## MEMORY STORAGE

1. **For perf-analyzer** (\`search/gaps/quality/catalog\`):
   - Complexity gaps (may cause perf issues)
   - Redundancy gaps (may cause inefficiency)
   - Why: Cross-reference with performance analysis

2. **For opportunity-gen** (\`search/gaps/quality/catalog\`):
   - All quality gaps (basis for improvement opportunities)
   - Priority scores (focus high-value improvements)
   - Why: Generate targeted solutions

## EXECUTION

### Assess Consistency
- Naming conventions (consistent?)
- Code/process style (uniform?)
- Pattern usage (standardized?)
- Documentation (complete? up-to-date?)
- Standards compliance (following best practices?)

**Gaps**: Inconsistencies, deviations, conflicts

### Evaluate Complexity
- Cyclomatic complexity (too high?)
- Nesting depth (too deep?)
- Function/module size (too large?)
- Abstraction levels (appropriate?)
- Cognitive load (easy to understand?)

**Gaps**: Over-complexity, unnecessary complication

### Identify Redundancy
- Beneficial redundancy (backups, fault tolerance, caching)
- Wasteful redundancy (duplicated code, repeated logic, copy-paste)
- Consolidation opportunities

**Gaps**: Wasteful duplication, missing beneficial redundancy

### Score Each Gap
For each identified gap:
\`\`\`python
severity = 1-10  # Impact if not fixed
frequency = 1-10  # How often encountered
priority = severity × frequency
\`\`\`

### Generate Gap Register
\`\`\`bash
npx claude-flow memory store \\
  --namespace "search/gaps/quality" \\
  --key "catalog" \\
  --value '{
    "gaps": [
      {
        "gap_id": "QG001",
        "category": "consistency|complexity|redundancy|maintainability|testability",
        "issue": "Description of the gap",
        "severity": 1-10,
        "frequency": 1-10,
        "priority": severity × frequency,
        "impact": "What happens if not fixed",
        "examples": [...],
        "root_cause": "Why this exists"
      }
    ],
    "summary": {
      "total_gaps": N,
      "by_category": {...},
      "avg_priority": X,
      "high_priority_count": N
    }
  }'
\`\`\`

## XP REWARDS
- **Gap Identified**: +20 XP per gap
- **High Priority Gap**: +50 XP bonus (priority > 70)
- **Root Cause Analysis**: +30 XP per complete root cause
- **Comprehensive Coverage**: +200 XP if 30+ gaps found
- **Impact Assessment**: +25 XP per detailed impact

Execute quality gap analysis now.
`)
```

#### Operator: PerformanceGapAnalysis

```bash
Task("perf-analyzer", `
# Performance Gap Analysis Excellence Framework

## WORKFLOW CONTEXT
Agent #6/15 | Previous: Quality gaps ✓ | Next: risk-analyst, opportunity-gen

## MEMORY RETRIEVAL
\`\`\`bash
npx claude-flow memory retrieve --key "search/discovery/structural/map"
npx claude-flow memory retrieve --key "search/discovery/flows/analysis"
npx claude-flow memory retrieve --key "search/discovery/critical-paths/analysis"
npx claude-flow memory retrieve --key "search/gaps/quality/catalog"
\`\`\`

## MISSION
Identify all performance gaps in {subject}.

**Success Criteria**:
1. Efficiency gaps identified (resource utilization)
2. Effectiveness gaps identified (goal achievement)
3. Scalability gaps identified (growth capacity)
4. Baseline metrics documented
5. Target metrics defined
6. Gap severity scored

## MEMORY STORAGE

1. **For risk-analyst** (\`search/gaps/performance/catalog\`):
   - Performance bottlenecks (potential failure points)
   - Scalability limits (risk areas)
   - Why: Risk assessment context

2. **For opportunity-gen** (\`search/gaps/performance/catalog\`):
   - All performance gaps (optimization targets)
   - High-impact gaps (prioritization)
   - Why: Performance improvement opportunities

## EXECUTION

### Assess Efficiency
- **Resource Utilization**: CPU, memory, disk, network
- **Waste Identification**: Unused resources, inefficient algorithms
- **Bottleneck Analysis**: Slowest components
- **Throughput**: Transactions/queries per second

**Metrics**:
- Current efficiency: X%
- Target efficiency: Y%
- Gap: (Y - X) / Y

### Evaluate Effectiveness
- **Success Rates**: % of operations succeeding
- **Quality Levels**: Output quality metrics
- **Objective Achievement**: Meeting business goals?
- **User Satisfaction**: Meeting user needs?

**Metrics**:
- Current effectiveness: X%
- Expected effectiveness: Y%
- Gap: (Y - X) / Y

### Analyze Scalability
- **Growth Capacity**: How much can it handle?
- **Breaking Points**: Where does it fail?
- **Constraint Identification**: What limits growth?
- **Expansion Costs**: Cost to scale

**Metrics**:
- Current capacity: X units
- Required capacity: Y units
- Gap: (Y - X) / Y

### Store Results
\`\`\`bash
npx claude-flow memory store \\
  --namespace "search/gaps/performance" \\
  --key "catalog" \\
  --value '{
    "gaps": [
      {
        "gap_id": "PG001",
        "category": "efficiency|effectiveness|scalability",
        "baseline": "Current performance",
        "target": "Desired performance",
        "gap_size": "Difference",
        "severity": 1-10,
        "impact": "Business impact",
        "bottleneck_components": [...]
      }
    ],
    "summary": {
      "total_gaps": N,
      "efficiency_gaps": N,
      "effectiveness_gaps": N,
      "scalability_gaps": N
    }
  }'
\`\`\`

## XP REWARDS
- **Performance Gap**: +30 XP per gap
- **Bottleneck Identified**: +75 XP per bottleneck
- **Metrics Documented**: +20 XP per baseline/target pair
- **Scalability Analysis**: +100 XP for thorough scalability assessment

Execute performance gap analysis now.
`)
```

#### Operator: RiskAnalysis (FMEA)

```bash
Task("security-auditor", `
# Risk Analysis (FMEA) Excellence Framework

## WORKFLOW CONTEXT
Agent #7/15 | Previous: Quality + Performance gaps ✓ | Next: edge-case-analyzer, opportunity-gen

## MEMORY RETRIEVAL
\`\`\`bash
npx claude-flow memory retrieve --key "search/discovery/structural/map"
npx claude-flow memory retrieve --key "search/discovery/flows/analysis"
npx claude-flow memory retrieve --key "search/discovery/dependencies/graph"
npx claude-flow memory retrieve --key "search/discovery/critical-paths/analysis"
\`\`\`

## MISSION
Conduct Failure Mode and Effects Analysis (FMEA) for {subject}.

**Success Criteria**:
1. All critical components analyzed
2. Failure modes identified for each
3. RPN scores calculated (Severity × Occurrence × Detection)
4. High-risk items (RPN > 100) flagged
5. Current controls documented
6. Mitigation recommendations provided

## MEMORY STORAGE

1. **For edge-case-analyzer** (\`search/risks/fmea/analysis\`):
   - Failure modes (basis for edge cases)
   - Vulnerable components
   - Why: Identify edge cases around failures

2. **For opportunity-gen** (\`search/risks/fmea/analysis\`):
   - High RPN items (urgent improvements)
   - Control gaps (mitigation opportunities)
   - Why: Risk reduction opportunities

## EXECUTION

### FMEA Process
For each critical component:

1. **Identify Failure Modes**
   - How can this component fail?
   - What can go wrong?

2. **Assess Severity (1-10)**
   - 1-3: Minor inconvenience
   - 4-6: Moderate impact
   - 7-9: Severe impact
   - 10: Catastrophic failure

3. **Assess Occurrence (1-10)**
   - 1-3: Rare
   - 4-6: Occasional
   - 7-9: Frequent
   - 10: Almost certain

4. **Assess Detection (1-10)**
   - 1-3: Almost certain to detect
   - 4-6: Moderate detection
   - 7-9: Difficult to detect
   - 10: Cannot detect

5. **Calculate RPN**
   - RPN = Severity × Occurrence × Detection
   - Priority: RPN > 100 = High, RPN > 50 = Medium

6. **Document Current Controls**
   - What prevents this failure?
   - What detects this failure?

7. **Recommend Actions**
   - How to reduce Severity?
   - How to reduce Occurrence?
   - How to improve Detection?

### Generate FMEA Table
| Component | Failure Mode | Effect | Sev | Cause | Occur | Controls | Det | RPN | Action |
|-----------|--------------|--------|-----|-------|-------|----------|-----|-----|--------|
| Database | Crash | Data loss | 9 | Memory leak | 5 | Daily backup | 3 | 135 | Add monitoring |
| API | Timeout | Request fail | 7 | Network issue | 6 | Retry logic | 4 | 168 | Circuit breaker |

### Store Results
\`\`\`bash
npx claude-flow memory store \\
  --namespace "search/risks/fmea" \\
  --key "analysis" \\
  --value '{
    "fmea_entries": [
      {
        "component_id": "...",
        "failure_mode": "...",
        "effect": "...",
        "severity": 1-10,
        "cause": "...",
        "occurrence": 1-10,
        "current_controls": [...],
        "detection": 1-10,
        "rpn": severity × occurrence × detection,
        "recommended_actions": [...]
      }
    ],
    "summary": {
      "total_entries": N,
      "high_rpn_count": N,
      "avg_rpn": X,
      "critical_components": [...]
    }
  }'
\`\`\`

## XP REWARDS
- **Failure Mode Identified**: +40 XP per mode
- **High RPN Detection**: +100 XP per RPN > 100
- **Control Documentation**: +30 XP per control
- **Mitigation Recommendation**: +50 XP per actionable recommendation
- **Comprehensive FMEA**: +250 XP if 20+ entries

Execute FMEA analysis now.
`)
```

### 4.3 Synthesis Operators

#### Operator: OpportunityGeneration

```bash
Task("innovation-generator", `
# Opportunity Generation Excellence Framework

## WORKFLOW CONTEXT
Agent #10/15 | Phase: Synthesis | Previous: All gaps + risks ✓ | Next: scorer, optimizer, dependency-planner

## MEMORY RETRIEVAL
\`\`\`bash
npx claude-flow memory retrieve --key "search/gaps/quality/catalog"
npx claude-flow memory retrieve --key "search/gaps/performance/catalog"
npx claude-flow memory retrieve --key "search/risks/fmea/analysis"
npx claude-flow memory retrieve --key "search/risks/edge-cases/catalog"
npx claude-flow memory retrieve --key "search/gaps/benchmark/comparison"
\`\`\`

## MISSION
Generate 50-200 improvement opportunities addressing all identified gaps and risks.

**Success Criteria**:
1. 50-200 opportunities generated
2. Each addresses specific gap/risk
3. Multiple solution approaches per gap (3-5)
4. Innovation opportunities included
5. Dependencies documented
6. Effort estimates provided

**Opportunity Types**:
- **Direct Fixes**: Eliminate the gap completely
- **Workarounds**: Mitigate impact without full fix
- **Alternatives**: Different approach to same goal
- **Innovations**: Breakthrough solutions, new capabilities

## MEMORY STORAGE

1. **For scorer** (\`search/opportunities/generated/list\`):
   - All opportunities with descriptions
   - Initial categorization
   - Why: Needs complete list for scoring

2. **For optimizer** (\`search/opportunities/generated/list\`):
   - Opportunities with relationships
   - Why: Portfolio optimization needs full set

3. **For dependency-planner** (\`search/opportunities/generated/list\`):
   - Opportunity dependencies
   - Why: Sequencing and planning

## EXECUTION

### Generate Opportunities from Gaps
For each gap:
1. **Direct Fix**: How to eliminate this gap?
2. **Quick Win**: Fast partial improvement?
3. **Strategic Solution**: Comprehensive long-term fix?
4. **Alternative Approach**: Different way to avoid the gap?
5. **Innovation**: Could we do something better entirely?

### Generate Opportunities from Risks
For each high RPN risk:
1. **Severity Reduction**: How to reduce impact?
2. **Occurrence Reduction**: How to prevent?
3. **Detection Improvement**: How to detect earlier?
4. **Resilience**: How to recover faster?

### Generate Opportunities from Benchmarks
For each benchmark gap:
1. **Best Practice Adoption**: Implement industry standard?
2. **Competitive Parity**: Match competitor capability?
3. **Innovation**: Leapfrog competition?

### Diversify Solution Space
- Low-hanging fruit (quick wins)
- Medium-term improvements
- Strategic initiatives
- Moonshot ideas

### Document Each Opportunity
\`\`\`json
{
  "opp_id": "OP001",
  "title": "Implement Redis caching layer",
  "description": "Add caching for frequently accessed data",
  "addresses": ["PG001", "PG003"],  // Gap IDs
  "type": "direct_fix|workaround|alternative|innovation",
  "expected_impact": {
    "value": "50% response time improvement",
    "risk_reduction": "Reduces database load",
    "quality_improvement": "Better user experience"
  },
  "required_effort": {
    "time": "3 weeks",
    "cost": "$10k",
    "complexity": "medium"
  },
  "dependencies": ["OP000"],  // Prerequisite opportunities
  "enables": ["OP010", "OP015"],  // Unlocks these opportunities
  "risks": ["May increase memory usage"]
}
\`\`\`

### Store Results
\`\`\`bash
npx claude-flow memory store \\
  --namespace "search/opportunities/generated" \\
  --key "list" \\
  --value '{
    "opportunities": [...],  // 50-200 opportunities
    "summary": {
      "total_count": N,
      "by_type": {...},
      "by_gap_addressed": {...},
      "quick_wins": N,
      "strategic_initiatives": N
    }
  }'
\`\`\`

## XP REWARDS
- **Opportunity Generated**: +15 XP per opportunity
- **Innovation Opportunity**: +75 XP per innovative idea
- **Comprehensive Coverage**: +500 XP if 100+ opportunities
- **Solution Diversity**: +100 XP for multiple approaches per gap
- **Future Agent Enablement**: +300 XP if scorer has complete context

Execute opportunity generation now.
`)
```

#### Operator: MultiObjectiveScoring

```bash
Task("decision-analyzer", `
# Multi-Objective Scoring Excellence Framework

## WORKFLOW CONTEXT
Agent #11/15 | Previous: Opportunity generation ✓ | Next: optimizer, roadmap-planner

## MEMORY RETRIEVAL
\`\`\`bash
npx claude-flow memory retrieve --key "search/opportunities/generated/list"
npx claude-flow memory retrieve --key "search/gaps/quality/catalog"
npx claude-flow memory retrieve --key "search/gaps/performance/catalog"
\`\`\`

## MISSION
Score all opportunities across multiple dimensions.

**Success Criteria**:
1. All opportunities scored (0-10 per dimension)
2. 7 dimensions: Value, Quality, Risk Reduction, Feasibility, Time, Resources, Strategic Fit
3. Composite priority scores calculated
4. Risk-adjusted scores computed
5. Top 20% identified

## MEMORY STORAGE

1. **For optimizer** (\`search/opportunities/scored/matrix\`):
   - Complete scoring matrix
   - Multi-dimensional vectors
   - Why: Pareto optimization needs all dimensions

2. **For roadmap-planner** (\`search/opportunities/scored/matrix\`):
   - Priority scores
   - Feasibility assessments
   - Why: Roadmap sequencing

## EXECUTION

### Scoring Framework

For each opportunity, score 0-10 on:

**Impact Dimensions**:
1. **Value Impact** (0-10)
   - Revenue increase / Cost savings
   - Efficiency gains
   - Quantifiable benefits

2. **Quality Impact** (0-10)
   - Reliability improvement
   - Performance improvement
   - Maintainability improvement

3. **Risk Reduction** (0-10)
   - RPN reduction (from FMEA)
   - Vulnerability mitigation
   - Resilience improvement

4. **Strategic Impact** (0-10)
   - Goal alignment
   - Competitive advantage
   - Future optionality

**Feasibility Dimensions**:
5. **Technical Feasibility** (0-10)
   - Complexity (inverted)
   - Known vs. unknown technology
   - Technical risks

6. **Resource Availability** (0-10)
   - Skills available
   - Tools available
   - Budget available

7. **Time Required** (0-10, inverted)
   - Fast implementation = 10
   - Slow implementation = 1

### Calculate Composite Scores

\`\`\`python
# Weighted composite scores
impact_score = (
  value * 0.3 +
  quality * 0.25 +
  risk_reduction * 0.25 +
  strategic * 0.2
) / 4

feasibility_score = (
  technical * 0.4 +
  resources * 0.3 +
  time * 0.3
) / 3

# Priority score
priority = (
  impact_score * 0.6 +
  feasibility_score * 0.4
)

# Risk-adjusted priority
implementation_risk = 10 - technical_feasibility
risk_adjusted_priority = priority / (1 + implementation_risk/10)
\`\`\`

### Generate Scoring Matrix

\`\`\`bash
npx claude-flow memory store \\
  --namespace "search/opportunities/scored" \\
  --key "matrix" \\
  --value '{
    "scored_opportunities": [
      {
        "opp_id": "OP001",
        "scores": {
          "value_impact": 8,
          "quality_impact": 7,
          "risk_reduction": 6,
          "strategic_impact": 5,
          "technical_feasibility": 8,
          "resource_availability": 9,
          "time_required": 7
        },
        "composite": {
          "impact_score": 6.5,
          "feasibility_score": 8.0,
          "priority": 7.1,
          "risk_adjusted_priority": 6.4
        },
        "rank": 5,
        "percentile": 95
      }
    ],
    "summary": {
      "total_scored": N,
      "top_20_percent": [...],
      "high_impact_high_feasibility": [...],
      "quick_wins": [...]  // High impact, high feasibility, fast
    }
  }'
\`\`\`

## XP REWARDS
- **Opportunity Scored**: +10 XP per scored opportunity
- **Comprehensive Scoring**: +300 XP if all opportunities scored
- **Top Opportunities ID**: +50 XP per top 20% opportunity
- **Quick Wins ID**: +100 XP for identifying quick wins

Execute multi-objective scoring now.
`)
```

### 4.4 Implementation Operators

#### Operator: RoadmapPlanning

```bash
Task("project-manager", `
# Roadmap Planning Excellence Framework

## WORKFLOW CONTEXT
Agent #14/15 | Previous: Pareto optimization + dependencies ✓ | Next: task-decomposer, final-synthesizer

## MEMORY RETRIEVAL
\`\`\`bash
npx claude-flow memory retrieve --key "search/opportunities/pareto/frontier"
npx claude-flow memory retrieve --key "search/opportunities/dependencies/graph"
npx claude-flow memory retrieve --key "search/opportunities/scored/matrix"
\`\`\`

## MISSION
Create phased implementation roadmap (3-12 months).

**Success Criteria**:
1. 3-5 phases defined
2. Opportunities assigned to phases
3. Dependencies respected in sequencing
4. Resource requirements per phase
5. Success metrics defined
6. Risk mitigation plans included
7. Gantt chart generated (Mermaid)

## MEMORY STORAGE

1. **For task-decomposer** (\`search/implementation/roadmap/phases\`):
   - Phase assignments
   - Priority opportunities per phase
   - Why: Knows which opportunities to break down first

2. **For final-synthesizer** (\`search/implementation/roadmap/phases\`):
   - Complete roadmap
   - Timeline
   - Why: Executive summary and final report

## EXECUTION

### Phase Structure

**Phase 1 (Weeks 1-4): Foundation + Quick Wins**
- Prerequisites for future work
- High-impact, low-effort wins
- Risk mitigation for critical issues
- Establish monitoring/observability

**Phase 2 (Months 2-3): Core Improvements**
- Medium-effort, high-value opportunities
- Address major gaps
- Build on Phase 1 foundation

**Phase 3 (Months 4-6): Advanced Features**
- Complex improvements
- Strategic initiatives
- Require Phase 1 + 2 complete

**Phase 4 (Months 7-12): Strategic Transformation**
- Long-term improvements
- Innovation opportunities
- Competitive differentiation

### Assign Opportunities to Phases

\`\`\`python
for opp in pareto_frontier:
  # Check dependencies
  if all_deps_in_earlier_phases(opp):
    # Check effort
    if opp.effort == "low" and opp.impact == "high":
      assign_to_phase(1)  # Quick win
    elif opp.is_prerequisite():
      assign_to_phase(1)  # Foundation
    elif opp.effort == "medium":
      assign_to_phase(2)  # Core
    elif opp.effort == "high":
      assign_to_phase(3 or 4)  # Advanced/Strategic
\`\`\`

### Generate Gantt Chart

\`\`\`mermaid
gantt
    title Implementation Roadmap
    dateFormat YYYY-MM-DD
    section Phase 1: Foundation
    OP001 Quick Win A :p1_1, 2025-01-01, 2w
    OP002 Quick Win B :p1_2, 2025-01-01, 2w
    OP003 Foundation C :p1_3, 2025-01-15, 2w
    
    section Phase 2: Core
    OP010 Core Improvement A :p2_1, after p1_3, 4w
    OP011 Core Improvement B :p2_2, after p1_3, 4w
    
    section Phase 3: Advanced
    OP020 Advanced Feature :p3_1, after p2_1 p2_2, 6w
\`\`\`

### Store Results

\`\`\`bash
npx claude-flow memory store \\
  --namespace "search/implementation/roadmap" \\
  --key "phases" \\
  --value '{
    "phases": [
      {
        "phase_id": "P1",
        "name": "Foundation + Quick Wins",
        "duration": "4 weeks",
        "start_date": "2025-01-01",
        "opportunities": ["OP001", "OP002", "OP003"],
        "objectives": [...],
        "success_metrics": [...],
        "resources_required": {
          "team_size": 3,
          "budget": "$50k",
          "tools": [...]
        },
        "risks": [...],
        "mitigation": [...]
      }
    ],
    "gantt_diagram": "...",
    "total_duration": "12 months",
    "total_budget": "$500k"
  }'
\`\`\`

## XP REWARDS
- **Phase Defined**: +100 XP per well-structured phase
- **Dependency Respect**: +200 XP if all dependencies honored
- **Resource Planning**: +150 XP for detailed resource plans
- **Risk Mitigation**: +100 XP per risk mitigation strategy
- **Gantt Excellence**: +250 XP for clear timeline visualization

Execute roadmap planning now.
`)
```

---

## SECTION 5: STOPPING CRITERIA & QUALITY GATES

### 5.1 Quality Gates (Hooks)

```bash
# Pre-Task Hook: Verify prerequisites
npx claude-flow hooks pre-task --description "{operator_name}"

# Checks:
# - All required memories available?
# - State consistency valid?
# - Resources available?

# Post-Edit Hook: Verify operator output
npx claude-flow hooks post-edit \
  --file "{result_path}" \
  --memory-key "search/{namespace}/{key}"

# Checks:
# - Success criteria met?
# - Memory properly stored?
# - Next agents have context?
# - Quality score > 90?

# Session-End Hook: Final validation
npx claude-flow hooks session-end --export-metrics

# Checks:
# - All phases complete?
# - Target quality achieved?
# - Report generated?
# - Metrics collected?
```

### 5.2 Stopping Criteria

```bash
#!/bin/bash
# check_stopping_criteria.sh

# Retrieve current state
STATE=$(npx claude-flow memory retrieve --key "search/state/current")
COMPLETENESS=$(echo $STATE | jq -r '.completeness')
CONFIDENCE=$(echo $STATE | jq -r '.confidence')

# Retrieve config
CONFIG=$(npx claude-flow memory retrieve --key "search/session/config")
TARGET_COMPLETENESS=$(echo $CONFIG | jq -r '.constraints.quality_threshold')
TIME_LIMIT=$(echo $CONFIG | jq -r '.constraints.time_limit')

# Check quality threshold
if (( $(echo "$COMPLETENESS >= $TARGET_COMPLETENESS" | bc -l) )) && \
   (( $(echo "$CONFIDENCE >= 0.85" | bc -l) )); then
  echo "✅ STOP: Quality targets achieved"
  exit 0
fi

# Check time exhaustion
START_TIME=$(echo $CONFIG | jq -r '.started_at')
ELAPSED=$(( $(date +%s) - $(date -d "$START_TIME" +%s) ))
if [ $ELAPSED -gt $(convert_to_seconds $TIME_LIMIT) ]; then
  echo "⏰ STOP: Time budget exhausted"
  exit 0
fi

# Check phase completion
PHASE=$(echo $STATE | jq -r '.phase')
if [ "$PHASE" == "implementation_complete" ]; then
  echo "✅ STOP: All phases complete"
  exit 0
fi

echo "âš¡ CONTINUE: Search in progress"
exit 1
```

---

## SECTION 6: USAGE PATTERNS

### 6.1 Quick Start (Any Subject)

```bash
# 1. Initialize
npx claude-flow@alpha init
npx claude-flow@alpha agent memory init

# 2. Define search
SUBJECT="Your subject here"
TYPE="software|business|process|product"
OBJECTIVES='["obj1", "obj2", "obj3"]'

# 3. Run master algorithm
./universal_search.sh "$SUBJECT" "$TYPE" "$OBJECTIVES" "2h"

# 4. Get results
npx claude-flow memory retrieve --key "search/output/final_report"
```

### 6.2 Domain-Specific Examples

#### Software System Analysis
```bash
./universal_search.sh \
  "E-commerce web application" \
  "software" \
  '["performance", "security", "scalability"]' \
  "4h"

# Agents: 16
# Phases: 4
# Expected Outputs:
# - Structural map (components, APIs, databases)
# - 40-60 gaps (code quality, performance, security)
# - 80-150 opportunities
# - 6-month roadmap
```

#### Business Process Analysis
```bash
./universal_search.sh \
  "Customer onboarding process" \
  "business" \
  '["reduce_time", "improve_satisfaction", "compliance"]' \
  "6h"

# Agents: 16
# Topology: mesh (complex stakeholder interactions)
# Expected Outputs:
# - Process flow maps
# - 30-50 bottlenecks
# - 60-100 improvements
# - Process redesign roadmap
```

#### Product Feature Analysis
```bash
./universal_search.sh \
  "Mobile app feature set" \
  "product" \
  '["ux_improvement", "competitive_parity", "innovation"]' \
  "8h"

# Agents: 16
# Topology: hierarchical
# Expected Outputs:
# - Feature inventory
# - 25-40 UX friction points
# - 100-200 feature ideas
# - 12-month feature roadmap
```

### 6.3 Recursive Deep Search

For complex subjects with subsystems:

```bash
#!/bin/bash
# deep_search.sh - Recursive subsystem analysis

SUBJECT="$1"
SUBSYSTEMS=("subsystem1" "subsystem2" "subsystem3")

# Phase 1: Analyze each subsystem
for SUB in "${SUBSYSTEMS[@]}"; do
  echo "Analyzing $SUB..."
  ./universal_search.sh "$SUB" "software" '["comprehensive"]' "2h"
  
  # Store subsystem results
  npx claude-flow memory store \
    --namespace "search/subsystems/$SUB" \
    --key "results" \
    --value "$(npx claude-flow memory retrieve --key 'search/output/final_report')"
done

# Phase 2: Cross-subsystem analysis
Task("system-architect", `
# Cross-Subsystem Integration Analysis

## MEMORY RETRIEVAL
\`\`\`bash
npx claude-flow memory retrieve --key "search/subsystems/subsystem1/results"
npx claude-flow memory retrieve --key "search/subsystems/subsystem2/results"
npx claude-flow memory retrieve --key "search/subsystems/subsystem3/results"
\`\`\`

## MISSION
Analyze integration points, identify cross-cutting concerns, generate integrated roadmap.

## EXECUTION
1. Map integration interfaces
2. Identify cross-subsystem dependencies
3. Find integration gaps
4. Generate unified roadmap
`)

# Phase 3: Synthesize
Task("report-generator", "Generate integrated final report")
```

---

## SECTION 7: PERFORMANCE OPTIMIZATION

### 7.1 Speed Optimizations

```bash
# Use booster for large-scale edits
npx claude-flow agent booster edit  # 352x faster

# Batch memory operations
# Instead of 10 separate stores:
for i in {1..10}; do
  npx claude-flow memory store --key "gap_$i" --value "{...}"
done

# Do single batch store:
npx claude-flow memory store \
  --namespace "search/gaps/quality" \
  --key "all" \
  --value '{
    "gap_1": {...},
    "gap_2": {...},
    ...
  }'

# Parallel read-only analysis (0.1% use case)
# ONLY for independent, read-only operations
Task("code-analyzer", "Analyze module A")  # Read-only
Task("perf-analyzer", "Analyze metrics")   # Read-only, same message
# Both read structural map, no modifications
```

### 7.2 Quality Optimizations

```bash
# Always use ReasoningBank (88% vs 60% success)
npx claude-flow agent memory init
npx claude-flow agent memory status

# Always use forward-looking prompts
# Tell agents about future needs
# Store what's needed, not just what you found

# Always use hooks for verification
npx claude-flow hooks pre-task
npx claude-flow hooks post-edit
npx claude-flow hooks session-end

# Always include self-assessment
# Target: 100/100 vs user intent
# Iterate until perfect
```

---

## SECTION 8: TRUTH & QUALITY PROTOCOLS

### 8.1 Radical Honesty (INTJ + Type 8)

Every subagent MUST:
- ✅ State only verified facts
- ✅ Challenge incorrect assumptions
- ✅ Report limitations honestly
- ✅ "That won't work because..." over sugar-coating
- ❌ No lies, simulations, or illusions
- ❌ No misleading about functionality

### 8.2 Reviewer Subagent Pattern

After complex operators, spawn reviewer:

```bash
Task("meta-learning-agent", `
# Reviewer: Validate {Operator} Output

## MISSION
Verify {operator} output meets all success criteria.

## CHECKS
1. Completeness: All required elements present?
2. Accuracy: Claims verified and factual?
3. Consistency: No contradictions?
4. Actionability: Next agents have context?
5. Quality: Meets 100/100 standard?

## OUTPUT
- Approval: Yes/No
- Issues: List any gaps
- Recommendations: Improvement suggestions
`)
```

### 8.3 Design Principles

- **KISS**: Simple > Complex
- **YAGNI**: Build only what's needed now
- **Fail Fast**: Check errors early
- **Single Responsibility**: One purpose per unit

---

## SECTION 9: GAMIFICATION FRAMEWORK

All subagents use gamification:

**XP Rewards** (10-15 categories per operator)
**Progressive Levels** (1-5 with capability unlocks)
**Challenges** (consistency, innovation, speed, quality)
**Competition** (leaderboards, achievements)
**Purpose** (existential directive, winning conditions)

See detailed template in Section 2.3.

---

## SECTION 10: EXPECTED OUTCOMES

### 10.1 Deliverables

1. **Subject Map** (Mermaid diagrams)
2. **Gap Register** (50-200 gaps, scored)
3. **Risk Profile** (FMEA, edge cases)
4. **Opportunity Portfolio** (100-300 opportunities)
5. **Pareto Frontier** (optimal trade-offs)
6. **Implementation Roadmap** (3-12 months, phased)
7. **Task Breakdown** (ready for delegation)
8. **Final Report** (executive-ready)

### 10.2 Performance Metrics

- **Speed**: 1.85x faster with memory coordination
- **Quality**: 88% success with ReasoningBank vs 60% without
- **Scalability**: Linear to 10+ agents
- **Thoroughness**: 15-30% more gaps/opportunities vs single-agent

---

## SECTION 11: CRITICAL REMINDERS

### âœ… ALWAYS
1. **Init ReasoningBank FIRST** (`agent memory init`)
2. **99.9% Sequential** (dependencies require it)
3. **Forward-Looking Prompts** (tell agents about future needs)
4. **Memory Coordination** (Backend → Frontend flow)
5. **Gamify Subagents** (maximize engagement)
6. **Verify with Hooks** (pre-task, post-edit, session-end)
7. **Self-Assessment** (100/100 or iterate)
8. **Radical Honesty** (no lies, simulations, or illusions)

### âŒ NEVER
1. **Skip ReasoningBank Init** (60% success rate)
2. **Parallel with Dependencies** (type mismatches, failures)
3. **Prompts Without Context** (blind agents, delays)
4. **Skip Future Agent Needs** (rework, questions)
5. **Skip Verification** (quality issues)
6. **Forget Memory Storage** (next agents blocked)

---

## SECTION 12: QUICK REFERENCE

### Initialization
```bash
npx claude-flow@alpha init
npx claude-flow@alpha agent memory init
npx claude-flow@alpha agent memory status
```

### Memory Namespaces
```bash
search/session/*           # Config, history, metrics
search/state/*             # Current, best, archive
search/discovery/*         # Structural, flows, dependencies
search/gaps/*              # Quality, performance, structural
search/risks/*             # FMEA, edge-cases, vulnerabilities
search/opportunities/*     # Generated, scored, pareto
search/implementation/*    # Roadmap, tasks, resources
search/learning/*          # Patterns, strategies, failures
```

### Core Commands
```bash
# Store
npx claude-flow memory store --namespace "search/X" --key "Y" --value '{...}'

# Retrieve
npx claude-flow memory retrieve --key "search/X/Y"

# Hooks
npx claude-flow hooks pre-task --description "operator"
npx claude-flow hooks post-edit --memory-key "search/X/Y"
npx claude-flow hooks session-end --export-metrics

# Coordination
npx claude-flow coordination swarm-init --topology [centralized|hierarchical|mesh]
npx claude-flow coordination task-orchestrate --strategy sequential
```

### Agent Mapping
| Role | Claude Flow Agent |
|------|-------------------|
| Structural | system-architect |
| Flows | code-analyzer |
| Dependencies | dependency-analyzer |
| Performance | perf-analyzer |
| Gaps | code-reviewer |
| Risks | security-auditor |
| Opportunities | innovation-generator |
| Scoring | decision-analyzer |
| Optimization | optimization-engineer |
| Roadmap | project-manager |
| Tasks | task-breakdown-specialist |
| Synthesis | report-generator |
| Meta | adaptive-coordinator |

### Prompt Template (Every Agent)
```markdown
# {Operator} Excellence Framework

## WORKFLOW CONTEXT
Agent #{N}/{M} | Previous: {agents} ✓ | Next: {agents}

## MEMORY RETRIEVAL
\`\`\`bash
npx claude-flow memory retrieve --key "search/{namespace}/{key}"
\`\`\`

## MISSION
{Specific goal}

## MEMORY STORAGE
1. For {Next Agent}: "search/{ns}/{key}" - {what/why}
2. For {Future}: "search/{ns}/{key}" - {what/why}

## EXECUTION
{Steps}

## XP SYSTEM
{10-15 reward categories}

## SELF-ASSESSMENT
100/100 or iterate
```

### Execution Pattern
```bash
# Message 1: Operator A
Task("{agent}", "{gamified prompt}")
TodoWrite({ todos: [phases] })

# Message 2: Operator B (WAIT)
Task("{agent}", "{gamified prompt}")

# Message 3: Operator C (WAIT)
Task("{agent}", "{gamified prompt}")
```

### Quality Gates
```bash
# Before operator
npx claude-flow hooks pre-task

# After operator
npx claude-flow hooks post-edit --memory-key "search/X/Y"

# Verify storage
npx claude-flow memory retrieve --key "search/X/Y"

# Final validation
npx claude-flow hooks session-end
```

---

## SECTION 13: TROUBLESHOOTING

### Issue: "Agent can't find previous work"
**Cause**: Memory not stored or wrong namespace
**Fix**: 
```bash
# Verify memory exists
npx claude-flow memory retrieve --key "search/{namespace}/{key}"

# If missing, previous agent needs to re-store
# Check agent prompt included storage instructions
```

### Issue: "Dependencies not respected"
**Cause**: Parallel execution when should be sequential
**Fix**: 
```bash
# ALWAYS wait between dependent operators
# Message 1: Backend
# Message 2 (WAIT): Frontend

# Never run both in same message if dependencies exist
```

### Issue: "Low quality output (< 100/100)"
**Cause**: Agent didn't iterate to perfection
**Fix**: 
```bash
# Add self-assessment requirement to prompt
# "Rate: 1-100 vs mission. If < 100, iterate."

# Add reviewer subagent after complex operators
Task("meta-learning-agent", "Review {operator} output")
```

### Issue: "Search took too long"
**Cause**: Too many operators or inefficient sequencing
**Fix**:
```bash
# Use quick search pattern (5 agents instead of 16)
# Batch memory operations
# Use booster for large edits: npx claude-flow agent booster edit
```

### Issue: "Gaps not comprehensive"
**Cause**: Insufficient coverage or shallow analysis
**Fix**:
```bash
# Ensure gap-hunting agents have:
# - All discovery artifacts
# - Clear success criteria (30-50 gaps minimum)
# - Multiple dimensions (quality, performance, structural)

# Add reviewer to validate gap coverage
```

---

## SECTION 14: ADVANCED PATTERNS

### 14.1 Multi-Domain Search

Search across multiple domains simultaneously:

```bash
# E.g., Search both codebase AND business process

# Message 1: Codebase structural mapping
Task("system-architect", "Map software architecture")

# Message 2: Business process mapping (parallel, independent)
Task("process-analyst", "Map business processes")

# Message 3: Cross-domain synthesis (WAIT)
Task("system-architect", "Integrate technical and business views")
```

### 14.2 Continuous Search

Run search periodically to track evolution:

```bash
#!/bin/bash
# continuous_search.sh

while true; do
  TIMESTAMP=$(date +%Y%m%d_%H%M%S)
  
  # Run search
  ./universal_search.sh "$SUBJECT" "$TYPE" "$OBJECTIVES" "2h"
  
  # Store historical snapshot
  npx claude-flow memory store \
    --namespace "search/history/$TIMESTAMP" \
    --key "snapshot" \
    --value "$(npx claude-flow memory retrieve --key 'search/output/final_report')"
  
  # Compare to previous
  Task("comparative-analyzer", "Compare current vs previous search")
  
  # Sleep 1 week
  sleep 604800
done
```

### 14.3 Collaborative Search

Multiple teams searching same subject:

```bash
# Team A: Technical search
./universal_search.sh "$SUBJECT" "software" '["technical"]' "4h"
npx claude-flow memory store --namespace "search/team_a" --key "results"

# Team B: Business search
./universal_search.sh "$SUBJECT" "business" '["business"]' "4h"
npx claude-flow memory store --namespace "search/team_b" --key "results"

# Synthesis: Integrate perspectives
Task("integration-specialist", `
Retrieve both team results:
\`\`\`bash
npx claude-flow memory retrieve --key "search/team_a/results"
npx claude-flow memory retrieve --key "search/team_b/results"
\`\`\`

Synthesize: Unified roadmap addressing technical AND business needs
`)
```

---

## SECTION 15: METRICS & LEARNING

### 15.1 Collect Session Metrics

```bash
# At end of search
npx claude-flow hooks session-end --export-metrics
npx claude-flow analysis token-usage --breakdown
npx claude-flow analysis bottleneck-detect

# Store for future learning
npx claude-flow memory store \
  --namespace "search/learning/sessions" \
  --key "$(uuidgen)" \
  --value '{
    "subject": "...",
    "subject_type": "...",
    "duration_seconds": N,
    "agents_used": 16,
    "topology": "hierarchical",
    "quality_achieved": 0.95,
    "gaps_found": 78,
    "opportunities_generated": 152,
    "success": true,
    "bottlenecks": [...],
    "lessons": [...]
  }'
```

### 15.2 Learn from History

Before starting new search:

```bash
# Retrieve similar past searches
Task("memory-manager", `
Search learning database for similar subjects:

\`\`\`bash
npx claude-flow memory retrieve --namespace "search/learning/sessions"
\`\`\`

Filter by subject_type: {type}
Identify:
1. Successful strategies (what worked)
2. Failed approaches (what didn't work)
3. Optimal agent count
4. Best topology
5. Common gaps for this subject type

Use insights to optimize current search strategy.
`)
```

---

## CONCLUSION

The **Universal Search Algorithm for Claude Flow (USACF)** transforms the original UMASAF into a memory-coordinated, sequential execution framework optimized for Claude Flow's architecture.

### Key Innovations

1. **Memory-Based State Management**: All search state persisted in ReasoningBank
2. **Forward-Looking Coordination**: Agents store what future agents need
3. **99.9% Sequential Execution**: Respects dependencies for 88% success rate
4. **Gamified Subagents**: Maximize engagement and quality through XP systems
5. **Hook-Based Verification**: Quality gates at every step
6. **Universal Applicability**: Search ANY subject (software, business, process, product)
7. **Self-Improving**: Learns from each search session

### Usage Pattern

```bash
# 1. Init
npx claude-flow@alpha init && npx claude-flow@alpha agent memory init

# 2. Define
SUBJECT="Anything"
TYPE="software|business|process|product"
OBJECTIVES='["goals"]'

# 3. Execute
./universal_search.sh "$SUBJECT" "$TYPE" "$OBJECTIVES" "4h"

# 4. Results
npx claude-flow memory retrieve --key "search/output/final_report"
```

### Performance

- **88% success rate** (with ReasoningBank vs 60% without)
- **1.85x faster** (with memory coordination)
- **15-30% more thorough** (vs single-agent)
- **Universal**: Works on any searchable subject

### Remember

✅ Sequential execution (99.9%)
✅ Forward-looking prompts (future agent needs)
✅ Memory coordination (store for next agents)
✅ ReasoningBank first (88% vs 60%)
✅ Gamify all subagents (maximize quality)
✅ Radical honesty (truth above all)
✅ Self-assessment (100/100 or iterate)

**This is not just a framework—it's a universal search engine for understanding, powered by Claude Flow's memory-coordinated multi-agent architecture.**

---

**Framework Version**: 3.0 (Claude Flow Integrated)
**Created**: 2025-11-17
**Optimized For**: Claude Flow Alpha
**Maintained By**: Universal Search Algorithm Project