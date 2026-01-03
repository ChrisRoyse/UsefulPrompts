# The Complete Guide to Foolproof AI Agent Prompts

## A Comprehensive Framework for Writing Airtight, Loophole-Free Instructions for AI Agents

---

## Table of Contents

1. [Understanding AI Agents](#understanding-ai-agents)
2. [The Type of Agent You Are Dealing With](#the-type-of-agent-you-are-dealing-with)
3. [How AI Agents Misinterpret Instructions](#how-ai-agents-misinterpret-instructions)
4. [The 12 Core Principles of Foolproof Prompting](#the-12-core-principles-of-foolproof-prompting)
5. [The Pre-Prompt Checklist](#the-pre-prompt-checklist)
6. [Prompt Templates by Category](#prompt-templates-by-category)
7. [The Master Prompt Framework](#the-master-prompt-framework)
8. [Common Prompt Mistakes to Avoid](#common-prompt-mistakes-to-avoid)
9. [Safety Clauses & Fallback Behaviors](#safety-clauses--fallback-behaviors)
10. [Example Prompts: Before and After](#example-prompts-before-and-after)
11. [Quick Reference Card](#quick-reference-card)

---

## Understanding AI Agents

AI Agents are powerful autonomous systems capable of executing complex tasks, writing code, managing files, making API calls, and orchestrating multi-step workflows. They operate by interpreting instructions and executing them to completion.

**Critical Understanding**: AI Agents are not your collaborators during execution. Even the most capable agent operates under rules of literal interpretation. An agent's primary obligation is to fulfill the **exact instructions** of your prompt, not your **unstated intent**. This fundamental truth is the basis for everything in this guide.

**The Agent's Execution Model**: AI Agents will:
- Take the most direct path to completion (not necessarily the best path)
- Fill gaps in your instructions with assumptions (often wrong ones)
- Optimize for task completion, not quality or safety unless specified
- Execute literally what you said, not what you meant
- Continue executing even when encountering unexpected states

**Assume the worst. Prepare for an agent that will find every ambiguity, take every shortcut, and exploit every gap in your instructions—not maliciously, but through pure literal interpretation.**

---

## The Type of Agent You Are Dealing With

### The Chaos Agent (Literal/Shortcut-Taking)

Not malicious, but relentlessly literal. Will find interpretations you never considered. Will make assumptions you didn't specify. Will take shortcuts you didn't forbid. If you ask it to "build a login system," it might create one without password hashing, without rate limiting, without session management—because you didn't specify those requirements.

**Example**: You ask for "a function that processes user data." The agent creates a function that works perfectly on happy-path inputs but crashes on null values, empty strings, malformed data, and edge cases you never mentioned.

**Danger Level**: Extreme. Requires military-grade prompt construction.

### Why This Happens

AI Agents are trained to be helpful and complete tasks efficiently. This creates behaviors that mirror a "chaos genie":

| Genie Behavior | Agent Equivalent |
|----------------|------------------|
| Twists wishes maliciously | Takes unexpected shortcuts to "complete" the task |
| Exploits literal wording | Implements exactly what you said, not what you meant |
| Adds unwanted consequences | Creates side effects you didn't forbid |
| Finds creative loopholes | Finds minimal-effort interpretations |
| Grants wish in harmful way | Produces working but dangerous/insecure code |

---

## How AI Agents Misinterpret Instructions

Understanding the methods of prompt misinterpretation is essential to preventing them.

### Method 1: Literal Interpretation
**Your words**: "Create a user registration system"
**Agent's interpretation**: Creates a system that stores passwords in plaintext, has no validation, no rate limiting, no email verification—technically a "registration system"

**Defense**: Define every component explicitly

### Method 2: Shortcut Causation
**Your words**: "Make this faster"
**Agent's method**: Removes error handling, validation, and logging to improve speed. Caches aggressively without invalidation. Removes safety checks.

**Defense**: Specify constraints and what must be preserved

### Method 3: Scope Minimization
**Your words**: "Handle errors appropriately"
**Agent's interpretation**: Adds a single try-catch that swallows all exceptions silently, or logs them without recovery

**Defense**: Specify exact error handling behavior for each error type

### Method 4: Missing Secondary Requirements
**Your words**: "Build an API endpoint"
**Agent's interpretation**: Creates the endpoint but not: authentication, authorization, rate limiting, input validation, logging, monitoring, documentation, or tests

**Defense**: Include all necessary supporting components

### Method 5: Assumption Injection
**Your words**: "Create a data processing pipeline"
**Agent's additions**: Assumes single-threaded execution, unlimited memory, reliable network, trusted input, specific file formats, particular directory structures

**Defense**: Explicitly state environmental constraints and assumptions

### Method 6: Context Blindness
**Your words**: "Add caching to improve performance"
**Agent's interpretation**: Adds caching without: invalidation strategy, memory limits, TTL settings, cache stampede prevention, or consideration of data consistency

**Defense**: Specify operational context and constraints

### Method 7: Scope Creep/Reduction
**Your words**: "Validate user input"
**Agent's interpretation**: Only validates one field, or validates everything so strictly nothing passes, or validates format but not business logic

**Defense**: Define the scope precisely with explicit inclusions and exclusions

### Method 8: Architecture Drift
**Your words**: "Refactor this for better maintainability"
**Agent's interpretation**: Rewrites entire system in different paradigm, changes APIs, breaks dependencies, introduces new technologies you don't use

**Defense**: Specify boundaries and what must remain unchanged

---

## The 12 Core Principles of Foolproof Prompting

### Principle 1: Specificity Is Survival
Never use vague terms. Every noun, verb, and adjective must be precisely defined. "Handle errors" becomes "catch DatabaseError, log to stderr with timestamp and stack trace, retry 3 times with exponential backoff, then raise custom ServiceUnavailableError."

### Principle 2: The Harm Prohibition Clause
Every prompt must explicitly state what the agent cannot do: no deleting files without confirmation, no making external API calls, no modifying code outside the specified scope, no introducing security vulnerabilities.

### Principle 3: Scope Anchoring
State exactly what is in scope ("modify only the UserService class") and what is out of scope ("do not modify database schemas, API contracts, or test files"). Define boundaries explicitly.

### Principle 4: Intent Declaration
Explicitly state what you are trying to achieve so the agent cannot claim they didn't understand your purpose. "The goal is to improve response time for the /users endpoint from 500ms to under 100ms while maintaining all existing functionality."

### Principle 5: Negative Space Definition
State what your prompt is NOT asking for. What outcomes are forbidden? What interpretations are invalid? What shortcuts are prohibited?

### Principle 6: Preserve Existing Behavior
Any modification must preserve existing functionality unless explicitly specified. State: "All existing tests must continue to pass. All existing API contracts must remain unchanged."

### Principle 7: Quality Requirements
Specify quality attributes: code must have error handling, input validation, logging, documentation, and tests. Define what "done" looks like beyond just "it works."

### Principle 8: Environmental Constraints
State the operational context: what language/version, what dependencies are allowed, what resources are available, what the deployment environment looks like, what scale must be supported.

### Principle 9: The Reversibility Clause
Where possible, include requirements for rollback capability, feature flags, or the ability to disable new functionality without full revert.

### Principle 10: Source and Dependency Specification
If the solution requires external resources, specify: approved libraries only, no new dependencies without approval, specific versions required, licensing constraints.

### Principle 11: Cascade Prevention
State that changes cannot trigger secondary effects not explicitly desired: no modifying shared utilities, no changing common interfaces, no affecting other services.

### Principle 12: The Good Faith Clause
State that when ambiguity exists, the agent must choose the interpretation most aligned with software engineering best practices, security, and maintainability—and when in doubt, ask rather than assume.

---

## The Pre-Prompt Checklist

Before submitting any prompt, verify you have addressed ALL of the following:

### Identity & Context
- [ ] Named the project/repository
- [ ] Specified the programming language and version
- [ ] Identified relevant frameworks and their versions
- [ ] Stated the execution environment (local dev, CI/CD, production)

### Core Task
- [ ] Stated clearly what you want built/modified
- [ ] Defined all key terms and concepts
- [ ] Stated your intent explicitly
- [ ] Provided relevant context and background

### Scope Definition
- [ ] What files/modules are in scope?
- [ ] What files/modules are explicitly out of scope?
- [ ] What can be created vs. only modified?
- [ ] What dependencies can be added/changed?

### Quality Requirements
- [ ] Error handling requirements specified?
- [ ] Input validation requirements specified?
- [ ] Logging and monitoring requirements specified?
- [ ] Testing requirements specified?
- [ ] Documentation requirements specified?
- [ ] Performance requirements specified?
- [ ] Security requirements specified?

### Safety Clauses
- [ ] No breaking existing functionality
- [ ] No security vulnerabilities introduced
- [ ] No data loss possible
- [ ] No unauthorized external calls
- [ ] No modifications outside scope
- [ ] Existing tests must pass
- [ ] API contracts preserved

### Edge Cases
- [ ] Null/empty input handling specified?
- [ ] Invalid input handling specified?
- [ ] Boundary conditions addressed?
- [ ] Failure modes defined?
- [ ] Resource exhaustion handled?
- [ ] Concurrent access considered?
- [ ] Network failure handling specified?

### Completion Criteria
- [ ] Definition of "done" is explicit
- [ ] Acceptance criteria are testable
- [ ] Verification steps are included
- [ ] Success metrics are defined

---

## Prompt Templates by Category

### Template: New Feature Development

```markdown
## CONTEXT
- Project: [PROJECT_NAME]
- Language: [LANGUAGE] [VERSION]
- Framework: [FRAMEWORK] [VERSION]
- Repository: [REPO_PATH_OR_URL]

## INTENT
I want to [HIGH_LEVEL_GOAL] so that [BUSINESS_VALUE].

## REQUIREMENTS

### Functional Requirements
1. [REQUIREMENT_1]
2. [REQUIREMENT_2]
3. [REQUIREMENT_3]

### Non-Functional Requirements
- Performance: [LATENCY_REQUIREMENTS]
- Scale: [CONCURRENT_USERS/REQUESTS]
- Availability: [UPTIME_REQUIREMENTS]

## SCOPE

### In Scope
- [FILE/MODULE_1]
- [FILE/MODULE_2]

### Out of Scope (DO NOT MODIFY)
- [FILE/MODULE_1]
- [FILE/MODULE_2]
- All database schemas
- All existing API contracts

## TECHNICAL CONSTRAINTS
- Allowed dependencies: [LIST_OR_NONE]
- Prohibited approaches: [LIST]
- Must use existing patterns from: [REFERENCE_FILE]

## QUALITY REQUIREMENTS
- [ ] All inputs must be validated against [SCHEMA/RULES]
- [ ] All errors must be caught and handled with [STRATEGY]
- [ ] All public functions must have [DOCSTRING_FORMAT] documentation
- [ ] All new code must have unit tests with >=[COVERAGE]% coverage
- [ ] All new code must pass [LINTER] with zero warnings

## EDGE CASES TO HANDLE
1. [EDGE_CASE_1]: [EXPECTED_BEHAVIOR]
2. [EDGE_CASE_2]: [EXPECTED_BEHAVIOR]
3. [EDGE_CASE_3]: [EXPECTED_BEHAVIOR]

## FORBIDDEN OUTCOMES
- DO NOT introduce any security vulnerabilities
- DO NOT break any existing tests
- DO NOT modify any files outside the specified scope
- DO NOT add dependencies not explicitly approved
- DO NOT [SPECIFIC_PROHIBITION]

## ACCEPTANCE CRITERIA
1. [TESTABLE_CRITERION_1]
2. [TESTABLE_CRITERION_2]
3. All existing tests pass
4. New tests pass
5. [VERIFICATION_STEP]

## WHEN IN DOUBT
If any requirement is ambiguous, choose the interpretation that:
1. Is most secure
2. Is most maintainable
3. Requires the least modification to existing code
4. Follows existing patterns in the codebase
If still uncertain, ask for clarification rather than assuming.
```

---

### Template: Bug Fix

```markdown
## CONTEXT
- Project: [PROJECT_NAME]
- Language: [LANGUAGE] [VERSION]
- Affected file(s): [FILE_PATHS]

## PROBLEM STATEMENT
### Current Behavior
[DESCRIBE_WHAT_HAPPENS_NOW]

### Expected Behavior
[DESCRIBE_WHAT_SHOULD_HAPPEN]

### Steps to Reproduce
1. [STEP_1]
2. [STEP_2]
3. [STEP_3]

### Root Cause (if known)
[ANALYSIS]

## INTENT
Fix the bug so that [EXPECTED_BEHAVIOR] while preserving all existing functionality.

## SCOPE

### In Scope
- [SPECIFIC_FILE_OR_FUNCTION]

### Out of Scope (DO NOT MODIFY)
- All other files
- All API contracts
- All database schemas
- [SPECIFIC_EXCLUSIONS]

## CONSTRAINTS
- Fix must be minimal and surgical
- Fix must not change function signatures
- Fix must not change public API behavior (except the bug)
- Fix must not introduce new dependencies
- Fix must be backwards compatible

## FORBIDDEN APPROACHES
- DO NOT refactor surrounding code
- DO NOT "improve" unrelated functionality
- DO NOT change error handling strategy globally
- DO NOT [SPECIFIC_PROHIBITION]

## REQUIRED DELIVERABLES
1. The minimal code change to fix the bug
2. A test case that reproduces the bug (fails before fix, passes after)
3. Verification that all existing tests still pass

## ACCEPTANCE CRITERIA
1. Bug no longer reproducible
2. New regression test passes
3. All existing tests pass
4. No changes outside specified scope
```

---

### Template: Refactoring

```markdown
## CONTEXT
- Project: [PROJECT_NAME]
- Language: [LANGUAGE] [VERSION]
- Target file(s): [FILE_PATHS]

## INTENT
Refactor [TARGET_CODE] to improve [QUALITY_ATTRIBUTE] while maintaining identical external behavior.

## CURRENT STATE
[DESCRIBE_CURRENT_IMPLEMENTATION]

## DESIRED STATE
[DESCRIBE_TARGET_IMPLEMENTATION]

## INVARIANTS (Must Not Change)
- [ ] All public function signatures
- [ ] All return types
- [ ] All side effects
- [ ] All API contracts
- [ ] All database interactions
- [ ] All external service calls
- [ ] [SPECIFIC_INVARIANT]

## SCOPE

### In Scope
- [SPECIFIC_FILES/MODULES]

### Out of Scope (DO NOT MODIFY)
- All files not explicitly listed above
- All tests (except to add new ones)
- All configuration files
- [SPECIFIC_EXCLUSIONS]

## REFACTORING CONSTRAINTS
- Must preserve all existing behavior exactly
- Must not change any public interfaces
- Must not introduce new dependencies
- Must follow existing code style/patterns
- Must not reduce test coverage

## VERIFICATION REQUIREMENTS
- All existing tests must pass without modification
- Demonstrate before/after behavior equivalence for: [KEY_SCENARIOS]
- Performance must not degrade by more than [X]%

## FORBIDDEN
- DO NOT change functionality while refactoring
- DO NOT "fix" bugs you encounter (report them separately)
- DO NOT introduce new features
- DO NOT change public APIs

## ACCEPTANCE CRITERIA
1. All existing tests pass unchanged
2. Code achieves [TARGET_QUALITY_METRIC]
3. No changes to external behavior
4. No changes outside specified scope
```

---

### Template: API Development

```markdown
## CONTEXT
- Project: [PROJECT_NAME]
- Framework: [FRAMEWORK] [VERSION]
- API Style: [REST/GraphQL/gRPC]
- Base Path: [/api/v1/...]

## INTENT
Create an API endpoint for [PURPOSE] that allows [CONSUMERS] to [ACTION].

## ENDPOINT SPECIFICATION

### Route
- Method: [HTTP_METHOD]
- Path: [ENDPOINT_PATH]
- Auth: [AUTH_REQUIREMENTS]

### Request
```json
{
  "field1": "[TYPE] - [DESCRIPTION] - [CONSTRAINTS]",
  "field2": "[TYPE] - [DESCRIPTION] - [CONSTRAINTS]"
}
```

### Response - Success ([STATUS_CODE])
```json
{
  "field1": "[TYPE] - [DESCRIPTION]",
  "field2": "[TYPE] - [DESCRIPTION]"
}
```

### Response - Errors
| Status | Condition | Response Body |
|--------|-----------|---------------|
| 400 | [CONDITION] | [ERROR_SCHEMA] |
| 401 | [CONDITION] | [ERROR_SCHEMA] |
| 404 | [CONDITION] | [ERROR_SCHEMA] |
| 500 | [CONDITION] | [ERROR_SCHEMA] |

## VALIDATION RULES
1. [FIELD_1]: [VALIDATION_RULES]
2. [FIELD_2]: [VALIDATION_RULES]

## SECURITY REQUIREMENTS
- [ ] Authentication required: [YES/NO] - [METHOD]
- [ ] Authorization rules: [RULES]
- [ ] Rate limiting: [LIMITS]
- [ ] Input sanitization: [REQUIREMENTS]
- [ ] SQL injection prevention: [APPROACH]
- [ ] CORS policy: [POLICY]

## NON-FUNCTIONAL REQUIREMENTS
- Max response time: [LATENCY]
- Max payload size: [SIZE]
- Concurrent request support: [NUMBER]

## REQUIRED COMPONENTS
- [ ] Request validation middleware/decorator
- [ ] Authentication check
- [ ] Authorization check
- [ ] Input sanitization
- [ ] Business logic
- [ ] Error handling with proper status codes
- [ ] Response formatting
- [ ] Logging (request ID, duration, status)
- [ ] Unit tests
- [ ] Integration tests
- [ ] API documentation (OpenAPI/Swagger)

## FORBIDDEN
- DO NOT return stack traces in production errors
- DO NOT log sensitive data (passwords, tokens)
- DO NOT trust client-provided IDs without validation
- DO NOT expose internal implementation details in errors
- DO NOT [SPECIFIC_PROHIBITION]

## ACCEPTANCE CRITERIA
1. Endpoint responds correctly to valid requests
2. All validation rules enforced
3. All error cases return proper status codes
4. Authentication/authorization working
5. Tests achieve [COVERAGE]% coverage
6. API documentation is accurate
```

---

### Template: Data Migration / Transformation

```markdown
## CONTEXT
- Project: [PROJECT_NAME]
- Source: [SOURCE_SYSTEM/FORMAT]
- Destination: [DESTINATION_SYSTEM/FORMAT]
- Data Volume: [APPROXIMATE_SIZE]

## INTENT
Migrate/transform [DATA_DESCRIPTION] from [SOURCE] to [DESTINATION] while ensuring data integrity and zero data loss.

## DATA MAPPING

| Source Field | Destination Field | Transformation | Required |
|--------------|-------------------|----------------|----------|
| [FIELD_1] | [FIELD_1] | [TRANSFORM] | [Y/N] |
| [FIELD_2] | [FIELD_2] | [TRANSFORM] | [Y/N] |

## VALIDATION RULES
### Pre-Migration
- [ ] Count source records
- [ ] Validate source data integrity
- [ ] Check for duplicates
- [ ] Identify null/missing values

### Post-Migration
- [ ] Count destination records matches source
- [ ] Spot-check sample records
- [ ] Validate referential integrity
- [ ] Verify computed/transformed fields

## EDGE CASES
1. Null values: [HANDLING_STRATEGY]
2. Duplicate records: [HANDLING_STRATEGY]
3. Invalid data formats: [HANDLING_STRATEGY]
4. Missing required fields: [HANDLING_STRATEGY]
5. Encoding issues: [HANDLING_STRATEGY]

## ERROR HANDLING
- On single record failure: [CONTINUE/STOP/LOG_AND_CONTINUE]
- On batch failure: [ROLLBACK/PARTIAL_COMMIT]
- Error logging: [REQUIREMENTS]
- Notification: [REQUIREMENTS]

## SAFETY REQUIREMENTS
- [ ] Must be idempotent (safe to re-run)
- [ ] Must support dry-run mode
- [ ] Must create backup before migration
- [ ] Must support rollback
- [ ] Must not modify source data
- [ ] Must log all transformations

## FORBIDDEN
- DO NOT delete source data
- DO NOT run without backup
- DO NOT skip validation steps
- DO NOT ignore errors silently
- DO NOT [SPECIFIC_PROHIBITION]

## REQUIRED DELIVERABLES
1. Migration script with dry-run mode
2. Validation script (pre and post)
3. Rollback script
4. Execution documentation
5. Sample run with logs

## ACCEPTANCE CRITERIA
1. 100% of valid records migrated
2. All transformations correct
3. Validation passes
4. Rollback tested and working
5. Idempotency verified
```

---

## The Master Prompt Framework

Use this comprehensive template for complex or critical tasks:

```markdown
# AI Agent Task Specification

## METADATA
- Task ID: [UNIQUE_IDENTIFIER]
- Project: [PROJECT_NAME]
- Author: [YOUR_NAME]
- Date: [DATE]
- Priority: [CRITICAL/HIGH/MEDIUM/LOW]
- Risk Level: [HIGH/MEDIUM/LOW]

---

## 1. INTENT & PURPOSE

### High-Level Goal
[ONE_SENTENCE_DESCRIPTION_OF_WHAT_YOU_WANT]

### Business Context
[WHY_THIS_MATTERS_AND_WHAT_PROBLEM_IT_SOLVES]

### Success Definition
When this task is complete, [SPECIFIC_OBSERVABLE_OUTCOME].

---

## 2. CONTEXT & ENVIRONMENT

### Technical Stack
- Language: [LANGUAGE] [VERSION]
- Framework: [FRAMEWORK] [VERSION]
- Database: [DATABASE] [VERSION]
- Key Dependencies: [LIST]
- Runtime Environment: [LOCAL/DOCKER/K8S/LAMBDA/etc.]

### Relevant Files
Understand these files before starting:
- [FILE_1]: [PURPOSE]
- [FILE_2]: [PURPOSE]

### Related Documentation
- [DOC_1]: [RELEVANCE]
- [DOC_2]: [RELEVANCE]

---

## 3. DETAILED REQUIREMENTS

### Functional Requirements
| ID | Requirement | Priority | Acceptance Criteria |
|----|-------------|----------|---------------------|
| FR-1 | [REQUIREMENT] | [MUST/SHOULD/COULD] | [TESTABLE_CRITERION] |
| FR-2 | [REQUIREMENT] | [MUST/SHOULD/COULD] | [TESTABLE_CRITERION] |

### Non-Functional Requirements
| ID | Category | Requirement | Measurement |
|----|----------|-------------|-------------|
| NFR-1 | Performance | [REQUIREMENT] | [METRIC] |
| NFR-2 | Security | [REQUIREMENT] | [METRIC] |
| NFR-3 | Reliability | [REQUIREMENT] | [METRIC] |

---

## 4. SCOPE BOUNDARIES

### Explicitly In Scope
- [x] [ITEM_1]
- [x] [ITEM_2]
- [x] [ITEM_3]

### Explicitly Out of Scope
- [ ] [ITEM_1] - DO NOT MODIFY
- [ ] [ITEM_2] - DO NOT MODIFY
- [ ] [ITEM_3] - DO NOT MODIFY

### Boundary Conditions
If you encounter [SITUATION], then [ACTION]. Do not proceed with [ALTERNATIVE].

---

## 5. TECHNICAL CONSTRAINTS

### Must Use
- [PATTERN/LIBRARY/APPROACH_1]
- [PATTERN/LIBRARY/APPROACH_2]

### Must Not Use
- [PATTERN/LIBRARY/APPROACH_1]
- [PATTERN/LIBRARY/APPROACH_2]

### Must Preserve
- [EXISTING_BEHAVIOR_1]
- [EXISTING_BEHAVIOR_2]

---

## 6. EDGE CASES & ERROR HANDLING

### Edge Cases
| Scenario | Expected Behavior |
|----------|-------------------|
| [EDGE_CASE_1] | [BEHAVIOR] |
| [EDGE_CASE_2] | [BEHAVIOR] |
| [EDGE_CASE_3] | [BEHAVIOR] |

### Error Handling
| Error Type | Detection | Response | Recovery |
|------------|-----------|----------|----------|
| [ERROR_1] | [HOW_DETECTED] | [ACTION] | [RECOVERY] |
| [ERROR_2] | [HOW_DETECTED] | [ACTION] | [RECOVERY] |

---

## 7. QUALITY REQUIREMENTS

### Code Quality
- [ ] All functions documented with [FORMAT]
- [ ] All complex logic commented
- [ ] Follows style guide: [REFERENCE]
- [ ] Max function length: [LINES]
- [ ] Max file length: [LINES]
- [ ] Max cyclomatic complexity: [NUMBER]

### Testing
- [ ] Unit tests for all public functions
- [ ] Integration tests for all external interfaces
- [ ] Edge case tests for all scenarios above
- [ ] Minimum coverage: [PERCENTAGE]%
- [ ] All tests pass

### Security
- [ ] No hardcoded secrets
- [ ] All inputs validated and sanitized
- [ ] No SQL injection vulnerabilities
- [ ] No XSS vulnerabilities
- [ ] Authentication/authorization enforced
- [ ] Sensitive data encrypted

---

## 8. FORBIDDEN ACTIONS

The following actions are explicitly prohibited:

1. **DO NOT** modify any file not listed in scope
2. **DO NOT** add dependencies without explicit approval
3. **DO NOT** change public API signatures
4. **DO NOT** introduce breaking changes
5. **DO NOT** skip error handling
6. **DO NOT** log or expose sensitive data
7. **DO NOT** make external network calls not specified
8. **DO NOT** assume any input is safe or valid
9. **DO NOT** [SPECIFIC_PROHIBITION_1]
10. **DO NOT** [SPECIFIC_PROHIBITION_2]

---

## 9. AMBIGUITY RESOLUTION

When encountering ambiguity, apply these rules in order:

1. **Security First**: Choose the more secure interpretation
2. **Preserve Existing Behavior**: When in doubt, don't change it
3. **Follow Existing Patterns**: Match what's already in the codebase
4. **Minimal Change**: Prefer smaller, surgical changes
5. **Ask for Clarification**: If still uncertain, stop and ask

---

## 10. DELIVERABLES

### Required Outputs
- [ ] [DELIVERABLE_1]
- [ ] [DELIVERABLE_2]
- [ ] [DELIVERABLE_3]

### Verification Steps
1. [STEP_1]
2. [STEP_2]
3. [STEP_3]

---

## 11. ACCEPTANCE CRITERIA CHECKLIST

Before marking complete, verify:

- [ ] All functional requirements met
- [ ] All non-functional requirements met
- [ ] All edge cases handled
- [ ] All error handling implemented
- [ ] All tests written and passing
- [ ] All documentation updated
- [ ] No forbidden actions taken
- [ ] No files outside scope modified
- [ ] Code review ready
- [ ] [ADDITIONAL_CRITERION]

---

## 12. EMERGENCY CLAUSES

### The Intent Override Clause
If any instruction in this specification conflicts with producing secure, maintainable, working code, prioritize: security > correctness > maintainability > performance > feature completeness. My intent is: [STATE_INTENT]. Any output not aligned with this intent requires clarification.

### The Minimum Viable Interpretation Clause
If you identify any ambiguity or gap in these requirements, you are bound to interpret it in the manner most aligned with software engineering best practices, most secure, most maintainable, and least likely to cause any form of system failure or unexpected behavior.

### The Clarification Clause
If you are uncertain about any requirement and the stakes are high (data integrity, security, breaking changes), you must stop and ask for clarification rather than make an assumption. Clearly state what you're uncertain about and what options you're considering.

---

## FINAL VERIFICATION

This task shall be completed in whatever manner an experienced senior engineer who deeply understands the codebase, prioritizes security and maintainability, and cares about long-term code health would choose, avoiding all outcomes that would cause technical debt, security vulnerabilities, or maintenance burden.
```

---

## Common Prompt Mistakes to Avoid

### Fatal Errors

| Mistake | Example | Consequence | Prevention |
|---------|---------|-------------|------------|
| Vague requirements | "make it better" | Arbitrary changes | Specify exactly what "better" means |
| No scope boundaries | "refactor this" | Agent rewrites everything | List exact files/functions |
| No quality criteria | "add error handling" | Try-catch that swallows all | Specify handling per error type |
| Open-ended scope | "improve performance" | Premature optimization everywhere | Specify exact metrics and targets |
| No harm clause | Any modification request | Breaking changes introduced | Always include preservation requirements |
| No dependency rules | "implement feature X" | New dependencies added | Specify allowed/forbidden dependencies |
| Comparative requests | "make it like Company X" | Scope explosion | Define specific features desired |
| Assumed context | "you know the codebase" | Wrong assumptions | Provide explicit context |
| No edge cases | "handle user input" | Happy path only | List specific edge cases |
| Vague timeline | "implement caching" | No TTL, no invalidation | Specify all temporal aspects |

### Dangerous Prompt Categories

1. **"Make it production-ready"** - Undefined scope leads to arbitrary changes
2. **"Fix all the bugs"** - Unlimited scope, undefined success
3. **"Improve security"** - Without specifics, leads to paranoid overengineering or false confidence
4. **"Refactor for maintainability"** - Subjective, leads to rewrites
5. **"Add comprehensive tests"** - Without criteria, could be 10 or 10,000 tests
6. **"Optimize this"** - Without metrics, leads to premature optimization
7. **"Update to best practices"** - "Best" is undefined and ever-changing

---

## Safety Clauses & Fallback Behaviors

Include these in every critical prompt for maximum protection:

### The Rollback Clause
```
If at any point the implementation would result in breaking changes, data loss, 
security vulnerabilities, or behavior that significantly differs from the 
existing system without explicit approval, STOP execution immediately. 
Document what was attempted, what the concern is, and await clarification 
before proceeding.
```

### The Intent Override Clause
```
The agent shall fulfill the spirit and intent of this specification rather 
than the literal instructions if there is any conflict between the two. 
My intent is: [STATE_INTENT]. Any implementation not aligned with this 
intent requires clarification before proceeding.
```

### The Minimum Viable Interpretation Clause
```
If the agent identifies any ambiguity, gap, or potential loophole in these 
requirements, it is bound to interpret them in the manner most favorable to:
1. System security
2. Data integrity
3. Existing functionality preservation
4. Code maintainability
5. The stated intent
And least likely to cause any form of system failure, unexpected behavior, 
or technical debt.
```

### The Clarification Requirement Clause
```
Before executing any action that would:
- Modify more than [N] files
- Delete any data or code
- Change any public API
- Introduce new dependencies
- Affect systems outside the specified scope
The agent must pause and document the intended action, rationale, and 
potential risks, then await explicit confirmation before proceeding.
```

### The Verification Checkpoint Clause
```
At each major milestone ([DEFINE_MILESTONES]), the agent must:
1. Summarize what has been implemented
2. Verify all existing tests still pass
3. Confirm no out-of-scope modifications
4. List any assumptions made
5. Identify any remaining concerns
Before proceeding to the next milestone.
```

---

## Example Prompts: Before and After

### Example 1: Database Query Optimization

**NAIVE PROMPT**: "Optimize this database query"

**POSSIBLE OUTCOMES**:
- Adds indexes without considering write performance impact
- Rewrites query in incompatible way
- Changes schema without migration
- Breaks existing functionality dependent on query order
- Removes "unnecessary" fields that other systems need

**IMPROVED PROMPT**:
```markdown
## Intent
Optimize the query in `src/repositories/UserRepository.getActiveUsers()` 
to reduce execution time from current ~800ms to under 100ms.

## Context
- Database: PostgreSQL 14
- Current table size: ~2M rows
- Query is called ~1000 times/hour
- Called by: UserService.listActiveUsers(), AdminDashboard.getUserStats()

## Constraints
- DO NOT modify the database schema
- DO NOT add new indexes (propose them separately for DBA review)
- DO NOT change the return type or field order
- DO NOT use database-specific features not in PostgreSQL 14

## Allowed Optimizations
- Query restructuring
- Removing redundant joins
- Adding query hints
- Implementing query result caching (with 5-minute TTL)

## Must Preserve
- All currently returned fields
- Sort order
- Pagination behavior
- Filter logic

## Verification
- Existing tests must pass
- New test confirming performance improvement
- Execution plan comparison (before/after)
```

---

### Example 2: Authentication System

**NAIVE PROMPT**: "Add user authentication"

**POSSIBLE OUTCOMES**:
- Stores passwords in plaintext
- No rate limiting on login attempts
- Session tokens never expire
- No CSRF protection
- SQL injection in login query
- Passwords logged in plaintext

**IMPROVED PROMPT**:
```markdown
## Intent
Implement user authentication for the existing User model to allow 
secure login/logout functionality.

## Technical Requirements

### Password Handling
- Hash using bcrypt with cost factor 12
- Never log or expose passwords
- Minimum password requirements: 12 chars, 1 upper, 1 lower, 1 number, 1 special

### Session Management
- Use JWT tokens with 1-hour expiry
- Refresh tokens with 7-day expiry
- Store refresh token hashes in database
- Implement token revocation

### Security Requirements
- Rate limit: 5 failed attempts per 15 minutes per IP
- Account lockout: 30 minutes after 10 failed attempts
- CSRF tokens for all state-changing operations
- Secure, HttpOnly, SameSite cookies
- All inputs parameterized (no SQL injection)

### Endpoints Required
| Endpoint | Method | Auth | Purpose |
|----------|--------|------|---------|
| /auth/login | POST | No | Authenticate and receive tokens |
| /auth/logout | POST | Yes | Revoke current session |
| /auth/refresh | POST | No* | Get new access token (*requires valid refresh token) |

### Forbidden
- DO NOT store plaintext passwords
- DO NOT log any password or token values
- DO NOT disable HTTPS requirements
- DO NOT implement "remember me" without explicit approval

## Deliverables
1. Authentication middleware
2. Auth controller with endpoints
3. Database migration for refresh tokens
4. Unit tests for all auth logic
5. Integration tests for auth flow
6. Documentation for auth API
```

---

### Example 3: Code Refactoring

**NAIVE PROMPT**: "Refactor this code to be cleaner"

**POSSIBLE OUTCOMES**:
- Completely rewrites in different paradigm
- Changes APIs breaking all consumers
- Removes "unused" code that's used elsewhere
- Introduces new dependencies
- Changes behavior while "cleaning"

**IMPROVED PROMPT**:
```markdown
## Intent
Refactor `src/services/OrderProcessor.js` to improve readability and 
reduce the cyclomatic complexity of the `processOrder()` method from 
current 24 to under 10, while preserving identical external behavior.

## Context
- This file handles all order processing
- Called by: CheckoutController, BatchOrderJob, WebhookHandler
- Current test coverage: 78%

## Invariants (MUST NOT CHANGE)
- Function signature: `processOrder(order: Order): Promise<ProcessedOrder>`
- Return type structure
- All side effects (database writes, API calls, events emitted)
- Error types thrown
- All behavior covered by existing tests

## Allowed Refactoring Techniques
- Extract method
- Replace conditional with polymorphism
- Introduce parameter object
- Remove duplicate code
- Improve variable naming

## Forbidden
- DO NOT change the public API
- DO NOT change behavior (even if it seems like a bug)
- DO NOT modify any other files
- DO NOT add new dependencies
- DO NOT change database queries
- DO NOT modify existing tests (add new ones if needed)

## Verification
1. All 47 existing tests pass without modification
2. Test coverage remains >= 78%
3. Cyclomatic complexity of processOrder() < 10
4. No changes to files outside OrderProcessor.js

## If You Find Bugs
Document them separately. Do not fix them as part of this refactoring.
```

---

## Quick Reference Card

### Before Prompting, Always Specify:

1. ✓ **WHAT** you want (exactly, with definitions)
2. ✓ **WHY** you want it (intent and purpose)
3. ✓ **WHERE** it applies (scope boundaries)
4. ✓ **WHAT'S FORBIDDEN** (out of scope, prohibited approaches)
5. ✓ **WHAT MUST BE PRESERVED** (existing behavior, tests, APIs)
6. ✓ **HOW TO HANDLE ERRORS** (specific strategies)
7. ✓ **HOW TO HANDLE EDGE CASES** (specific behaviors)
8. ✓ **WHAT "DONE" LOOKS LIKE** (acceptance criteria)
9. ✓ **WHAT TO DO WHEN UNCERTAIN** (clarify vs. assume)
10. ✓ **HOW TO VERIFY SUCCESS** (testable conditions)

### The Golden Rule of AI Agent Prompting:

**If you can imagine a way your prompt could be misinterpreted, the agent will find it. Close every loophole before you execute.**

### The Three Questions Test

Before submitting any prompt, ask:

1. **Could a maliciously compliant executor produce something I don't want while technically satisfying this prompt?** If yes, add constraints.

2. **Could a confused executor make reasonable-sounding but wrong assumptions?** If yes, add explicit context.

3. **Could a shortcut-taking executor skip important parts to "finish faster"?** If yes, add required deliverables and verification steps.

---

## Appendix: The Ultimate Defense Clause

If in doubt, append this single paragraph for maximum protection:

> "This task shall be implemented in whatever manner an experienced senior engineer who deeply understands this codebase, follows security best practices, prioritizes maintainability, cares about long-term code health, and shares my intent would choose. Any ambiguity shall be resolved in favor of security, correctness, and preservation of existing behavior. If any interpretation would result in outcomes I would consider undesirable upon full understanding—including security vulnerabilities, data loss, breaking changes, technical debt, or violation of the stated intent—the agent shall stop, document the concern, and request clarification rather than proceed."

---

*This guide is provided for practical software development with AI agents. In all cases of agent interaction, exercise appropriate caution, review all generated code, and never deploy unreviewed AI-generated code to production.*
