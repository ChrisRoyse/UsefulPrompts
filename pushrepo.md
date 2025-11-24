---
description: Check uncommited git history then push to repo.
---

## Primary Objective

Conduct a systematic code quality audit of all uncommitted changes in the git repository to identify and eliminate temporary workarounds, incomplete implementations, and anti-patterns that may mask system failures or impede proper error handling. The system must maintain a "fail-fast" philosophy with clear, actionable error messages and comprehensive logging.

---

## Investigation Scope & Methodology

### Phase 1: Git History Analysis

1. **Enumerate all uncommitted changes:**
   - Execute `git status` to identify modified, new, and deleted files
   - Run `git diff` to examine line-by-line changes in tracked files
   - Check for untracked files that may contain new code
   - Review staged vs unstaged changes separately
   - Document the full scope of changes before beginning analysis

2. **Change categorization:**
   - Group files by type (source code, configuration, tests, documentation)
   - Identify languages and frameworks in use
   - Note the functional domains affected (API, database, UI, business logic, etc.)
   - Create a structured inventory of what's being reviewed

---

### Phase 2: Anti-Pattern Detection

Systematically search for the following problematic patterns in ALL uncommitted files:

#### A. Mock/Stub Code in Production Paths

**What to find:**
- Functions returning hardcoded/dummy data instead of real implementations
- Mock objects or services instantiated outside of test contexts
- Stubbed API calls that return fake responses
- Database queries bypassed with static data
- Authentication/authorization checks that are commented out or return `true` unconditionally

**Examples to flag:**
```python
# PROBLEMATIC
def get_user_data(user_id):
    # TODO: implement real database query
    return {"id": 123, "name": "Test User"}

# PROBLEMATIC
if ENVIRONMENT == "production":
    return mock_payment_processor()  # TEMPORARY
```

**Acceptable exceptions:**
- Mocks explicitly in test files (`test_*.py`, `*.test.js`, `spec/`, etc.)
- Feature flags with clear documentation and tracking tickets
- Documented A/B testing stubs with expiration dates

---

#### B. Placeholder Implementations

**What to find:**
- Functions with `pass`, `return None`, `return {}`, `return []` as the only implementation
- Empty catch blocks that suppress errors silently
- TODO/FIXME/HACK comments indicating incomplete work
- Functions that always return success without performing operations
- Unimplemented interface methods or abstract class violations

**Examples to flag:**
```javascript
// PROBLEMATIC
async function processPayment(amount) {
    // TODO: integrate with payment gateway
    return { success: true };
}

// PROBLEMATIC
catch (error) {
    // will fix later
}

// PROBLEMATIC
def validate_input(data):
    pass  # validation coming soon
```

**Search patterns:**
- Comments containing: `TODO`, `FIXME`, `HACK`, `XXX`, `TEMP`, `TEMPORARY`, `PLACEHOLDER`, `WIP`, `PENDING`
- Empty function bodies or single-line returns of default values
- Commented-out blocks of functional code

---

#### C. Silent Failure & Error Suppression

**What to find:**
- Try-catch blocks that catch exceptions but don't log or re-throw
- Error handlers that return default values without alerting
- Functions that swallow errors and continue execution
- Missing error boundaries in UI components
- Validation that passes by default when checks fail
- Network/database operations without timeout or retry logic that fail silently

**Examples to flag:**
```java
// PROBLEMATIC
try {
    criticalDatabaseOperation();
} catch (Exception e) {
    return defaultValue;  // Error lost!
}

// PROBLEMATIC
if (!validateUserInput(data)) {
    // just continue anyway
    processData(data);
}
```

**Required instead:**
- Explicit error logging with context
- Error propagation to appropriate handlers
- User-facing error messages
- Telemetry/monitoring hooks

---

#### D. Fallback Logic Without Failure Signals

**What to find:**
- Fallback values used without logging why the primary path failed
- Default configurations applied silently when loading fails
- Graceful degradation that masks underlying problems
- Retry logic that exhausts attempts without alerting
- Cache misses that don't log or monitor

**Examples to flag:**
```python
# PROBLEMATIC
def get_config():
    try:
        return load_from_database()
    except:
        return DEFAULT_CONFIG  # Why did it fail? No one knows!

# PROBLEMATIC - Fallback masks the problem
data = fetch_from_api() or fetch_from_cache() or {}
```

**Required instead:**
- Log warnings when falling back
- Increment metrics/counters for monitoring
- Alert on repeated fallback usage
- Document expected vs unexpected fallback scenarios

---

#### E. Workarounds & Quick Fixes

**What to find:**
- Code that works around known bugs instead of fixing them
- Race condition "fixes" using arbitrary sleep/delays
- Hard-coded values that should be configurable
- Copied/duplicated code to avoid touching existing functions
- Type coercion or string manipulation to work around validation
- Comments indicating "temporary fix" or "will refactor later"

**Examples to flag:**
```javascript
// PROBLEMATIC
await new Promise(resolve => setTimeout(resolve, 1000)); // fixes race condition

// PROBLEMATIC
const result = parseFloat(stringValue) || 0; // handles bad data from API

// PROBLEMATIC
// Don't use getUserById, it's broken, use this instead
function getUserByIdFixed(id) { ... }
```

---

#### F. Incomplete Error Handling

**What to find:**
- Network requests without error callbacks
- Database operations without transaction rollback
- File operations without cleanup (missing `finally` blocks)
- Promise chains without `.catch()` handlers
- Async functions without proper error propagation
- Missing null/undefined checks on external data
- Array/object access without existence validation

**Examples to flag:**
```typescript
// PROBLEMATIC
const data = await fetch(url).then(r => r.json());
// No error handling!

// PROBLEMATIC
const userName = response.user.profile.name;
// No null checks!

// PROBLEMATIC
function processFile(path) {
    const file = openFile(path);
    processContent(file);
    // What if processContent throws? File never closes!
}
```

---

#### G. Insufficient Logging & Observability

**What to find:**
- Critical operations without logging (auth, payments, data mutations)
- Error conditions that don't log stack traces or context
- External API calls without request/response logging
- State transitions without audit trails
- Missing correlation IDs for request tracing
- Debug logging left in production code
- Sensitive data (passwords, tokens, PII) in logs

**Required presence:**
- Entry/exit logging for critical functions
- Structured logging with consistent formats
- Appropriate log levels (ERROR, WARN, INFO, DEBUG)
- Context information (user ID, request ID, timestamp)
- Performance metrics for slow operations

---

#### H. Configuration & Environment Issues

**What to find:**
- Hard-coded credentials, API keys, or secrets
- Environment-specific code without abstraction
- Missing environment variable validation
- Default values that wouldn't work in production
- Configuration files with `localhost` or development URLs
- Commented-out production configurations

**Examples to flag:**
```python
# PROBLEMATIC
API_KEY = "sk_test_12345"  # Hardcoded!

# PROBLEMATIC
if env == "production":
    db_host = "prod-db.company.com"
else:
    db_host = "localhost"  # Better to use env vars
```

---

### Phase 3: Validation & Quality Gates

For each issue found, assess:

1. **Severity Classification:**
   - **CRITICAL**: Security vulnerabilities, data loss risks, silent failures in production paths
   - **HIGH**: Missing error handling, incomplete implementations in core features
   - **MEDIUM**: Poor logging, workarounds that function but are fragile
   - **LOW**: Code style issues, minor technical debt

2. **Impact Analysis:**
   - Which components/modules are affected?
   - What user flows or business processes depend on this code?
   - What's the blast radius if this fails?
   - Is there existing test coverage that would catch issues?

3. **Root Cause Identification:**
   - Is this rushed implementation?
   - Is there a missing dependency or blocker?
   - Is this technical debt from earlier shortcuts?
   - Are there knowledge gaps in the team?

---

### Phase 4: Documentation & Reporting

Generate a comprehensive report with:

#### Structure:
```
## Code Quality Review Report
**Date**: [timestamp]
**Reviewer**: [AI model identifier]
**Scope**: [number] files, [number] lines changed

### Executive Summary
- Total issues found: [count]
- Critical: [count]
- High: [count]  
- Medium: [count]
- Low: [count]

### Detailed Findings

#### [Category Name]
**File**: `path/to/file.ext`
**Lines**: [start-end]
**Severity**: [CRITICAL/HIGH/MEDIUM/LOW]

**Issue Description**:
[Clear explanation of the problem]

**Current Code**:
```[language]
[problematic code snippet with context]
```

**Why This Is Problematic**:
[Specific risks and consequences]

**Recommended Fix**:
```[language]
[suggested corrected code]
```

**Acceptance Criteria**:
- [ ] Proper error handling implemented
- [ ] Logging added with appropriate level
- [ ] Tests added/updated
- [ ] Documentation updated

---

### Phase 5: Decision Matrix & Interactive Approval

Based on findings, execute the appropriate flow:

#### IF NO ISSUES FOUND:
```
✅ CODE QUALITY: APPROVED FOR COMMIT

All uncommitted changes have been reviewed and meet quality standards:
- No mock/placeholder code in production paths
- Proper error handling implemented throughout
- Comprehensive logging in place
- No workarounds or temporary fixes
- Fail-fast principles maintained

🔄 PROCEEDING WITH AUTOMATIC COMMIT...
```

**AUTOMATIC ACTIONS:**
1. Stage all changes: `git add -A`
2. Generate commit message based on changes
3. Create commit with the generated message
4. Display commit summary
5. Ask user if they want to push to remote

**Commit Message Format:**
```
[type]: [concise description]

- [detailed change 1]
- [detailed change 2]
- [detailed change 3]

[Optional: references to tickets/issues]
```

---

#### IF MINOR ISSUES FOUND (LOW/MEDIUM severity):
```
⚠️ CODE QUALITY: REVIEW RECOMMENDED

Minor issues found that should be addressed but don't block commit:
- [count] logging improvements needed
- [count] minor code quality issues

🛑 STOPPING FOR USER REVIEW
```

**PRESENT DETAILED ISSUE REPORT:**

For each issue found, display:
1. **File Path**: `path/to/file.ext:line_number`
2. **Severity**: LOW/MEDIUM
3. **Category**: [Pattern type from Phase 2]
4. **Issue Description**: Clear explanation of what's wrong
5. **Why It's Problematic**: Specific risks and consequences
6. **Recommended Fix**: Code snippet or approach to fix

**ASK USER FOR DECISION:**

Present options to the user using AskUserQuestion tool:
```
⚠️ Minor issues detected in uncommitted files. How would you like to proceed?

Options:
A) Fix issues now before committing (RECOMMENDED)
   - I will help you fix each issue
   - Then automatically commit when clean

B) Commit anyway with issue tracking
   - Add TODO comments to code
   - Create tracking tickets for issues
   - Commit with note about pending work

C) Show me the detailed report first
   - Display full analysis
   - Then ask again for decision

D) Cancel and fix manually
   - Stop the process
   - Let you fix issues yourself
```

**WAIT FOR USER RESPONSE BEFORE PROCEEDING**

---

#### IF MAJOR ISSUES FOUND (HIGH/CRITICAL severity):
```
❌ CODE QUALITY: CHANGES REQUIRED

Critical issues found that MUST be resolved before commit:
- [count] critical issues (security, data loss, silent failures)
- [count] high-priority issues (missing error handling, incomplete features)

🛑 COMMIT BLOCKED - STOPPING FOR USER REVIEW
```

**PRESENT CRITICAL ISSUE REPORT:**

For each critical/high issue found, display:
1. **File Path**: `path/to/file.ext:line_number`
2. **Severity**: CRITICAL/HIGH ⚠️
3. **Category**: [Pattern type from Phase 2]
4. **Issue Description**: Clear explanation of what's wrong
5. **Why It's Critical**: Specific risks (security, data loss, production failures)
6. **Code Snippet**: Show the problematic code with context
7. **Recommended Fix**: Detailed fix with code example
8. **Impact Analysis**: What breaks if this isn't fixed

**EXPLAIN WHY THESE ARE BLOCKING:**

Provide clear rationale:
- Security vulnerabilities could expose user data
- Silent failures could cause data loss
- Missing error handling could crash production
- Mock code in production paths will fail under load
- Incomplete implementations will break user workflows

**ASK USER FOR DECISION:**

Present options to the user using AskUserQuestion tool:
```
❌ Critical issues detected that block commit. What would you like to do?

Options:
A) Fix critical issues with my help (REQUIRED)
   - I will guide you through fixing each critical issue
   - Then re-run scan to verify fixes
   - Auto-commit when all critical issues resolved

B) Show me detailed fix recommendations
   - Display code snippets for each fix
   - Provide step-by-step remediation plan
   - Then ask again for decision

C) Fix some issues, defer others
   - Fix critical issues now
   - Create tickets for high-priority issues
   - Commit with explicit TODO tracking

D) Cancel commit entirely
   - Stop the process
   - Let you review and fix manually
   - Re-run /pushrepo when ready
```

**DO NOT PROCEED WITH COMMIT UNTIL:**
- All CRITICAL issues are resolved, OR
- User explicitly chooses to defer with tracking (Option C)

**WAIT FOR USER RESPONSE BEFORE ANY ACTION**

---

### Phase 6: Language-Agnostic Pattern Matching

Apply these universal checks regardless of language:

1. **Function/Method Analysis:**
   - Signature matches intended behavior
   - Return types are meaningful (not just null/void)
   - Parameters are validated
   - Edge cases are handled

2. **Control Flow:**
   - No unreachable code after early returns
   - All branches lead to valid states
   - Loops have proper termination conditions
   - Recursion has base cases

3. **Resource Management:**
   - Files, connections, locks are properly released
   - Memory allocations are tracked
   - Cleanup happens in all exit paths

4. **Data Validation:**
   - Input sanitization at boundaries
   - Type checking where languages allow
   - Range/bounds validation
   - Format verification

---

### Phase 7: Testing & Coverage Verification

Check for:

1. **Test Presence:**
   - Do new functions have corresponding tests?
   - Are edge cases tested?
   - Are error paths tested?

2. **Test Quality:**
   - Tests actually assert behavior (not just execution)
   - Tests use real implementations, not mocks (unless integration tests)
   - Tests cover happy path AND failure scenarios

3. **Missing Tests Flag:**
   - If production code changed without test changes, flag for review

---

## Execution Instructions

Execute this review in the following order:

1. **Start** by running `git status` and `git diff` to understand scope
2. **Scan** every modified file systematically (don't skip any)
3. **Document** findings in real-time with file paths and line numbers
4. **Classify** each issue by severity and category
5. **Aggregate** findings into the structured report format
6. **Decide** whether code is ready for commit using the decision matrix
7. **STOP and present findings if issues are found**
8. **ASK USER for approval** on how to proceed using AskUserQuestion tool
9. **WAIT for user response** before taking any commit action
10. **AUTO-COMMIT only if** no issues found OR user approves after review

---

## Success Criteria

This review is successful when:

- ✅ Every uncommitted file has been examined
- ✅ All patterns from Phase 2 have been checked
- ✅ A clear decision (approve/review/reject) has been made
- ✅ Specific, actionable feedback has been provided
- ✅ **If issues found**: Process STOPS and user is asked for approval
- ✅ **If issues found**: User receives detailed explanation of why each issue is problematic
- ✅ **If issues found**: User is presented with clear options via AskUserQuestion tool
- ✅ **If no issues**: Automatic commit proceeds with generated commit message
- ✅ If approved for commit: commit message is ready and commit is executed
- ✅ If rejected: exact changes needed are documented and process stops

---

## Output Format

Provide your response in this structure:

```markdown
# Code Review Results

## Scan Summary
- Files reviewed: X
- Total changes: Y lines
- Scan duration: Z seconds
- Issues found: [NONE / MINOR / CRITICAL]

## Findings Overview
[Summary table with counts by severity and category]

## Detailed Analysis
[Full findings with code snippets, explanations, and recommendations]

## Final Verdict
[✅ APPROVED - AUTO-COMMITTING / ⚠️ NEEDS REVIEW - STOPPING / ❌ BLOCKED - STOPPING]

---

### IF NO ISSUES (APPROVED):
```
✅ CODE QUALITY: CLEAN - PROCEEDING WITH COMMIT

Staging changes...
Creating commit with message:
---
[generated commit message]
---

Commit created: [commit hash]

Would you like to push to remote? (y/n)
```

### IF ISSUES FOUND (NEEDS REVIEW or BLOCKED):
```
🛑 STOPPING - USER APPROVAL REQUIRED

[Detailed issue report with:]
- File paths and line numbers
- Severity levels
- Issue descriptions
- Why each issue is problematic
- Recommended fixes

[Present options using AskUserQuestion tool]
[WAIT for user response]
```

## Next Steps
- **No issues**: Commit executed, ask about push
- **Issues found**: Awaiting user decision from presented options
- **After user response**: Execute chosen action (fix, defer, cancel)

## Appendix
- Git commands used
- Files examined
- Patterns searched
- Issues categorized by severity
```
