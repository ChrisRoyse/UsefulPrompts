# Designing Product Requirements Documents for AI Agents

## A Complete Framework for Unambiguous, Machine-Executable PRDs

---

## Executive Summary

Product Requirements Documents (PRDs) are experiencing a profound transformation as artificial intelligence agents move from theoretical possibilities to practical implementation partners in software delivery. While traditional PRDs have evolved over decades to serve human-centric teams, the introduction of AI agents—particularly large language model-based coding agents, research agents, and autonomous systems—demands a fundamentally different approach to requirements engineering.

This report synthesizes established PRD best practices with emerging techniques from specification-driven AI development to provide a comprehensive framework for designing PRDs that are simultaneously human-readable and machine-executable. The key insight is simple but powerful: **a PRD designed for AI agents must treat every requirement as a machine-parseable contract, not merely an aspirational narrative**.

The framework presented here builds on three governing principles:

1. **Intent as Truth**: PRDs capture the *what* and *why*; implementation details are deferred to technical specifications.
2. **Machine-First Readability**: Structured, unambiguous language with stable identifiers enable both human understanding and reliable LLM parsing.
3. **Context Persistence**: Self-contained PRDs enable agents to resume work with zero context loss, critical for distributed and asynchronous agent execution.

By applying this framework—from PRD decomposition through functional and technical specifications to deterministic task generation—organizations can reduce ambiguity, eliminate hallucination-prone gaps, and achieve **end-to-end traceability** from business intent to implementation, test, and verification. This report provides the complete workflow, templates, quality gates, and practical examples needed to make this shift.

---

## 1. Introduction: From Human Teams to Human+Agent Teams

### 1.1 What Traditional PRDs Are and What They Miss for AI

A conventional PRD is a document that captures the essence of what a software product should accomplish. Industry guidance consistently emphasizes clear objectives, personas, user journeys, feature descriptions, acceptance criteria, and success metrics. Tools like Atlassian, ProductPlan, and Aha! have popularized PRD templates that emphasize narrative clarity and business context over algorithmic precision.

This approach works well for human engineering teams because humans carry substantial tacit knowledge:

- **Contextual inference**: Engineers intuit whether "fast" means "sub-100ms" or "sub-5s" based on domain experience.
- **Ambiguity resolution**: Developers recognize that "secure storage" implies encryption with key management, not just filesystem permissions.
- **Gap-filling**: Teams ask clarifying questions, discuss trade-offs in meetings, and collectively resolve inconsistencies before code is written.
- **Institutional memory**: Decisions made six months ago live in Slack, wiki pages, and team members' heads; the PRD can be looser because context is reachable.

AI agents do not have these advantages. When a large language model-based agent reads a PRD:

- **No contextual inference**: The agent processes language statistically and is sensitive to phrasing, punctuation, and order. "Fast" without a number is ambiguous; the agent may optimize for throughput when you meant latency.
- **No clarification capacity**: Unlike humans who ask "what do you mean by secure?", agents must choose an interpretation or fail. Ambiguity becomes a decision point where agents explore all possible interpretations, often selecting suboptimal ones.
- **No institutional memory**: Each agent session is relatively stateless. Context from Slack, previous retrospectives, or informal decisions is invisible to the agent unless explicitly embedded in the PRD or related specifications.
- **Hallucination and confidence**: A common failure mode is agents confidently implementing the wrong thing. Unlike humans who feel uncertain about unclear requirements and escalate, agents may forge ahead with plausible-but-incorrect interpretations.

### 1.2 Why AI Agents Change Requirements Engineering

The introduction of autonomous or semi-autonomous AI agents into development pipelines creates new constraints on requirements engineering:

**Scale and Velocity**: Human developers complete 5–10 substantial tasks per week. AI agents can propose code, tests, and system designs at rates of tens to hundreds per hour. When scale increases, so does the magnitude of potential errors. A misinterpreted requirement that affects one human's work over a day can affect an agent's output for thousands of generated lines of code in minutes.

**Non-Determinism and Reproducibility**: Human decisions are traceable (a developer can explain why they chose one approach). LLM-based agents are probabilistic; the same PRD read by the same agent model at different temperatures or in different order may yield different outcomes. This requires PRDs to be so unambiguous that they constrain the probability space, making "any reasonable interpretation" lead to acceptable outcomes.

**Autonomy Levels**: Human teams operate at a consistent autonomy level; they execute tasks assigned by a manager. AI agents exist on a spectrum from fully supervised (co-pilot) to fully autonomous. A PRD that suffices for a human co-pilot (where a human reviews every decision) is inadequate for a Level 3–4 autonomous agent (where the agent makes most decisions and humans intervene only at escalation points). The PRD must pre-specify which decisions are safe for autonomous action and which require human approval.

**Tool and API Usage**: Modern agentic systems operate by invoking tools—APIs, code interpreters, file systems, databases. A PRD for human developers can say "integrate with the payment API." A PRD for an agent must specify exactly which endpoints are accessible, what error handling is required, which recovery strategies to use, and what must escalate to humans.

**Compliance and Audit**: Autonomous agents can make thousands of decisions per day. Regulatory bodies and risk teams need auditability: proof that every action traces back to an explicit requirement or policy. This demands PRDs where requirements are stable, identifiable, and connected to implementation through unbroken chains of traceability.

### 1.3 Scope and Objectives of This Report

This report provides a complete, opinionated framework for designing PRDs specifically for AI-driven delivery. It extends the **AI-Agent Specification Development Guide** (the foundation document you provided) upstream, showing how to structure PRDs so they feed seamlessly into the functional → technical → task hierarchy already described in that guide.

The framework is organized around six key themes:

1. **Conceptual Foundations**: How PRDs fit into a broader specification hierarchy and why machine-first readability matters.
2. **Structural Design**: Concrete patterns for PRD sections that LLM agents can parse reliably.
3. **Decomposition Discipline**: Systematic methods to extract requirements, edge cases, and non-functional constraints from PRD prose.
4. **Ambiguity Elimination**: Concrete techniques to replace vague language with machine-checkable definitions.
5. **Multi-Agent and Governance**: Extensions for systems with multiple agents, autonomy levels, and regulatory constraints.
6. **End-to-End Workflow**: Integrating PRDs into the specification→task→execution pipeline with quality gates at each step.

The report is addressed to product managers, technical leads, architects, and engineers who are adopting AI agents in their delivery workflow. It assumes familiarity with basic PRD concepts and provides concrete, implementable practices rather than abstract principles.

---

## 2. Conceptual Foundations: What Makes a PRD "AI-Executable"?

### 2.1 The Specification Hierarchy and PRD's Role

Your AI-Agent Specification Development Guide establishes a clear five-level hierarchy:

```
Level 1: Constitution (immutable rules, tech stack, coding standards)
Level 2: Functional Specs (what to build, user stories, requirements)
Level 3: Technical Specs (how to build, architecture, data models, APIs)
Level 4: Task Specs (atomic work units, exact signatures, constraints)
Level 5: Context Files (active session state, decisions, progress)
```

The PRD is the **source material for Levels 2–4**. It is not, itself, a functional or technical spec; rather, it is the business-layer input that gets decomposed and refined into those downstream artifacts.

Think of the relationship this way:

- **PRD** = "What problem are we solving, for whom, and what constitutes success?"
- **Functional Spec** = "What user-visible behaviors must the system exhibit?"
- **Technical Spec** = "What are the component contracts, data models, and API surfaces?"
- **Task Specs** = "What is the exact code (or implementation unit) that must be written?"

A well-formed PRD does not jump to technical details (e.g., "implement a Django REST API with PostgreSQL"). Instead, it captures intent: "Users need fast, reliable access to real-time alerts. The system must support 100K concurrent users with <200ms latency."

Later documents (technical specs, task specs) make the implementation choices. If those choices are later questioned or changed, the PRD remains stable, allowing for clear discussion of "did we deviate from the original intent?"

### 2.2 Three Governing Principles for AI-Executable PRDs

**Principle 1: Intent as Truth**

Your specification guide articulates that **intent as truth** means capturing *what* and *why*, not detailed *how*. At the PRD level, this principle means:

- The PRD is the **canonical representation of user-visible outcomes, problems, and constraints**—not implementation details.
- Any PRD statement about implementation (e.g., "use MySQL") must be explicitly labeled as non-binding or moved into the technical spec.
- Intent is made atomic and traceable via **intent blocks** with unique IDs.

Example intent block:

```xml
<product_intent id="INT-AUTH-01">
  <problem>
    Users abandon signup when account creation is complex. Current flow has 6 steps; 
    industry benchmark is 2–3 steps with 85%+ completion.
  </problem>
  <desired_outcome>
    First-time visitors can create a verified account in ≤60 seconds with ≥85% 
    completion rate.
  </desired_outcome>
  <success_metric>
    Measure signup conversion weekly. Alert if it drops below 80%.
  </success_metric>
  <constraints>
    Email verification is required (regulatory). Single sign-on is Phase 2.
  </constraints>
</product_intent>
```

Later, functional requirements (`REQ-AUTH-##`), user stories, and tasks point back to `INT-AUTH-01`. For an AI agent, this creates a stable semantic anchor: when implementing a requirement, the agent can trace back to understand *why* that requirement exists, which helps resolve local ambiguities.

**Principle 2: Machine-First Readability**

Most current PRD guidance assumes human readers. For AI agents, PRDs must be **both human-legible and machine-addressable**.

Machine-first readability means:

- **Structured wrapping**: Critical sections (objectives, personas, requirements, edge cases, constraints) are wrapped in consistent XML/JSON-like structures.
- **Stable identifiers**: Requirements, journeys, and constraints have permanent IDs (e.g., `REQ-`, `US-`, `NFR-`, `EC-`) assigned in the PRD, not added later.
- **Minimal ambiguity in layout**: No inline bullet lists hiding inside long paragraphs. Each concept gets its own section.
- **Deterministic parsing**: An agent can extract "all edge cases related to `REQ-AUTH-01`" with confidence, without relying on keyword matching or fuzzy search.

A machine-first PRD does not abandon markdown or human-friendly writing. Instead, you layer structured tags on top of narrative content:

```markdown
## Feature: Quick Registration

### Intent
Ref: INT-AUTH-01

### User Story
US-AUTH-001:
- As a new user visiting for the first time
- I want to create an account in under 60 seconds
- So that I can start using the product immediately

### Functional Requirements

REQ-AUTH-001 (Priority: MUST):
- The registration flow must complete in ≤ 60 seconds for typical users
- **Definition**: Measured on Chrome on a 5Mbps connection from signup load to post-verify confirmation screen

REQ-AUTH-002 (Priority: MUST):
- Email address must be the unique identifier
- Validation: RFC 5322 format; domain must resolve; confirmation email required

### Edge Cases

EC-AUTH-001 (Related to REQ-AUTH-002):
- Scenario: User enters email with typo (e.g., "gmial.com" instead of "gmail.com")
- Expected: System warns of likely typo; suggests fix; allows override
- Rationale: Prevents account lockout from simple mistakes

### Non-Functional Requirements

NFR-AUTH-001 (Category: Performance):
- POST /auth/register endpoint: p95 latency < 500ms under 100 req/s load

NFR-AUTH-002 (Category: Security):
- Passwords: minimum 12 characters, must contain uppercase, lowercase, number, symbol
- Rationale: NIST 800-63B guidelines
```

This structure allows agents to reliably extract related items (all reqs related to INT-AUTH-01, all edge cases for REQ-AUTH-002, etc.). Humans can still read it as a normal markdown document.

**Principle 3: Context Persistence**

Autonomous agents require more deliberate context design than human teams. Your specification hierarchy addresses this via Level 5 (Context Files): `.ai/activeContext.md`, `decisionLog.md`, and `progress.md`.

At the PRD layer, context persistence implies:

- The PRD is **self-contained** enough that an agent can understand the full scope and intent without replaying months of chat history or hunting through Slack threads.
- Key decisions that change "what we're building" (pivoted personas, removed features, changed KPIs) must be reflected as **versioned edits to the PRD**, not buried in logs.
- References from specs and tasks back to the PRD use **stable IDs and version numbers**, so agents can determine whether their work aligns with the *current* PRD version or an outdated one.

Practically:

- **PRD versioning**: Include `prd_version` and `effective_date` at the top. When the PRD changes, increment the version and document what changed in a "Changelog" section.
- **Linking to decisions**: If the `decisionLog.md` records a decision that affects PRD scope (e.g., "we descoped password reset"), link it explicitly in the PRD with a note like "See decisionLog entry DEC-042: Password reset deferred to Phase 2."
- **Expiration markers**: For assumptions that were made at a point in time but may drift (e.g., "user base is 1,000 active daily"), mark them with a review date: "Assumption: DAU ~1K (review quarterly; escalate if >10K)."

### 2.3 Requirements Engineering for AI Agents vs. Classical RE

Classical Requirements Engineering (RE) focuses on completeness, consistency, correctness, and feasibility. Most RE literature and practice assumes that humans are the primary interpreters and mediators of requirements.

When AI agents take on design, coding, or operations tasks, several new RE concerns emerge:

**1. Parseability**: The consumer is partially or fully non-human. LLMs are sensitive to phrasing, structure, and context. Ambiguities that a senior engineer resolves in seconds can cause an agent to choose wrong tools or call APIs with malformed arguments.

**2. Toolability**: Agents operate through APIs, tools, and other agents. Requirements must cover not just *what* the system should do, but *through what capabilities* agents will achieve it. Example: not just "system must support bulk user exports," but "POST /users/export/bulk, accepts CSV format, rate-limited to 1 request per minute, token-based auth required, expires jobs after 24h."

**3. Autonomy Constraints**: How much freedom does the agent have to decide? Requirements must pre-specify which decisions are safe (agent can auto-approve) and which require human escalation.

**4. Audit and Traceability**: Autonomous agents make many decisions per unit time. Regulators and risk teams need proof that every action traces back to an explicit requirement or policy. Requirements must be stable, identifiable, and connected to implementation and testing through unbroken chains.

**5. Error Handling and Recovery**: For human teams, error handling is often implicit ("if something goes wrong, the engineer will figure it out"). Agents need explicit error budgets, retry policies, and escalation rules defined in requirements.

You can extend classical RE dimensions as follows:

| RE Dimension | Classical Definition | AI-Extended Definition |
|---|---|---|
| **Completeness** | All functional and non-functional requirements captured | + All agent-relevant constraints (tooling, access boundaries, escalation points, autonomous decision criteria) explicitly specified |
| **Consistency** | No conflicting requirements across documents | + PRD, functional spec, technical spec, and task layer form a consistent graph with stable IDs; no dangling references; automated traceability checks pass |
| **Correctness** | Requirements can be validated with acceptance tests | + Each requirement includes or implies machine-checkable acceptance criteria (latency budgets, accuracy thresholds, safety checks); agents can verify behavior automatically |
| **Feasibility** | Requirements can be built within engineering constraints | + Agentic feasibility: Can the intended agent (with its context window, tool set, environment) actually complete this task end-to-end? Does the PRD contemplate necessary human oversight? |

When these extended RE concerns are applied consistently, the PRD becomes the **well-typed root of a multi-layer artifact tree** that agents can traverse deterministically.

---

## 3. Designing PRD Structure for AI Agents

### 3.1 Baseline PRD Sections (Adapted from Industry Templates)

Most modern PRD templates include these core sections. For AI-executable PRDs, we keep these but add AI-specific extensions:

**Metadata Section**
- PRD ID and version
- Effective date and expiration date
- Owner and key stakeholders
- Status (draft, in review, approved, archived)
- Links to related documents (vision, strategy, competitive analysis)

**Executive Summary**
- One-paragraph description of what the feature/product accomplishes
- Key business drivers (revenue impact, competitive differentiation, user retention)
- Success criteria in one sentence

**Problem Statement**
- What pain point are we solving?
- Who experiences it?
- How is it being solved today (current state)?
- What are the consequences of not solving it?

**Personas and Use Cases**
- Who are the primary users?
- What are their goals, frustrations, and contexts?
- What tasks do they perform?
- How frequently?

**Feature List and User Stories**
- High-level features and capabilities
- User stories in "As a [user type], I want [action] so that [benefit]" format
- Priority: MoSCoW (Must, Should, Could, Won't)
- Acceptance criteria for each story

**Non-Functional Requirements (NFRs)**
- Performance budgets (latency, throughput, capacity)
- Reliability and uptime targets
- Security and compliance requirements
- Accessibility, internationalization, and other cross-cutting concerns

**Success Metrics / KPIs**
- How do we know if this succeeds?
- Metrics to track (e.g., adoption rate, NPS, time-to-value, error rate)
- Target values and measurement frequency

**Scope and Non-Scope**
- What is in this release?
- What is explicitly out of scope?
- Known dependencies on other features or teams

**Release Strategy**
- Rollout plan (phased, blue-green, etc.)
- Go/no-go decision criteria
- Rollback procedures

### 3.2 AI-Specific Extensions to the PRD

Building on the baseline, AI-executable PRDs require additional sections:

**Agent Roles and Responsibilities**
- Which parts of this feature are built by AI agents vs. humans?
- Example: "AI coding agents implement the API layer. Humans do security review and acceptance testing."
- Implications: Agents must have clear specs for what they own; humans must have clear specs for what they validate.

**Agent Capability Assumptions**
- What model class is expected to implement this? (e.g., "reasoning model like Claude 3.5 Sonnet," "code-specialized model like Codestral," "lightweight model like Qwen 2.5")
- Context window limits (e.g., "agent has 100K token context; full codebase is 50K tokens, leaving 50K for task spec and reasoning")
- Latency constraints (e.g., "each task must complete in <10 minutes for real-time feedback")
- Cost constraints (e.g., "budget $0.50 per task on API costs")

**Tooling and Environment Assumptions**
- What APIs, repositories, databases, and external systems can agents access?
- Authentication and authorization (what tokens, keys, or credentials does the agent have?)
- MCP (Machine Control Protocol) servers or function tools available?
- File system access? Code interpreter access? Ability to make HTTP calls?

**Autonomy Level**
- Specify the intended autonomy level (using frameworks like Bessemer's L1–L5 or Knight Institute's operator/collaborator/consultant/approver/observer):
  - **L1 / Operator**: Agent proposes actions; human always reviews before execution.
  - **L2 / Collaborator**: Agent and human jointly plan; frequent back-and-forth.
  - **L3 / Consultant**: Agent proposes actions; human reviews significant decisions; most small decisions are autonomous.
  - **L4 / Approver**: Agent acts autonomously; human reviews outcomes; escalates anomalies.
  - **L5 / Observer**: Agent operates fully autonomously; humans monitor for anomalies.
- Different features may have different autonomy levels. Specify per-feature or per-agent.

**Risk Posture and Guardrails**
- What are the failure modes we're willing to tolerate?
- What are non-negotiable safety constraints? (e.g., "agent cannot delete data without explicit user confirmation and backup verification")
- What must escalate to humans? (e.g., "any access to PII fields requires human approval")
- What are the recovery procedures if the agent makes a mistake?

### 3.3 Structural Patterns That Help LLM Parsing

Modern LLMs are surprisingly good at parsing structured text, but patterns matter:

**Use Consistent Wrappers for Critical Sections**
- All requirements wrapped in consistent XML-like syntax (or markdown with consistent heading hierarchy)
- All acceptance criteria in a consistent format
- All constraints called out in a dedicated section, not scattered through prose

**Enforce Stable Identifiers**
- Assign IDs to requirements (`REQ-DOMAIN-##`), user stories (`US-##`), edge cases (`EC-##`), non-functional requirements (`NFR-##`), acceptance criteria (`AC-##`)
- These IDs are the "foreign keys" that link PRD → Functional Spec → Technical Spec → Tasks
- Once assigned, IDs do not change (even if the requirement text is refined)

**Separate Intent from Implementation**
- If the PRD mentions implementation details, clearly label them as "suggested approach" or move them to the technical spec
- Example: Not "implement in PostgreSQL", but "system requires ACID compliance for transaction safety; team recommends PostgreSQL but alternative is acceptable"

**Use Enumerations and Tables**
- Personas: table with name, goals, pain points, usage patterns
- Requirements: table with ID, description, priority, related story, rationale
- Constraints: list with category, constraint, justification
- Edge cases: table with scenario, expected behavior, related requirement

Tables are easier for agents to parse than free-form prose.

### 3.4 Quality Gates for PRD Review (Before Decomposition)

Before a PRD is decomposed into functional and technical specs, it should pass a quality gate:

**Clarity Gate**
- [ ] No vague language ("fast," "simple," "intuitive," "robust") without numeric definitions
- [ ] All terms defined or obvious to a technical audience
- [ ] Examples provided for complex concepts

**Completeness Gate**
- [ ] All user journeys have associated personas
- [ ] All user journeys have acceptance criteria
- [ ] All features have success metrics
- [ ] All constraints (performance, security, compliance) are explicit

**Consistency Gate**
- [ ] No contradictory requirements (e.g., "low latency" and "strong consistency" both required without clarification of trade-off)
- [ ] All acronyms defined
- [ ] All references to related features are traceable

**Ambiguity Gate**
- [ ] All requirements with success criteria are measurable (not subjective)
- [ ] All error conditions are specified (not just happy path)
- [ ] Scaling limits are explicit (e.g., "supports up to X users, Y transactions per second")

**AI-Readiness Gate**
- [ ] All agent autonomy levels specified
- [ ] All agent tooling constraints specified
- [ ] All escalation rules specified
- [ ] All failure modes and recovery procedures specified

---

## 4. From PRD to Decomposed Requirements for Agents

### 4.1 Applying the Decomposition Process Upstream

Your specification guide provides a five-step PRD decomposition process:

1. **Extract User Journeys**: Identify user types, triggers, success criteria, failure modes
2. **Identify Functional Domains**: Auth, CRUD operations, workflows, integrations, analytics, admin
3. **Extract Requirements with IDs**: Assign `REQ-DOMAIN-##` IDs; enable traceability
4. **Identify Non-Functional Requirements**: Surface performance, security, reliability, accessibility, compliance
5. **Surface Edge Cases**: For each requirement, ask what fails, what are boundaries, what are invalid inputs

This decomposition is the critical bridge between high-level PRD intent and detailed specifications. It is best captured in a "PRD Analysis" document that mirrors the decomposition template provided in your guide.

### 4.2 Making PRD Content Traceable from Day One

The moment a PRD is approved, every requirement should get a permanent ID. The pattern is:

```
REQ-[DOMAIN]-[SEQUENCE]
```

Examples:
- `REQ-AUTH-001`: First requirement in the Auth domain
- `REQ-ORDER-042`: 42nd requirement in the Order domain
- `REQ-PAYMENT-005`: Fifth requirement in Payment domain

Key discipline: **IDs never change or get reused**. If you decide a requirement is no longer needed, mark it as "DEPRECATED (as of v2.1)" but do not delete it. This preserves traceability: if a task or test references `REQ-AUTH-001`, it is always clear what that ID meant, even across versions.

Enforce traceability links:

- Every user story should link to at least one requirement: `US-AUTH-001 → REQ-AUTH-001, REQ-AUTH-002`
- Every requirement should link to the user story that motivated it: `REQ-AUTH-001 ← US-AUTH-001`
- Every functional requirement should link to at least one non-functional requirement or constraint: `REQ-AUTH-001 → NFR-AUTH-002 (latency), SEC-CONSTRAINT-001 (password policy)`
- Every acceptance criterion should link to the requirement it validates: `AC-AUTH-001a ← REQ-AUTH-001`

These links form a graph. Use automated tools (or manual review) to check for gaps: "Which requirements have no acceptance criteria? Which user stories have no requirements? Which NFRs are not tested?"

### 4.3 Capturing Non-Functional Requirements with Metrics

NFRs are often implicit in traditional PRDs ("the system should be fast"). For AI-driven delivery, NFRs must be explicit and measurable:

**Performance NFRs**:
```
NFR-PERF-001: API Response Time
- Metric: p95 latency for POST /users
- Target: < 200ms under normal load (≤100 req/s)
- Measurement: CloudWatch percentile metric
- Failure mode: Alert if p95 > 300ms for 5+ minutes
```

**Reliability NFRs**:
```
NFR-REL-001: Availability
- Metric: System uptime
- Target: 99.9% monthly uptime (allow 43 minutes downtime/month)
- Measurement: Synthetic health check every 60s
- Failure mode: Page on-call; trigger incident if downtime > 15 minutes
```

**Security NFRs**:
```
NFR-SEC-001: Password Policy
- Metric: Enforced password strength
- Target: Minimum 12 characters; must contain uppercase, lowercase, digit, symbol
- Measurement: Password validation function; test suite checks constraints
- Failure mode: Registration endpoint rejects weak passwords with clear error message
```

**Compliance NFRs**:
```
NFR-COMP-001: Data Retention
- Metric: User data retention period
- Target: Delete all user data 30 days after account deletion request
- Measurement: Weekly audit job; reports any data >30 days old post-deletion
- Failure mode: Alert compliance team; manual review + deletion
```

NFRs should be organized by category (Performance, Reliability, Security, Compliance, Accessibility, Scalability) and each should have:
- **ID** (NFR-CATEGORY-##)
- **Description** (one sentence)
- **Metric** (what you measure)
- **Target** (the number)
- **Measurement Method** (how you collect the data)
- **Failure Mode** (what happens if target is missed)

### 4.4 Systematically Surfacing Edge Cases in PRDs

Edge cases are requirements that specify behavior outside the happy path. They are critical for AI agents because agents will encounter edge cases, and if the PRD doesn't specify expected behavior, agents will make it up (often incorrectly).

For each functional requirement, surface edge cases by asking:

- **Null/Empty**: What if the input is null, empty, or missing?
- **Boundaries**: What if the input is at the boundary of valid range? (e.g., zero items, maximum integer, very long string)
- **Concurrency**: What if two requests happen simultaneously? (e.g., two edits to the same record, two checkouts of the last item in stock)
- **Permissions**: What if the user doesn't have permission?
- **Failures**: What if a dependent service is down? (e.g., payment API, email service)
- **Rate Limits**: What if the user exceeds rate limits?
- **Data Consistency**: What if the data is in an unexpected state? (e.g., order exists but customer doesn't)
- **Timeouts**: What if the request takes too long?

Capture edge cases in a structured table:

```
| EC ID | Related Req | Scenario | Expected Behavior |
|-------|-------------|----------|------------------|
| EC-AUTH-001 | REQ-AUTH-002 | User enters email with typo (gmial.com) | System warns of likely typo; suggests correction; allows override |
| EC-AUTH-002 | REQ-AUTH-003 | User's email is already registered | System shows error: "Email already in use. Try login or password reset." |
| EC-AUTH-003 | REQ-AUTH-004 | Email verification link expires (>24h) | System shows "link expired"; offers to resend verification |
| EC-AUTH-004 | REQ-AUTH-004 | User clicks verification link twice | System handles idempotently; shows success both times |
| EC-AUTH-005 | REQ-AUTH-005 | Password reset request for non-existent email | System shows generic success message to prevent email enumeration attacks |
```

Every functional requirement should have at least 2–3 associated edge cases. If a requirement has no edge cases, either it's trivial (mark it) or you haven't thought through the failure modes.

---

## 5. Spec Models Derived from the PRD

### 5.1 Deriving Functional Specs from the PRD

Once a PRD is decomposed and all requirements are identified, the next step is to create functional specifications. The PRD is the *input*; the functional spec is the *output*.

Your specification guide provides a functional spec template. The mapping is:

- **PRD user stories** → **Functional spec `<user_stories>` section**
- **PRD requirements** → **Functional spec `<requirements>` section**
- **PRD edge cases** → **Functional spec `<edge_cases>` section**
- **PRD NFRs and failure modes** → **Functional spec `<error_states>` section**
- **PRD acceptance criteria** → **Functional spec `<test_plan>` section**

The functional spec should preserve all IDs from the PRD (REQ-AUTH-001 remains REQ-AUTH-001) and add new IDs for items created during spec development (error state IDs, test case IDs).

Example derivation:

```
PRD:
  Feature: Quick Registration
  US-AUTH-001: As a new user, I want to create an account in <60 seconds
  REQ-AUTH-001: Registration flow must complete in ≤60 seconds
  REQ-AUTH-002: Email must be unique identifier
  AC-AUTH-001: System rejects duplicate emails

↓ Becomes ↓

Functional Spec:
  <functional_spec id="SPEC-AUTH-REGISTRATION">
    <requirement id="REQ-AUTH-001">
      <description>Registration flow completes in ≤60 seconds</description>
      <acceptance_criteria>
        <criterion id="AC-AUTH-001">
          Given: User on signup page
          When: User fills form and clicks Submit
          Then: Confirmation screen displays within 60 seconds
        </criterion>
      </acceptance_criteria>
    </requirement>
    <error_states>
      <error id="ERR-AUTH-001" http_code="409">
        <condition>Email already registered</condition>
        <message>"This email is already registered. Please log in or use a different email."</message>
        <recovery>Link to login page; link to password reset</recovery>
      </error>
    </error_states>
    <test_plan>
      <test_case id="TC-AUTH-001">
        <description>Successful registration completes within 60 seconds</description>
        <test_data>email: "user@example.com", password: "ValidP@ssw0rd"</test_data>
        <expected>Confirmation screen displays; email received</expected>
      </test_case>
    </test_plan>
  </functional_spec>
```

### 5.2 Deriving Technical Specs for AI-Driven Systems

The technical spec layer adds implementation details: architecture, data models, API contracts, component contracts, and implementation notes.

Derived from the functional spec, a technical spec might look like:

```xml
<technical_spec id="TECH-AUTH-REGISTRATION" implements="SPEC-AUTH-REGISTRATION">
  <architecture>
    Registration flow: User → API Gateway → Auth Service → User DB → Email Queue
    Latency budget: API Gateway 10ms, Auth Service 100ms, Email Queue 50ms (async)
    Total: 160ms p50, 300ms p95 (leaving buffer for network jitter)
  </architecture>
  
  <data_models>
    <model name="User">
      <field name="id" type="UUID" constraints="primary_key, auto-generated"/>
      <field name="email" type="string(255)" constraints="unique, indexed, not_null"/>
      <field name="password_hash" type="string(60)" constraints="bcrypt, not_null"/>
      <field name="email_verified" type="boolean" constraints="default: false"/>
      <field name="created_at" type="timestamp" constraints="auto, not_null"/>
    </model>
  </data_models>
  
  <api_contracts>
    <endpoint path="/auth/register" method="POST" implements="REQ-AUTH-001, REQ-AUTH-002">
      <request_body>
        {
          "email": "user@example.com",
          "password": "ValidP@ssw0rd",
          "name": "John Doe"
        }
      </request_body>
      <responses>
        201 Created: {id, email, name, message: "Check your email to verify"}
        400 Bad Request: {error: "validation_error", details: [{field, message}]}
        409 Conflict: {error: "email_exists", message: "This email is already registered"}
      </responses>
    </endpoint>
  </api_contracts>
  
  <component_contracts>
    <component name="AuthService" path="src/services/auth.service.ts">
      <method name="registerUser">
        <signature>async registerUser(dto: RegisterDto): Promise<User></signature>
        <implements>REQ-AUTH-001, REQ-AUTH-002</implements>
        <behavior>
          1. Validate email format (RFC 5322)
          2. Check email uniqueness (query db)
          3. Validate password strength
          4. Hash password with bcrypt (cost=12)
          5. Create user record (transaction)
          6. Queue verification email (async)
          7. Return user object (without password_hash)
        </behavior>
        <throws>ValidationError, ConflictError, WeakPasswordError</throws>
      </method>
    </component>
  </component_contracts>
</technical_spec>
```

Key additions in the technical spec that are **not** in the functional spec:

- **Architecture**: Specific tech stack choices, service boundaries, communication patterns
- **Data models**: Exact field types, constraints, indexes, relationships
- **API contracts**: Request/response schemas, HTTP status codes, error details
- **Component contracts**: Exact method signatures, exception types, implementation behavior
- **Performance details**: Latency budgets per component; scaling decisions
- **Implementation notes**: Trade-offs, known limitations, future enhancements

### 5.3 Agent-Specific Specification Extensions

For systems where AI agents are primary implementers, the technical spec should include agent-specific information:

```xml
<agent_specs>
  <agent_role name="SWE Agent">
    <responsibilities>
      Implement data models, services, and API endpoints per technical spec
    </responsibilities>
    <constraints>
      - Cannot access production databases; only staging
      - Cannot deploy to production; only create PRs for human review
      - Must write unit tests for business logic (80% coverage minimum)
      - Cannot modify security or compliance code without explicit approval
    </constraints>
    <tool_access>
      - GitHub API: read/write to feature branches; PRs only
      - AWS (staging): read/write; no production access
      - Slack: post updates to #engineering channel; read-only elsewhere
    </tool_access>
    <context_window>
      Available: 100K tokens
      Codebase snapshot: 50K tokens
      Spec and task: 20K tokens
      Reasoning/scratch: 30K tokens
    </context_window>
  </agent_role>
  
  <failure_modes>
    <mode id="FM-001" probability="medium" severity="high">
      <description>Agent generates SQL injection vulnerability</description>
      <mitigation>
        - SQL always parameterized (verified by linter and human review)
        - Forbidden: string concatenation for SQL; mandatory: ORM methods
      </mitigation>
    </mode>
    <mode id="FM-002" probability="low" severity="critical">
      <description>Agent deletes production data</description>
      <mitigation>
        - Agent has zero production database access (enforced by IAM)
        - All writes test database only
        - Delete operations always require human confirmation
      </mitigation>
    </mode>
  </failure_modes>
</agent_specs>
```

### 5.4 Traceability Matrix Blueprint

A traceability matrix connects PRD → Functional Spec → Technical Spec → Tasks → Tests. At the spec level, it looks like:

```
| PRD Item | Functional Spec Item | Technical Spec Item | Task | Status |
|----------|---------------------|-------------------|------|--------|
| INT-AUTH-01 (intent) | SPEC-AUTH-REGISTRATION | TECH-AUTH-REGISTRATION | — | ✓ |
| US-AUTH-001 | US-AUTH-001 (copied) | Referenced in component_contracts | — | ✓ |
| REQ-AUTH-001 | REQ-AUTH-001 in requirements | REQ-AUTH-001 in endpoint contract | TASK-AUTH-008 | — |
| REQ-AUTH-002 | REQ-AUTH-002 in requirements | REQ-AUTH-002 in data_models | TASK-AUTH-001 | — |
| EC-AUTH-001 | EC-AUTH-001 in edge_cases | Handled by validation in component | TASK-AUTH-005 | — |
| NFR-PERF-001 | — | Latency budget in architecture_diagram | TASK-AUTH-009 | — |
```

Create this matrix in a spreadsheet or markdown table. Use automated tools to check coverage: "Which PRD requirements have no functional spec? Which functional specs have no technical spec? Which technical specs have no corresponding tasks?"

---

## 6. Eliminating Ambiguity for AI Agents

### 6.1 Linguistic Ambiguity

Vague language is the enemy of AI-executable specs. Common problem words and how to replace them:

| Problem Word | Problem | Solution | Example |
|---|---|---|---|
| "Fast" | Subjective; agent may optimize wrong metric | Define latency in milliseconds, percentile, and load | "< 200ms p95 response time under 100 req/s" |
| "Simple" | Unclear; agent may build minimal but wrong | Define in terms of user actions or clicks | "Signup takes ≤ 3 screens, each ≤ 1 minute" |
| "Intuitive" | Subjective; no objective measure | Define via user research or usability criteria | "70% of new users complete task without help" |
| "Secure" | Vague; agent may implement wrong controls | Specify algorithms, key sizes, cert standards | "HTTPS/TLS 1.3; AES-256-GCM encryption; OWASP Top 10 protections" |
| "Reliable" | Vague; agent may not add redundancy | Define uptime percentage or error budgets | "99.9% uptime; < 0.1% error rate" |
| "Scalable" | Unclear limits; agent may not design for growth | Specify expected growth and performance at scale | "Support 1M users with <200ms p95 latency; auto-scale infrastructure" |

**Time Zones, Locales, and Formats**:

Specify explicitly:

```
NON-AMBIGUOUS:
- Timestamps stored as UTC ISO 8601 (e.g., 2025-12-04T15:25:00Z)
- User-facing times displayed in user's local time zone
- Date format in UI: MM/DD/YYYY for US, DD/MM/YYYY for EU
- Decimal separator: period (.) in APIs; locale-aware in UI
```

**Number Ranges and Limits**:

```
AMBIGUOUS: "Support up to 10K users"
CLEAR: "Support up to 10,000 concurrent users; scale to 100K with auto-scaling"

AMBIGUOUS: "Handle large files"
CLEAR: "Support file uploads up to 5GB; reject files >5GB with clear error message"

AMBIGUOUS: "Very fast response time"
CLEAR: "p50 latency <100ms; p95 latency <300ms; p99 latency <1000ms"
```

### 6.2 Structural Ambiguity

How requirements are organized affects LLM parsing. Avoid:

- **Mixed concerns in one block**: "The system must be fast, reliable, and secure" → three separate requirements
- **Interleaving intent and weak implementation**: "Use PostgreSQL for reliability" → separate "why" (need ACID compliance) from "what" (PostgreSQL is suggested but not required)
- **Implicit negations**: "Support all payment methods" vs. explicit list of methods

Better structure:

```markdown
### Requirement: Payment Processing

#### Intent
Users need convenient payment options; support multiple methods for accessibility.

#### What
System must support: credit/debit card (Visa, Mastercard, Amex), PayPal, Apple Pay, Google Pay

#### Constraints
- Card transactions: PCI-DSS compliant
- All methods: tokenized (no raw card data stored)
- Latency: payment validation <500ms p95
- Retry: failed transactions auto-retry up to 3 times

#### What We're NOT doing
- ACH transfers (complex; low volume)
- Wire transfers (high friction)
- Cryptocurrency (unclear regulation)
```

### 6.3 Tooling and Environment Ambiguity

Specify exactly what agents can do:

```
AMBIGUOUS:
"Agent can use available APIs to implement features"

CLEAR:
"Agent has access to:
- Stripe Payments API (https://api.stripe.com/v1/*)
  - Methods: POST /charges, GET /charges/{id}
  - Auth: Bearer token from STRIPE_API_KEY env var
  - Rate limit: 100 req/s
- Customer DB (PostgreSQL at prod-db.internal:5432)
  - Read: all schemas; Write: customers, orders schemas only
  - Auth: connection string from DB_CONN env var
  - No access to: payments schema (PCI-DSS restricted), admin schema
- Email Service (internal SMTP at mail.internal:25)
  - Methods: SMTP AUTH via credentials; send-to address must match @company.com
  - Rate limit: 100 emails/min

Agent CANNOT:
- Make HTTP calls outside the approved list
- Execute arbitrary CLI commands
- Access /etc, /root, or other privileged directories
- Modify code in security/ or compliance/ directories without human review"
```

### 6.4 Human–Agent Ambiguity

Define decision boundaries:

```
DECISION BOUNDARY EXAMPLE:

Feature: User Invitation Flow

AGENT CAN DECIDE AUTONOMOUSLY:
- Email template formatting (as long as brand guidelines are followed)
- Retry logic for failed email sends (standard exponential backoff)
- Database schema for invitation tokens

HUMAN MUST APPROVE:
- Changes to invitation expiration time (business decision)
- Who can send invitations (requires business rules review)
- What information is exposed in the invitation link (security review needed)

AGENT MUST ESCALATE IF:
- Invitation accepted by non-existent user (ask: create account or reject?)
- Invitation sent to company's admin email (ask: intended or mistake?)
- Bulk invitation request >1000 at once (ask: legitimate or abuse?)
```

### 6.5 Rewrite Patterns: From Vague to Precise

Here are real-world before/after examples:

**Example 1: Performance**

```
VAGUE PRD:
"The system should be fast and responsive."

CLEAR PRD:
REQ-PERF-001: API Response Time
- Metric: p95 latency for GET /users/{id}
- Target: < 100ms (measured from API Gateway entry to response body sent)
- Load: Under normal load (≤100 req/s)
- Test: CloudWatch Insights; alert if >150ms for 5 min
- Measurement method: Every request logged with response_time_ms

REQ-PERF-002: UI Responsiveness
- Metric: First Contentful Paint (FCP)
- Target: < 1.5 seconds on 3G networks (as measured by Lighthouse)
- Test: Weekly Lighthouse runs; alert if FCP > 2s
```

**Example 2: Security**

```
VAGUE PRD:
"The system must be secure and comply with industry standards."

CLEAR PRD:
REQ-SEC-001: Authentication
- Method: OAuth 2.0 with PKCE (for SPAs)
- Session duration: 24 hours; must re-authenticate after logout
- Password policy: Min 12 chars; uppercase, lowercase, digit, symbol required
- Verification: NIST 800-63-3 section 5.1.1

REQ-SEC-002: Data in Transit
- Protocol: HTTPS/TLS 1.2 or higher
- Certificate: Issued by trusted CA; valid for custom domain
- Test: SSLLabs grade A or higher

REQ-SEC-003: Data at Rest
- Sensitive fields (email, password_hash): AES-256-GCM encryption
- Encryption keys: Stored in AWS Secrets Manager; rotated annually
- Test: Encryption verified in code review; keys verified in security audit
```

**Example 3: Error Handling**

```
VAGUE PRD:
"Handle errors gracefully and show helpful messages."

CLEAR PRD:
REQ-ERR-001: Database Connection Failure
- Trigger: Cannot connect to primary DB for >5 seconds
- User-facing message: "Service temporarily unavailable. Trying again..."
- Retry strategy: Exponential backoff; 5s, 10s, 20s, 40s; give up after 4 attempts
- Escalation: Page on-call if unresolved after 5 minutes

REQ-ERR-002: External API Timeout
- Trigger: Third-party API doesn't respond within timeout (Stripe: 10s)
- User-facing message: "Payment processing delayed. We'll confirm your order shortly."
- Retry strategy: Up to 3 times; use idempotency keys to prevent double-charges
- Escalation: Email user if not resolved within 1 hour

REQ-ERR-003: Invalid Input
- Trigger: User submits invalid email, short password, etc.
- User-facing message: Specific to field (not generic "validation failed")
- Example: "Password must be at least 12 characters and include a symbol"
- No server-side logging of user input; no PII in error messages
```

---

## 7. Multi-Agent and Multi-System PRDs

### 7.1 PRDs for Multi-Agent Systems vs. Single-Agent

When a single PRD describes work intended for multiple agents, new considerations emerge.

**Single-Agent PRD**: One agent (e.g., a SWE agent) implements a feature end-to-end. The PRD describes what the agent should build.

**Multi-Agent PRD**: Multiple agents with different roles collaborate. Example:
- Planner agent: reads feature PRD, decomposes into tasks
- Coder agent: implements tasks
- Reviewer agent: reviews code against requirements
- Tester agent: generates and runs tests

For multi-agent PRDs, specify:

```xml
<multi_agent_orchestration>
  <workflow>
    1. Planner reads PRD (this document)
    2. Planner generates tasks (TASK-FEATURE-001, TASK-FEATURE-002, ...)
    3. Planner assigns tasks to Coder agents based on domain
    4. Coder agents execute tasks in dependency order
    5. Reviewer agent checks each code submission against:
       - Functional spec requirements (REQ-*)
       - Technical spec contracts (API, data model)
       - Constitution (coding standards, security rules)
    6. Tester agent generates test cases for each requirement
    7. Human (PO, tech lead) reviews PRs and approves for merge
  </workflow>
  
  <agent_roles>
    <role id="planner" model="reasoning">
      <responsibility>Decompose PRD into atomic, ordered tasks</responsibility>
      <output>Task specs with exact signatures and constraints</output>
    </role>
    <role id="coder" model="code-specialist">
      <responsibility>Implement task specs</responsibility>
      <input>Task spec with definition_of_done</input>
      <output>Pull request with code and tests</output>
    </role>
    <role id="reviewer" model="reasoning">
      <responsibility>Validate code against spec and constitution</responsibility>
      <input>Code, test results, spec references</input>
      <output>Approve/request changes</output>
    </role>
  </agent_roles>
</multi_agent_orchestration>
```

### 7.2 Specifying Divisions of Labor

In multi-agent systems, explicitly define who does what:

| Concern | Planner Agent | Coder Agent | Reviewer Agent | Human |
|---------|---|---|---|---|
| Understand requirements | ✓ | ✓ | ✓ | — |
| Decompose into tasks | ✓ | — | — | Approve |
| Implement code | — | ✓ | — | Final review |
| Write tests | ✓ (spec) | ✓ (impl) | — | — |
| Catch regressions | — | — | ✓ | — |
| Architectural decisions | — | — | ✓ (validation) | Final say |
| Approval for production | — | — | — | ✓ |

Specify escalation rules: "If reviewer agent finds a code pattern that violates constitution, it requests changes. If it finds an ambiguity in the spec, it escalates to human for clarification."

### 7.3 Cross-Agent Contracts in the PRD

Multi-agent systems require well-defined contracts (interfaces, data schemas, message formats):

```xml
<inter_agent_contracts>
  <contract id="CONTRACT-PLANNER-CODER">
    <from>Planner</from>
    <to>Coder</to>
    <message_type>TaskAssignment</message_type>
    <schema>
      {
        "task_id": "TASK-FEATURE-001",
        "title": "Create User Entity and Migration",
        "spec_ref": "SPEC-AUTH-REGISTRATION",
        "input_files": ["specs/technical/auth.md"],
        "definition_of_done": {
          "signatures": ["export class User { ... }"],
          "tests": ["User entity compiles", "Migration runs without errors"]
        },
        "constraints": ["Use TypeORM", "Support PostgreSQL and MySQL"],
        "deadline": "2025-12-05T18:00:00Z"
      }
    </schema>
  </contract>
  
  <contract id="CONTRACT-CODER-REVIEWER">
    <from>Coder</from>
    <to>Reviewer</to>
    <message_type>CodeReview</message_type>
    <schema>
      {
        "task_id": "TASK-FEATURE-001",
        "pr_link": "https://github.com/...",
        "implementation_notes": "Created User entity with TypeORM...",
        "tests_passing": true,
        "coverage_percent": 85,
        "requirement_refs": ["REQ-AUTH-001", "REQ-AUTH-002"]
      }
    </schema>
  </contract>
</inter_agent_contracts>
```

### 7.4 Monitoring, Observability, and Accountability Requirements

With multiple agents making decisions, observability becomes essential:

```xml
<observability_requirements>
  <requirement id="OBS-001">
    <name>Audit Trail for All Decisions</name>
    <description>Every agent decision must be logged with: agent_id, decision_type, 
      timestamp, context (PRD ref, task ref), outcome, rationale</description>
    <example>
      Agent: coder-001
      Decision: "Implemented UserRepository using TypeORM"
      Context: TASK-AUTH-001, REQ-AUTH-001
      Timestamp: 2025-12-04T10:30:00Z
      Rationale: "Tech spec specifies TypeORM"
    </example>
  </requirement>
  
  <requirement id="OBS-002">
    <name>Traceability from Requirements to Implementation</name>
    <description>All code must have comments linking to requirements. Example: 
      // Implements REQ-AUTH-001: Email must be unique identifier</description>
  </requirement>
  
  <requirement id="OBS-003">
    <name>Real-Time Escalation Alerts</name>
    <description>If any agent detects ambiguity, error, or violation, 
      immediately alert human. Include context (what was the ambiguity? what requirement?)</description>
  </requirement>
  
  <requirement id="OBS-004">
    <name>Compliance Reporting</name>
    <description>Monthly report: which requirements were implemented by which agents, 
      testing coverage per requirement, any deviations or escalations</description>
  </requirement>
</observability_requirements>
```

---

## 8. Governance, Risk, and Compliance Dimensions in AI-Oriented PRDs

### 8.1 Expanded Risk Sections for Autonomous Agents

Traditional PRDs often have a "Risks and Mitigation" section. For AI-driven systems, expand this section to cover agentic-specific risks:

**Risk Category: Safety**
```
RISK-SAFETY-001: Agent Makes Unauthorized Changes
- Scenario: Agent modifies data or code outside its intended scope
- Likelihood: Medium (if scope not well-defined)
- Impact: High (data corruption, security breach)
- Mitigation:
  * Agent has explicit access controls (IAM policies, database permissions)
  * All writes logged with agent_id and requirement reference
  * Human review required for any write to security/compliance code
  * Dry-run mode: agent proposes changes; human approves before execution
```

**Risk Category: Reliability**
```
RISK-RELIABILITY-001: Agent Generates Incorrect Code
- Scenario: Agent misunderstands requirement; generates code that compiles but doesn't meet spec
- Likelihood: High (partial interpretation common in LLMs)
- Impact: High (broken feature, user-facing bugs)
- Mitigation:
  * Comprehensive test plan in functional spec
  * Code review by human specialist
  * Automated tests must pass before PR approval
  * Periodic regression testing
```

**Risk Category: Cost**
```
RISK-COST-001: Agent Makes Expensive API Calls
- Scenario: Agent calls expensive third-party API excessively
- Likelihood: Medium (unbounded retry loops)
- Impact: Medium (increased operational costs)
- Mitigation:
  * Agent has per-request cost budget (e.g., $0.10 max per task)
  * Rate limits enforced
  * Escalation if cost exceeds budget
  * Monitoring and alerting for unusual spending patterns
```

### 8.2 Guardrails and Constitutions as First-Class PRD Artifacts

Your specification guide's "Constitution" layer is about immutable rules (tech stack, coding standards, security policies). In an AI-driven PRD, these guardrails should be explicitly referenced:

```xml
<guardrails>
  <guardrail id="GUARD-1" category="Security">
    <rule>No direct SQL string concatenation. All queries must be parameterized.</rule>
    <enforcement>Linter checks; code review; human approval required for exceptions</enforcement>
    <reference>Constitution.md, section "Anti-Patterns"</reference>
  </guardrail>
  
  <guardrail id="GUARD-2" category="Performance">
    <rule>No N+1 database queries. Eager-load related entities.</rule>
    <enforcement>Automated query analyzer; human review if flagged</enforcement>
    <reference>Constitution.md, section "Performance Requirements"</reference>
  </guardrail>
  
  <guardrail id="GUARD-3" category="Autonomy">
    <rule>Agent cannot delete user data. Soft-deletes only; human approval required.</rule>
    <enforcement>Code cannot call DELETE from users table; DELETE only marks is_deleted=true</enforcement>
    <reference>Autonomy level: Level 2 (agent proposes; human approves)</reference>
  </guardrail>
  
  <guardrail id="GUARD-4" category="Compliance">
    <rule>All user data stored at-rest is encrypted (AES-256).</rule>
    <enforcement>All string-type columns storing PII must use encrypted field type</enforcement>
    <reference>Constitution.md, section "Security Requirements"</reference>
  </guardrail>
</guardrails>
```

### 8.3 Human-in-the-Loop and Human-Over-the-Loop Requirements

Define where humans must intervene:

```xml
<human_oversight>
  <checkpoint id="HIL-1" stage="Planning">
    <trigger>Agent proposes task decomposition</trigger>
    <human_action>Review task ordering, dependencies, completeness</human_action>
    <approval_required>Yes (PM or tech lead)</approval_required>
    <sla>Review within 4 business hours</sla>
  </checkpoint>
  
  <checkpoint id="HIL-2" stage="Code Review">
    <trigger>Agent submits code for a security-related requirement</trigger>
    <human_action>Security specialist reviews for vulnerabilities</human_action>
    <approval_required>Yes (security team)</approval_required>
    <sla>Approval within 24 hours</sla>
  </checkpoint>
  
  <checkpoint id="HIL-3" stage="Escalation">
    <trigger>Agent encounters ambiguous requirement or decision point</trigger>
    <human_action>Agent escalates; human provides clarification</human_action>
    <approval_required>Yes (immediately)</approval_required>
    <sla>Response within 2 hours (or agent waits)</sla>
  </checkpoint>
  
  <checkpoint id="HIL-4" stage="Deployment">
    <trigger>All tests pass; code is approved</trigger>
    <human_action>Final approval before deployment to production</human_action>
    <approval_required>Yes (tech lead or manager)</approval_required>
    <sla>Approval within 4 business hours</sla>
  </checkpoint>
  
  <over_the_loop_monitoring>
    <monitor>Agent is generating unusually high error rate (>10%)</monitor>
    <action>Automatically pause agent; page on-call to investigate</action>
    
    <monitor>Agent has violated guardrail (tried to write unparameterized SQL)</monitor>
    <action>Block execution; alert security team; add new test to catch this pattern</action>
    
    <monitor>Agent spending exceeds monthly budget</monitor>
    <action>Throttle agent usage; escalate to finance team</action>
  </over_the_loop_monitoring>
</human_oversight>
```

### 8.4 Failure Handling and Incident Response Requirements

Specify how failures are handled:

```xml
<failure_handling>
  <scenario id="FH-1">
    <failure>Agent generates code with a security vulnerability (e.g., SQL injection)</failure>
    <detection>
      - Automated linter detects pattern
      - Human reviewer flags during code review
      - Security scanning tool fails CI/CD gate
    </detection>
    <response>
      1. Block PR merge
      2. Alert security team (Slack + ticket)
      3. Request agent regenerate code
      4. Add test case to prevent recurrence
      5. Post-mortem if pattern repeats >2x
    </response>
    <timeline>Detect <1 min; respond <30 min
  </scenario>
  
  <scenario id="FH-2">
    <failure>Agent misinterprets requirement; implements wrong behavior</failure>
    <detection>
      - Integration tests fail
      - Human acceptance testing reveals wrong behavior
    </detection>
    <response>
      1. Revert code changes
      2. Clarify requirement with PM
      3. Ask agent to regenerate with clarified spec
      4. Update PRD if ambiguity was root cause
    </response>
    <timeline>Detect at test time; respond <4 hours
  </scenario>
  
  <scenario id="FH-3">
    <failure>Agent makes hundreds of failed API calls; incurs high cost</failure>
    <detection>
      - Cost monitoring alert (threshold exceeded)
    </detection>
    <response>
      1. Immediately throttle/pause agent
      2. Investigate logs (why the retry loop?)
      3. Fix underlying issue (timeout, rate limit, etc.)
      4. Add cost budget constraint to task spec
    </response>
    <timeline>Detect <5 min; respond <15 min
  </scenario>
  
  <rollback_procedure>
    If deployed code causes production incident:
    1. Identify which PRD requirement the code implements
    2. Revert to previous version (automated or manual)
    3. Document what went wrong (escalation log)
    4. Regenerate code with additional constraints
    5. Extended testing before re-deploy
  </rollback_procedure>
</failure_handling>
```

---

## 9. Workflow: From PRD to Deterministic Assembly Line

### 9.1 Complete Pipeline with Quality Gates

The full workflow integrates PRD creation, decomposition, specs, tasks, execution, and feedback:

```
┌─────────────────────────────────────────────────────────────────────┐
│ Business Input → PRD → PRD Analysis                                │
│ (PM, stakeholders define what to build)                            │
└─────────────────────────────────────────────────────────────────────┘
                            ↓
                 ★ PRD_REVIEW GATE ★
                 (clarity, completeness, AI-readiness)
                            ↓
┌─────────────────────────────────────────────────────────────────────┐
│ Functional Spec, Technical Spec                                    │
│ (Architects define how to build it)                                │
└─────────────────────────────────────────────────────────────────────┘
                            ↓
                 ★ SPEC_REVIEW GATE ★
                 (traceability to PRD, completeness, feasibility)
                            ↓
┌─────────────────────────────────────────────────────────────────────┐
│ Task Generation, Dependency Graph, Traceability Matrix             │
│ (Decompose into atomic, ordered tasks)                             │
└─────────────────────────────────────────────────────────────────────┘
                            ↓
                 ★ TASK_REVIEW GATE ★
                 (ordering correct, no gaps, definitions precise)
                            ↓
┌─────────────────────────────────────────────────────────────────────┐
│ Sequential Agent Execution                                         │
│ For each task IN ORDER:                                            │
│   1. Read input context files                                      │
│   2. Read task spec (definition_of_done, constraints)              │
│   3. Execute task (write code, tests, etc.)                        │
│   4. Run self-verification (lint, type-check, tests)               │
│   5. Update context files                                          │
└─────────────────────────────────────────────────────────────────────┘
                            ↓
                 ★ CODE_REVIEW GATE ★
                 (signatures match, no regressions, guardrails met)
                            ↓
┌─────────────────────────────────────────────────────────────────────┐
│ Acceptance Testing, Deployment, Monitoring                         │
└─────────────────────────────────────────────────────────────────────┘
                            ↓
            ★ Continuous Feedback Loop ★
      (Telemetry feeds back into next PRD version)
```

### 9.2 Quality Gates Across the Pipeline

**PRD_REVIEW Gate** (before spec creation)

Purpose: Ensure PRD is complete, clear, and ready for spec decomposition.

Checklist:
- [ ] All user stories have acceptance criteria
- [ ] All features have success metrics
- [ ] No ambiguous language without definitions ("fast" → <200ms)
- [ ] All constraints explicit (performance, security, compliance)
- [ ] Agent autonomy level specified
- [ ] All agent tooling and access defined
- [ ] Edge cases identified for major requirements
- [ ] Risk and mitigation strategies documented
- [ ] Human oversight checkpoints defined

**SPEC_REVIEW Gate** (before task generation)

Purpose: Ensure functional and technical specs are complete and traceable to PRD.

Checklist:
- [ ] All PRD requirements have corresponding spec requirements
- [ ] Traceability matrix shows no gaps
- [ ] Data models defined with exact constraints
- [ ] API contracts have request/response schemas
- [ ] Component contracts have exact signatures
- [ ] All edge cases from PRD mapped to error states
- [ ] Performance budgets match NFRs
- [ ] Security constraints explicitly addressed
- [ ] No ambiguous language in specs

**TASK_REVIEW Gate** (before agent execution)

Purpose: Ensure tasks are atomic, ordered, and have precise definitions.

Checklist:
- [ ] Dependency graph is acyclic
- [ ] Foundation → Logic → Surface layer ordering observed
- [ ] Each task has definition_of_done with exact signatures
- [ ] Each task has minimal input_context_files
- [ ] No task references files from future tasks
- [ ] Traceability matrix covers all spec items
- [ ] Test commands specified for each task
- [ ] Constraints explicitly listed
- [ ] Estimated complexity and SLA defined

**CODE_REVIEW Gate** (after agent execution)

Purpose: Ensure code matches spec and constitution.

Checklist:
- [ ] All tests pass
- [ ] Code compiles without errors or warnings
- [ ] Signatures match definition_of_done exactly
- [ ] Comments link code to requirements (e.g., "// Implements REQ-AUTH-001")
- [ ] No forbidden patterns (hardcoded values, console.log, etc.)
- [ ] No security vulnerabilities detected by linter
- [ ] No N+1 queries or obvious performance issues
- [ ] Constitution naming conventions followed
- [ ] Test coverage meets minimum (80%)

### 9.3 Role-Specific Checklists

**For Product Managers** (PRD authoring and review)

- [ ] Is the problem statement clear? Can a stranger understand it?
- [ ] Are personas realistic and grounded in user research?
- [ ] Is the success metric quantifiable and measurable?
- [ ] Have we listed what we're NOT doing (non-scope)?
- [ ] Are assumptions about user behavior validated?
- [ ] Have we thought through failure modes?
- [ ] Is the autonomy level of agents defined?

**For Architects** (Spec design)

- [ ] Do all PRD requirements have spec requirements?
- [ ] Are data models consistent with entities?
- [ ] Do API contracts cover all user journeys?
- [ ] Are error scenarios explicitly handled?
- [ ] Do we have performance budgets for each component?
- [ ] Have we identified third-party dependencies?
- [ ] Is scaling architecture clear?

**For QA/Testing**

- [ ] Does the test plan cover all requirements?
- [ ] Are acceptance criteria testable?
- [ ] Have we defined test data and environments?
- [ ] Are edge cases included in test plan?
- [ ] Do we have regression tests?
- [ ] Is performance testing included?

**For Security**

- [ ] Are all security requirements explicit?
- [ ] Have we done threat modeling?
- [ ] Are guardrails clearly documented?
- [ ] Is data flow through the system secure?
- [ ] Have we addressed compliance requirements?
- [ ] Are audit/logging requirements defined?

### 9.4 Feedback Loops: Using Agent Behavior to Refine Future PRDs

After agents execute tasks and code is deployed, collect telemetry to improve future PRDs:

```xml
<feedback_loop>
  <metric id="FM-001">
    <name>Requirements Ambiguity Rate</name>
    <definition>
      Percentage of tasks that required clarification or were regenerated due to 
      agent misinterpretation
    </definition>
    <tracking>For each PRD version, log: tasks_total, tasks_clarified, 
      clarification_reason</tracking>
    <action>If >20% of tasks require clarification, PRD authoring process needs improvement</action>
  </metric>
  
  <metric id="FM-002">
    <name>Code Review Defect Rate</name>
    <definition>
      Percentage of agent-generated code that fails code review (security, 
      performance, architecture issues)
    </definition>
    <tracking>For each task, log: defects_found, defect_category, 
      defect_severity</tracking>
    <action>
      If defect rate >10%, investigate:
      - Is the task spec unclear?
      - Is the agent model inadequate?
      - Are guardrails insufficient?
    </action>
  </metric>
  
  <metric id="FM-003">
    <name>Spec Drift</name>
    <definition>
      Percentage of deployed code that deviates from technical spec (e.g., 
      uses different algorithm, different API contract)
    </definition>
    <tracking>Quarterly spec audit: sample deployed code; compare to original spec</tracking>
    <action>If drift >5%, review specification completeness; 
      tighten spec or improve spec enforcement</action>
  </metric>
  
  <metric id="FM-004">
    <name>Requirement Coverage</name>
    <definition>
      Percentage of PRD requirements that made it all the way to deployed code and 
      passing tests
    </definition>
    <tracking>
      PRD requirements: N
      Spec requirements: N (should equal PRD)
      Tasks generated: M (should cover all spec)
      Tasks completed: M' (should equal M)
      Tests passing: coverage_percent
    </tracking>
    <action>If coverage <95%, investigate gaps; update process</action>
  </metric>
  
  <continuous_improvement>
    Every quarter:
    1. Collect feedback metrics from all PRDs from past quarter
    2. Identify top sources of rework (ambiguity, spec drift, defects)
    3. Update PRD template and guidelines
    4. Train PMs on lessons learned
    5. Share with agent developers to improve prompts/models
  </continuous_improvement>
</feedback_loop>
```

---

## 10. Concrete PRD Templates and Examples for AI Agent Work

### 10.1 AI-Ready PRD Top-Level Template

Below is a complete, opinionated PRD template structured for AI agent implementation:

```markdown
# Product Requirements Document: [Feature Name]

**PRD ID**: PRD-[YEAR]-[SEQUENCE]  
**Version**: 1.0  
**Effective Date**: YYYY-MM-DD  
**Owner**: [PM Name]  
**Status**: Approved

---

## 1. Executive Summary

[One paragraph: what is this feature, why does it matter, expected impact]

---

## 2. Problem Statement

### Current State
[How is this problem solved today? What is the workflow?]

### Pain Points
[What specifically frustrates users? Include research/data.]

### Opportunity
[What would change if we solved this? Quantify impact.]

---

## 3. Target Users and Personas

| Persona | Goals | Pain Points | Usage Frequency |
|---------|-------|-------------|-----------------|
| [Name] | [Goal 1, Goal 2] | [Pain 1, Pain 2] | [Weekly/Daily/Monthly] |

---

## 4. Feature Description and User Stories

### Feature Overview
[High-level description of what users will be able to do]

### User Stories

**US-[DOMAIN]-001**: Priority [MUST/SHOULD/COULD]
```
As a [user type]
I want to [action]
So that [benefit]
```
**Acceptance Criteria**:
- [ ] [AC-##]
- [ ] [AC-##]

---

## 5. Functional Requirements

| ID | Description | Priority | Related Story | Rationale |
|----|-------------|----------|---------------|-----------|
| REQ-[DOMAIN]-001 | [Description] | MUST | US-[DOMAIN]-001 | [Why needed] |
| REQ-[DOMAIN]-002 | [Description] | SHOULD | US-[DOMAIN]-002 | [Why needed] |

---

## 6. Non-Functional Requirements

| ID | Category | Requirement | Metric | Target |
|----|----------|-------------|--------|--------|
| NFR-PERF-001 | Performance | API response time | p95 latency | <200ms |
| NFR-REL-001 | Reliability | Uptime | Monthly uptime % | 99.9% |
| NFR-SEC-001 | Security | Authentication | Method | OAuth 2.0 + PKCE |

---

## 7. Edge Cases

| ID | Related Req | Scenario | Expected Behavior |
|----|-------------|----------|------------------|
| EC-[DOMAIN]-001 | REQ-[DOMAIN]-001 | [Edge case] | [Expected behavior] |

---

## 8. Out of Scope

- [Feature not included in this release]
- [Deferred to Phase 2]
- [Not our responsibility]

---

## 9. Success Metrics

| Metric | Target | Measurement Frequency | Alert Threshold |
|--------|--------|----------------------|-----------------|
| [KPI Name] | [Target] | Weekly/Monthly | [Threshold] |

---

## 10. Agent Implementation Details

### Agent Roles and Responsibilities
- [Agent type]: [What it builds]
- [Agent type]: [What it builds]

### Agent Autonomy Level
[L1–L5; specify what agents can decide vs. what requires human approval]

### Agent Tooling and Access
- APIs: [List]
- Databases: [Connections and permissions]
- Rate limits: [Limits]
- Forbidden actions: [List]

### Agent Capability Assumptions
- Model class: [e.g., code-specialist model]
- Context window: [X tokens available]
- Latency budget per task: [Y minutes]
- Cost budget per task: [Z dollars]

---

## 11. Guardrails and Constraints

| ID | Category | Constraint | Enforcement |
|----|----------|-----------|-------------|
| GUARD-001 | Security | [Rule] | [How enforced] |
| GUARD-002 | Performance | [Rule] | [How enforced] |

---

## 12. Risk and Mitigation

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| [Risk] | [L/M/H] | [L/M/H] | [How mitigated] |

---

## 13. Human Oversight Checkpoints

| Checkpoint | Trigger | Human Action | Approval Required | SLA |
|-----------|---------|--------------|------------------|-----|
| [Checkpoint] | [When] | [What human does] | [Yes/No] | [Time] |

---

## 14. Success Criteria for Delivery

- [ ] All functional requirements implemented and tested
- [ ] All tests passing (unit, integration, e2e)
- [ ] Code review completed; no critical findings
- [ ] Security review completed
- [ ] Documentation updated
- [ ] Monitoring and alerts configured

---

## Appendix A: Glossary

[Define all acronyms and domain-specific terms]

---

## Appendix B: Related Documents

- Link to technical specifications
- Link to design documents
- Link to competitive analysis
- Link to user research

---

```

### 10.2 Example PRD for a Coding Agent

Here's a concrete example: a feature where a coding agent implements a user invitation system.

```markdown
# PRD: User Invitation System

**PRD ID**: PRD-2025-042  
**Version**: 1.0  
**Effective Date**: 2025-12-04  
**Owner**: Sarah Chen (Product Manager)  
**Status**: Approved

---

## 1. Executive Summary

Users need to quickly invite collaborators to projects. Currently, there's no self-serve invitation mechanism; all invitations require admin intervention. This feature enables project owners to send invitations directly, improving onboarding time from 2–3 days to minutes.

**Expected Impact**: 40% faster team onboarding; reduced support tickets.

---

## 2. Problem Statement

### Current State
- Admin manually adds users via backend tools
- Users email the admin requesting access
- Process takes 2–3 days; poor UX

### Pain Points
- Project owners feel blocked; cannot control who joins
- Admins bottleneck for every request
- New users cannot start work until manually added

### Opportunity
- Self-serve invitations reduce support burden
- Faster team assembly → projects start sooner
- Better ownership model: project owner controls membership

---

## 3. Target Users

| Persona | Goals | Pain Points | Usage Frequency |
|---------|-------|-------------|-----------------|
| Project Owner | Control team membership; add collaborators quickly | Slow manual process; no visibility | 2–3x per week |
| Invited User | Quick access to project resources | Wait time for admin; unclear process | Upon invitation |

---

## 4. User Stories

**US-INVITE-001**: Priority: MUST
```
As a project owner
I want to invite collaborators by email
So that my team can start working immediately without admin delay
```
**Acceptance Criteria**:
- [ ] Owner can enter email address(es) and click "Send Invite"
- [ ] Invited user receives email within 1 minute
- [ ] Invited user clicks link and gains project access
- [ ] Owner can see invitation status (pending, accepted, expired)

**US-INVITE-002**: Priority: SHOULD
```
As a project owner
I want to customize the invitation message
So that I can explain context and welcome the person
```

**US-INVITE-003**: Priority: COULD
```
As an admin
I want to see all pending invitations across projects
So that I can manage invitations and prevent abuse
```

---

## 5. Functional Requirements

| ID | Description | Priority | Related Story | Rationale |
|----|-------------|----------|---------------|-----------|
| REQ-INV-001 | Project owner can invite collaborators via email | MUST | US-INVITE-001 | Core feature |
| REQ-INV-002 | Invitation email arrives within 1 minute | MUST | US-INVITE-001 | User expectation |
| REQ-INV-003 | Invitation link is valid for 7 days; expires after | MUST | US-INVITE-001 | Security; prevent stale links |
| REQ-INV-004 | Invited user clicks link and gains project access | MUST | US-INVITE-001 | Completion of flow |
| REQ-INV-005 | Owner can see status of pending/accepted/expired invites | SHOULD | US-INVITE-001 | UX clarity |
| REQ-INV-006 | Owner can customize invitation message | SHOULD | US-INVITE-002 | Personalization |
| REQ-INV-007 | Invitations cannot be sent to already-members | MUST | US-INVITE-001 | Error handling |
| REQ-INV-008 | Bulk invitations (CSV upload) supported | COULD | US-INVITE-001 | Convenience |

---

## 6. Non-Functional Requirements

| ID | Category | Requirement | Target |
|----|----------|-------------|--------|
| NFR-PERF-001 | Performance | POST /projects/{id}/invitations response time | p95 < 100ms |
| NFR-PERF-002 | Throughput | Support bulk invite (100 emails in one request) | Complete <5s |
| NFR-REL-001 | Reliability | Email delivery success rate | >99% |
| NFR-SEC-001 | Security | Invitation link uses secure token (CSRF-protected) | Cryptographically random; 32+ bytes |
| NFR-SEC-002 | Security | Only project members with OWNER role can invite | RBAC enforced |
| NFR-COMP-001 | Compliance | Invitation history logged for audit | Full audit trail; 1-year retention |
| NFR-SCAL-001 | Scalability | Supports up to 10K invitations per day | Rate limiting: 100 inv/min per user |

---

## 7. Edge Cases

| ID | Related Req | Scenario | Expected Behavior |
|----|-------------|----------|------------------|
| EC-INV-001 | REQ-INV-007 | User tries to invite already-member | Show error: "User is already a member" |
| EC-INV-002 | REQ-INV-003 | User clicks expired invitation link | Show error: "Link expired (7 days old). Request new invitation." |
| EC-INV-003 | REQ-INV-004 | Invited user already has account; clicks link | Auto-login if not already logged in; show project access confirmation |
| EC-INV-004 | REQ-INV-001 | Invited email doesn't exist (typo: gmial.com) | Warn owner: "Email looks like it might be a typo. Did you mean gmail.com?" Allow override |
| EC-INV-005 | REQ-INV-002 | Email service is temporarily down | Queue invitation; retry up to 5 times over 24h; notify owner if failed |
| EC-INV-006 | REQ-INV-005 | Owner revokes invitation before recipient accepts | Update status to "revoked"; disable link |

---

## 8. Out of Scope

- Social login for invited users (users must create account or use existing)
- Tiered access levels for invitations (all invitations give full project access)
- Invitation expiration customization (always 7 days)
- Mobile app UI for invitations (web-only for Phase 1)

---

## 9. Success Metrics

| Metric | Target | Frequency | Alert |
|--------|--------|-----------|-------|
| Invitations sent per day | >500 | Daily | <100 = anomaly |
| Email delivery success rate | >99% | Daily | <98% = alert |
| Invitation acceptance rate | >70% | Weekly | <50% = investigate |
| Average time to acceptance | <4 hours | Weekly | >24h = concerning trend |
| Support tickets mentioning invitations | 0 | Weekly | >0 = investigate |

---

## 10. Agent Implementation Details

### Agent Roles
- **Coding Agent**: Implements API endpoints, database migrations, email service integration, tests
- **Testing Agent**: Writes integration tests, edge case coverage
- **Human**: Code review, security review, acceptance testing

### Agent Autonomy Level
- **L2 (Collaborator)**: Agent implements; human reviews all code before merge
- **Escalation**: Agent asks human if invitation email service fails; human decides retry vs. notify owner

### Agent Tooling Access
```
APIs Available:
- Internal User Service (authentication, profile lookup)
- Internal Email Service (queue email, retry logic)
- Project Database (read/write to invitations table, projects table, users table)

Rate Limits:
- Email service: 1000 emails/min
- Database: 100 writes/sec

Forbidden:
- Cannot call external email APIs (use internal service)
- Cannot modify user roles or permissions
- Cannot delete invitations (only revoke)
```

### Agent Capability Assumptions
- Model: Code-specialist (Codestral or similar)
- Context: 80K tokens; codebase ~40K tokens
- Latency: Each task <5 minutes
- Cost: <$0.50 per task

---

## 11. Guardrails and Constraints

| ID | Category | Constraint | Enforcement |
|----|----------|-----------|-------------|
| GUARD-INV-001 | Security | Invitation tokens are cryptographically random (min 32 bytes) | Code review; security audit |
| GUARD-INV-002 | Security | CSRF protection on invitation endpoints | Linter checks for @CSRF decorator |
| GUARD-INV-003 | Data | Invitation history is never deleted (audit trail) | No DELETE operations; only soft-deletes |
| GUARD-INV-004 | Performance | No N+1 queries when fetching invitations | Linter checks; code review |
| GUARD-INV-005 | Performance | Email sending is async (doesn't block API response) | Code review; verify queue integration |

---

## 12. Risk and Mitigation

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| Email service down → invitations not sent | Medium | High | Queue emails; retry 5x over 24h; notify owner |
| Attacker guesses/reuses invitation token | Low | Critical | Use cryptographically random tokens; expires 7 days |
| User spams invitations (DoS) | Medium | Medium | Rate limiting: 100 inv/min per user; alert on suspicious patterns |
| Invited user never gets email (spam filter) | Medium | Medium | Owner can see "pending" status; manual retry option |
| Sensitive data in invitation email | Low | High | Email template reviewed by security; no PII beyond email address |

---

## 13. Human Oversight Checkpoints

| Checkpoint | Trigger | Human Action | Approval Required | SLA |
|-----------|---------|--------------|------------------|-----|
| PRD Review | PRD finalized | PM + tech lead review for completeness | Yes | 1 day |
| Code Review | Agent submits PR | Tech lead + security review code | Yes | 1 day |
| Security Review | Code review passed | Security team reviews token handling, CSRF | Yes | 1 day |
| Acceptance Test | All tests passing | PM manually tests with real project | Yes | 1 day |
| Deployment Approval | Acceptance test passed | Tech lead approves deploy to staging → prod | Yes | Same day |
| Monitoring Setup | Code deployed | Ops team confirms alerts are firing | Implicit | Same day |

---

## 14. Success Criteria for Delivery

- [ ] All REQ-INV-\* requirements implemented
- [ ] All edge cases handled with appropriate error messages
- [ ] Unit test coverage: >85%; integration test coverage: 100% of user flows
- [ ] Load test: supports 100 invitations/second without degradation
- [ ] Security review passed; no CSRF or token-guessing vulnerabilities
- [ ] Email delivery >99% success rate in staging + production testing
- [ ] Monitoring configured for email failures, rate limiting, error rates
- [ ] Documentation updated (API, architecture, runbook)
- [ ] PM accepts via user acceptance test
- [ ] Zero support tickets in first week post-launch

---

## Appendix A: Glossary

- **Invitation Token**: Cryptographically random string that validates the invitation link
- **CSRF**: Cross-Site Request Forgery; attack type; mitigated by token validation
- **Async**: Email sending queued; response returned to user before email sent
- **Rate Limiting**: Max 100 invitations per minute per user; enforced at API level

---

## Appendix B: Related Documents

- [Technical Specification: Invitation System](./specs/technical/invitations.md)
- [Email Service SLA](./runbooks/email-service.md)
- [Database Schema: invitations Table](./db/schema.sql)

---
```

### 10.3 Example: Research and Summarization Agent PRD

For contrast, here's a PRD for a research agent (not code-focused):

```markdown
# PRD: Competitive Intelligence Agent

**PRD ID**: PRD-2025-051  
**Version**: 1.0  
**Effective Date**: 2025-12-04  
**Owner**: Michael Torres (Product Strategy)  
**Status**: Approved

---

## 1. Executive Summary

The product team needs rapid competitive intelligence to inform product roadmap decisions. Currently, competitive analysis is manual and slow (weeks). This agent autonomously monitors competitor announcements, analyzes feature releases, and summarizes implications—enabling data-driven prioritization.

---

## 2. Problem Statement

### Current State
- Team manually monitors competitor blogs, press releases, Twitter
- Analysis is ad-hoc; takes 2–4 weeks per analysis
- Insights are lost; no structured repository

### Pain Points
- Slow time-to-insight; decisions made without recent data
- Inconsistent analysis quality
- No audit trail of why decisions were made

### Opportunity
- Real-time competitive alerting
- Faster strategic prioritization
- Structured repository enables learning over time

---

[Personas, stories, requirements follow similar structure as coding example...]

---

## 10. Agent Implementation Details

### Agent Roles
- **Research Agent**: Monitor competitor websites, parse announcements, extract features
- **Summarization Agent**: Synthesize findings into strategic implications
- **Human**: Review summaries; make strategic decisions

### Agent Autonomy Level
- **L4 (Approver)**: Agent autonomously gathers and summarizes; human reviews and decides action
- **Escalation**: If competitor makes major announcement (funding, acquisition), immediately escalate to human

### Agent Tooling Access
```
APIs Available:
- Web scraping (competitor websites, Twitter API)
- Slack API (post summaries to #product-strategy)
- Postgres database (write research findings)

Forbidden:
- Cannot contact competitors directly
- Cannot modify internal product roadmap (read-only database)
```

### Constraints
- Summary must cite sources
- Tone must be objective (no opinions)
- Latency: Daily summary ready by 8 AM; urgent alerts <1 hour
- Cost: <$2/day

---

[Risk mitigation, guardrails follow...]

---

## 14. Success Criteria

- [ ] Daily summaries posted by 8 AM
- [ ] 95% of summaries cite sources correctly
- [ ] Product team action items created from 20%+ of summaries
- [ ] Time-to-insight: <24 hours from competitor announcement
- [ ] Zero hallucinations (invented competitor features)
- [ ] Sources include: 10+ competitors' official channels
- [ ] False positive rate: <5% (erroneous alerts)

---
```

---

## 11. Conclusion and Future Directions

### Current State and Adoption

Product Requirements Documents are evolving in response to AI agent adoption. Organizations that treat PRDs as machine-executable contracts—not just human narratives—are seeing measurable improvements in delivery speed and quality. The framework presented in this report synthesizes emerging best practices from specification-driven development, multi-agent systems research, and production deployments of LLM-based coding and operations agents.

The key insight: **a PRD that is clear enough for an AI agent to implement correctly is necessarily clear enough for a human team to implement correctly**. Clarity benefits all parties.

### Remaining Gaps and Research Opportunities

**Formal Verification**: Can we formally verify that a set of tasks, once executed, satisfies all PRD requirements? Some teams are exploring Z3 theorem provers and SAT solvers to check task coverage.

**Simulation**: Before executing tasks, can agents simulate the system against the PRD to catch architectural issues early? Research in model-based testing and simulation is relevant here.

**Ambiguity Detection**: Can we automatically detect ambiguous language in PRDs and propose clarifications? NLP and test generation techniques could help.

**Multi-Model Consistency**: Can we ensure that agents using different models (one coding model, one reasoning model) produce consistent implementations? Contract-based approaches show promise.

### Recommendations for Adoption

**For organizations starting with AI agents:**

1. **Start with a single, well-defined feature**: Apply this framework to one feature end-to-end before scaling.
2. **Invest in specification discipline**: The upfront cost (more rigorous PRDs, formal specs) pays dividends in reduced rework.
3. **Use version control for specs**: Treat specs like code; use Git to track changes and reasons.
4. **Establish quality gates**: Automate traceability checks; make spec coverage visible.
5. **Collect feedback metrics**: Track ambiguity rates, defect rates, delivery time; iterate on process.

**For enterprise deployments:**

1. **Standardize templates and identifiers**: All projects use same PRD, spec, task formats.
2. **Build internal tools**: Scripts to auto-generate traceability matrices, check coverage, lint specifications.
3. **Train PMs and architects**: Invest in training so they write AI-ready specs.
4. **Establish compliance review**: Have security, legal, and compliance teams review PRDs with regulatory requirements.
5. **Monitor agent performance**: Track error rates, cost, accuracy; adjust agent models and prompts based on data.

### The Path Forward

As AI agents become mainstream development partners, requirements engineering practices must evolve to meet them. This report provides a concrete, implementable framework. The next step is adoption: teams who apply these principles to their next feature will experience firsthand how clarity, structure, and traceability transform the relationship between intent and implementation.

The deterministic assembly line is not science fiction; it is a practical reality for teams willing to invest in rigor.

---

## Appendix: References

[1] Atlassian. (2025). "How to create a product requirements document (PRD)."  
[2] UXPin. (2025). "How to Structure AI-Assisted Development with PRDs."  
[3] Anthropic. (2024). "Building Effective AI Agents."  
[4] AWS. (2025). "Open-Sourcing Adaptive Workflows for AI-Driven Development Life Cycle."  
[5] Bessemer Venture Partners. (2025). "AI Agent Autonomy Scale."  
[6] Knight First Amendment Institute, Columbia University. (2025). "Levels of Autonomy for AI Agents."  
[7] arXiv. (2024). "Specification and Evaluation of Multi-Agent LLM Systems."  
[8] Amazon Science. (2025). "Structuring the Unstructured: A Multi-Agent LLM Framework."  
[9] Augment Code. (2025). "Spec-Driven Development & AI Agents Explained."  
[10] ChatPRD. (2025). "Best Practices for Using PRDs with Cursor."  
[11] LinkedIn. (2025). "AI for Smarter Requirements Traceability."  
[12] arXiv. (2025). "Structured Uncertainty Guided Clarification for LLM Agents."  
[13] Relevance AI. (2023). "What is a Multi Agent System."  
[14] Diggibyte. (2025). "AI Agent Safety: Managing Hallucination Risks & Guardrails."  
[15] Leanware. (2025). "AI Guardrails: Strategies, Mechanisms & Best Practices."  
[16] NVIDIA Technical Blog. (2025). "Agentic Autonomy Levels and Security."  

---

**Report compiled**: December 4, 2025  
**Version**: 1.0  
**Status**: Final
