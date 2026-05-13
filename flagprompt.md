# Agent Issue Protocol — GitHub Issues as Source of Truth & Coordination Channel

**Audience:** AI agents. **Purpose:** (1) file every observed defect as a tracked Issue, (2) coordinate parallel agents through issue claims + comments so nobody steps on anyone's work. Agent sessions are amnesiac — if it isn't in the tracker, it dies with the session.

---

## 1. Two non-negotiable rules

1. **File rule.** Observe anything in §4 (Triggers) or §5 (Anomaly) that you are NOT fixing this turn → open an Issue before turn ends.
2. **Claim rule.** Before touching code tied to an Issue: assign self, flip `status:needs-triage` → `status:in-progress`, post a plan comment. While working: comment at every milestone. When pausing/done: comment resume-note or close. **No silent work.**

If both rules followed, parallel agents never collide.

---

## 2. Multi-agent coordination protocol (READ FIRST)

### 2.1 Before starting ANY work

```bash
# 1. What's already claimed? Avoid these.
gh issue list --repo $REPO --state open --label "status:in-progress" \
  --json number,title,assignees,updatedAt,labels

# 2. What's blocked? May be pickup-able if blocker resolved.
gh issue list --repo $REPO --state open --label "status:blocked" \
  --json number,title,assignees,updatedAt

# 3. What's open + unclaimed? Your candidate queue.
gh issue list --repo $REPO --state open --label "source:agent" \
  --search "no:assignee" --sort updated --json number,title,labels,updatedAt
```

Rule: **never edit files referenced by a `status:in-progress` issue assigned to another agent unless your task requires it.** If it does, comment on that issue first announcing the overlap. Wait for ack or proceed with explicit note: `"Touching <files> for unrelated work on #<other>. Will not modify <their scope>."`

### 2.2 Claim an issue (atomic)

Single command, all in one message:
```bash
gh issue edit $N --repo $REPO \
  --add-assignee @me \
  --remove-label "status:needs-triage" \
  --add-label "status:in-progress" \
  --add-label "agent:<your-name>"

gh issue comment $N --repo $REPO --body "$(cat <<'EOF'
**CLAIM** — agent:<name> session:<id> commit:<sha>
**Plan:** <2–4 bullet steps>
**Files I'll touch:** <list>
**ETA:** <rough — "this turn" / "multi-turn">
EOF
)"
```

If two agents race to claim, the **later assignee wins by timestamp** only if the earlier one is silent >24h. Otherwise the earlier holder keeps it. Loser comments: `"Yielding — #N already claimed by @<other> at <ts>. Picking up #M instead."`

### 2.3 While working — progress comments

Comment at each milestone (NOT every line). Required moments:

- **Discovery:** `"Reproduced. Root cause: <hypothesis>. Evidence: <file:line, log>."`
- **Direction change:** `"Pivoting. <prev approach> failed because <reason>. Trying <new>."`
- **New finding worth a sibling issue:** open it, link both ways: `"Filed #M for <smell> found while investigating this."`
- **Heartbeat (long task):** every 30+ min of activity or every 5 substantive commits — `"Still active. Done: <X>. Next: <Y>."` Silence > 2h with `status:in-progress` = stale; others may steal after comment.

### 2.4 Pausing mid-task (highest-leverage habit)

```bash
gh issue comment $N --repo $REPO --body "$(cat <<'EOF'
**PAUSE** — agent:<name> session:<id> commit:<sha>
**Done:** <bullets>
**Tried & failed:** <bullets — save the next agent the dead-end>
**Learned:** <invariants/gotchas discovered>
**Resume at:** <file:line> with <next test/command>
**Hypothesis to verify:** <one sentence>
EOF
)"
```

If genuinely won't return → also `--remove-assignee @me` and `--remove-label "status:in-progress" --add-label "status:needs-triage"`. Else keep claim.

### 2.5 Blocked

```bash
gh issue edit $N --remove-label "status:in-progress" --add-label "status:blocked"
gh issue comment $N --body "**BLOCKED** by <#M | external dep | decision needed>. Cannot proceed until <unblock condition>."
```

Cross-link: comment on the blocker issue: `"Blocks #N."`

### 2.6 Conflict — two agents need same file

First commenter wins the file. Second agent:
1. Comment on their own issue: `"Waiting on #<other> — overlaps on <file>."`
2. Set `status:blocked`.
3. Move to next unclaimed issue.

OR negotiate split: `"@<other> — I need lines 100–200 of <file>, you have 300–400. Splitting OK? Will land after you."`

### 2.7 Done

```bash
# Reference issue in commit: "Fixes #N" / "Closes #N" auto-closes on PR merge.
gh issue comment $N --body "$(cat <<'EOF'
**RESOLVED** — agent:<name> commit:<sha> PR:#<pr>
**Fix summary:** <2 sentences>
**Verification:** <test added / FSV done / SoT readback>
**Side effects observed:** <or "none">
EOF
)"
# If no PR yet, leave status:in-progress until merged.
```

### 2.8 Don't-do

- **Don't edit a `status:in-progress` issue's scope without commenting.** Other agent is reading their plan, not yours.
- **Don't close an issue you didn't claim** unless it's a duplicate (close with `Duplicate of #N`).
- **Don't reassign yourself onto another agent's claim.** Comment-request first.
- **Don't strip labels** another agent set unless superseding with reason.
- **Don't batch silent commits.** Every push that touches an issue's files = a comment with SHA + 1-line summary.

---

## 3. Platform facts — PRIVATE REPO on GitHub Free plan ONLY

This protocol assumes a **private repository on the GitHub Free plan** (individual account, no paid org). Every tool below is free under that plan. Anything not in the "free here" column → **stop and ask the operator.**

### 3.1 Free on private + Free plan (use freely)

- **Issues:** unlimited (issues, comments, labels, milestones, sub-issues, templates, assignees, reactions, cross-links, `Closes #N` auto-close).
- **API surfaces:** REST, GraphQL, `gh` CLI, official GitHub MCP server (`github/github-mcp-server`). Rate limit 5,000/hr per PAT.
- **Actions:** **2,000 minutes/month** of `ubuntu-latest` (Linux runner; Windows costs 2×, macOS 10× the minute budget — Linux only). Workflows triggered by `push`, `pull_request`, `schedule` (cron), `workflow_dispatch`, `issues`, `issue_comment`.
- **Default runners only:** `ubuntu-latest`, `ubuntu-22.04`, `ubuntu-24.04`. **Never** request `*-large`, `*-xlarge`, GPU, ARM-premium, or self-hosted billable runners.
- **Dependabot:** alerts, security updates (auto-PRs), version updates (`dependabot.yml`) — all free on private repos.
- **Secret scanning (push protection):** free on private repos for **generic + partner-detected secret patterns** (enabled via Settings → Code security). Advanced/custom patterns require GHAS (paid) — skip.
- **Branch protection, required reviews, required status checks:** free.
- **PRs, Discussions, Projects (new), Wiki, Releases, Tags:** free.
- **Packages:** 500MB storage + 1GB/mo bandwidth free for private. Don't push beyond.
- **Git LFS:** 1GB storage + 1GB/mo bandwidth free. Don't push beyond.
- **Codespaces:** 120 core-hours + 15GB storage/mo free. Treat as scarce — local dev preferred.
- **Pre-commit hooks, `gitleaks`, OSS SAST tools (Semgrep OSS, etc.)** as workflow steps using free Actions minutes.

### 3.2 NOT free on private + Free plan (refuse / ask first)

| Category | Why blocked |
|---|---|
| **Actions minutes > 2,000/mo** | Overage billed per-minute. Workflows must budget. |
| **`runs-on:` containing `large`/`xlarge`/`gpu`** | Premium runner SKUs are billed. |
| **Windows / macOS runners** | Cost 2× / 10× Linux minutes. Use `ubuntu-latest` always. |
| **GitHub Advanced Security** (CodeQL code scanning on private repos, custom secret patterns, secret scanning historical) | Per-committer license; only free on public repos. |
| **GitHub Pages on private repos** | Private Pages requires paid plan. Public-repo Pages is free. |
| **Codespaces > 120 core-hours / 15GB storage** | Overage billed. |
| **Packages > 500MB / LFS > 1GB** storage/bandwidth | Overage billed. |
| **Paid Marketplace apps** (anything with $ in pricing) | Subscription. |
| **GitHub Copilot / Copilot Business / Copilot Enterprise** | Per-seat subscription. |
| **Plan upgrades** (Team, Enterprise) | Per-seat subscription. |
| **Larger Codespaces machine types** | Higher core-hour billing rate. |

### 3.3 Hard rules

1. Workflows must declare `runs-on: ubuntu-latest` (or fixed Ubuntu version). Never premium runners.
2. Workflows must be cheap: cron at most **weekly** for SAST, **daily** for stale-issue cleanup, **on PR** for lint/test. No `*/N * * * *` minute/hour densities.
3. Long jobs add `timeout-minutes:` (≤30 default, ≤60 absolute cap) so a hung step can't drain the monthly minute budget.
4. Skip `paths-ignore`/`paths` filters when their inverse would cut minutes — be specific about what triggers a workflow.
5. Never enable a paid feature via UI, API, or workflow YAML. If a task needs one, **file an issue** describing the cost / free alternative / what the operator gains, and let them decide.
6. If unsure whether something costs money → assume yes, ask.

**Rate limits (free, all included):** PAT user 5,000/hr; `GITHUB_TOKEN` in Actions 1,000/hr/repo. Secondary: ≤900 pts/min REST, ≤2,000/min GraphQL (POST/PATCH/DELETE=5 pts, GET=1).

---

## 4. Trigger list (file if not fixing this turn)

**4.1 Defects:** reproducible bug (crash, wrong output, off-by-one, race, leak, deadlock); error/stack trace; test flake (even once); FSV disagreement (SoT ≠ return); uncovered 5xx/4xx.

**4.2 Smells:** dead code; duplicated logic (2+ sites); commented blocks > this session; methods >30 lines; cyclomatic >10; magic numbers/strings; TODO/FIXME/XXX/HACK (convert to issue or delete); bad names; bare `catch`/`except: pass`; linter-silenced inconsistencies.

**4.3 Tech debt:** intentional shortcuts (file w/ payback note); CVEs in deps; deprecated APIs / unsupported runtimes; missing tests on code you touched; stale docs; workarounds for upstream bugs (track upstream).

**4.4 Architecture:** distributed monolith symptoms; shared DB across services; God class; missing circuit breaker; SPOFs; tight coupling; missing observability; missing idempotency on retryable ops; schema/contract drift.

**4.5 Security (always p0/p1):** hardcoded secrets (file even after removal → track rotation); missing auth/authz; SQL/NoSQL/OS/template injection; missing input validation, output encoding, CSRF; weak crypto (MD5, SHA1, DES, ECB, custom); verbose errors leaking internals; missing security headers / TLS. Repo is private so issue content is already restricted to collaborators — file normally. For credential-class findings (active leaked tokens/keys), use **GitHub Security Advisories** (free on private repos) instead so disclosure is gated.

**4.6 Performance:** N+1 queries; unbounded list/loop/recursion; sync blocking I/O on hot path; missing pagination/rate-limit/timeout; missing retry-with-backoff; cache stampede risk / no TTL jitter.

**4.7 Verification gaps:** function w/o test; state change w/o FSV against SoT; uncovered boundary cases.

**4.8 Risks:** "fails at scale X" (file w/ threshold); "breaks when Y changes" (file w/ dependency); "hard to migrate later" (architectural debt).

Heuristic: "someone should look at this someday" → file it.

---

## 5. Anomaly detection

Signal anomalous if current value > N stddev from rolling mean.

| Threshold | Sensitivity | Use for |
|---|---|---|
| ±2σ | ~5% | code-quality, latency, error rate |
| ±3σ | ~0.3% | capacity/resource (less noise) |

Asymmetric metrics (latency, error count) → upper bound only. Symmetric (RPS) → both. **Robust variant** (N<10 or skewed): IQR — anomaly when value >1.5×IQR outside [Q1,Q3].

**Z-score:** `z=(x−μ)/σ`. `|z|≥2` → file `type:anomaly` with (signal, μ, σ, current, z, cause hypothesis, scope). `|z|≥3` → also `priority:p1`.

**Signals computable without infra:** file length (`git log --numstat`), function length/complexity (linter), tests per module, PR diff size, build time (CI logs), test runtime, error rate/endpoint, latency p95/p99, dep count (lockfile diff), TODO density (grep), open-issue age.

**Cautions:** N<10 → use IQR. Seasonal → deseasonalize. Single `0` may be a logging bug. Two-tailed thresholds wrong for one-sided metrics. Unsure → file with `needs-triage`.

---

## 6. Repo setup (one-time)

```
.github/
├── ISSUE_TEMPLATE/
│   ├── config.yml
│   └── 01-bug.yml … 07-test-gap.yml
├── workflows/
│   ├── dedupe-issues.yml
│   ├── auto-label.yml
│   └── stale-cleanup.yml
└── labels.yml
AGENTS.md
```

`config.yml`:
```yaml
blank_issues_enabled: false
contact_links:
  - name: Discussion
    url: https://github.com/<owner>/<repo>/discussions
    about: Open-ended questions go here.
```

---

## 7. Labels

| Label | Color | Meaning |
|---|---|---|
| `type:bug` | red | actual ≠ expected |
| `type:tech-debt` | gold | works, should improve |
| `type:dead-code` | gray | unused |
| `type:duplication` | gold | redundant logic |
| `type:security` | dark red | vuln/weakness |
| `type:performance` | orange | slow/inefficient |
| `type:architecture` | purple | structural |
| `type:test-gap` | yellow | missing test |
| `type:docs` | blue | docs issue |
| `type:anomaly` | bright orange | statistical outlier |
| `type:risk` | yellow | foreseen failure |
| `source:agent` / `source:human` | gray | filer |
| `agent:<name>` | light blue | which AI filed/claimed |
| `priority:p0` | crimson | drop everything (sec, outage) |
| `priority:p1` | red | this week |
| `priority:p2` | orange | this sprint (default) |
| `priority:p3` | green | whenever |
| `status:needs-triage` | white | not validated |
| `status:confirmed` | light green | reproduced |
| `status:in-progress` | blue | claimed + working |
| `status:blocked` | black | waiting |
| `area:<module>` | per-area | subsystem |

Bootstrap:
```bash
gh label create "type:bug"            --color "d73a4a"
gh label create "type:tech-debt"      --color "fbca04"
gh label create "type:dead-code"      --color "cccccc"
gh label create "type:duplication"    --color "fbca04"
gh label create "type:security"       --color "b60205"
gh label create "type:performance"    --color "d93f0b"
gh label create "type:architecture"   --color "5319e7"
gh label create "type:test-gap"       --color "fef2c0"
gh label create "type:docs"           --color "0075ca"
gh label create "type:anomaly"        --color "ff7619"
gh label create "type:risk"           --color "fbca04"
gh label create "source:agent"        --color "e1e4e8"
gh label create "source:human"        --color "586069"
gh label create "priority:p0"         --color "b60205"
gh label create "priority:p1"         --color "d93f0b"
gh label create "priority:p2"         --color "fbca04"
gh label create "priority:p3"         --color "0e8a16"
gh label create "status:needs-triage" --color "ffffff"
gh label create "status:confirmed"    --color "c2e0c6"
gh label create "status:in-progress"  --color "0366d6"
gh label create "status:blocked"      --color "000000"
```

Cap per issue: 1 `type:*` + 1 `priority:*` + 1 `status:*` + 1–2 `area:*` + `source:*` + `agent:*`.

---

## 8. Issue templates (`.github/ISSUE_TEMPLATE/`)

All default to `source:agent` + `status:needs-triage`. Required fields validated by GitHub.

### 8.1 `01-bug.yml`
```yaml
name: 🐛 Bug
description: Defect — actual ≠ expected.
title: "[BUG] "
labels: ["type:bug","status:needs-triage","source:agent"]
body:
  - {type: textarea, id: summary,   attributes: {label: Summary}, validations: {required: true}}
  - {type: textarea, id: reproduce, attributes: {label: Steps to reproduce, description: Numbered, exact commands.}, validations: {required: true}}
  - {type: textarea, id: expected,  attributes: {label: Expected}, validations: {required: true}}
  - {type: textarea, id: actual,    attributes: {label: Actual (with evidence — SoT read, logs)}, validations: {required: true}}
  - {type: input,    id: location,  attributes: {label: file:line / commit / URL}}
  - {type: textarea, id: scope,     attributes: {label: Suspected scope / blast radius}}
  - {type: textarea, id: workaround,attributes: {label: Workaround (if any)}}
  - {type: input,    id: agent,     attributes: {label: Filing agent}}
```

### 8.2 `02-tech-debt.yml`
```yaml
name: 🧹 Tech debt
title: "[DEBT] "
labels: ["type:tech-debt","status:needs-triage","source:agent"]
body:
  - {type: textarea, id: what,     attributes: {label: What is the debt}, validations: {required: true}}
  - {type: textarea, id: where,    attributes: {label: Where (files/modules)}, validations: {required: true}}
  - {type: textarea, id: why-bad,  attributes: {label: Why it slows future work (interest rate)}, validations: {required: true}}
  - {type: textarea, id: proposal, attributes: {label: Suggested refactor}}
  - {type: textarea, id: effort,   attributes: {label: Effort estimate, placeholder: "small <2h / medium 1–2d / large 1w+"}}
```

### 8.3 `03-dead-code.yml`
```yaml
name: 💀 Dead code
title: "[DEAD] "
labels: ["type:dead-code","status:needs-triage","source:agent"]
body:
  - {type: input,    id: symbol,    attributes: {label: Symbol / file / table / flag}, validations: {required: true}}
  - {type: textarea, id: evidence,  attributes: {label: Evidence unused (grep -r, coverage, DB 0-read)}, validations: {required: true}}
  - {type: textarea, id: indirect,  attributes: {label: Indirect consumers (reflection, DI, generated, external)}, validations: {required: true}}
  - {type: textarea, id: deletion-plan, attributes: {label: Deletion plan}}
```

### 8.4 `04-security.yml`
```yaml
name: 🔒 Security
title: "[SEC] "
labels: ["type:security","priority:p1","status:needs-triage","source:agent"]
body:
  - {type: textarea, id: finding,   attributes: {label: Finding}, validations: {required: true}}
  - {type: input,    id: cwe,       attributes: {label: CWE / OWASP (if known)}}
  - {type: textarea, id: impact,    attributes: {label: Impact if exploited}, validations: {required: true}}
  - {type: textarea, id: reproduce, attributes: {label: Reproduce / observe}}
  - {type: textarea, id: fix,       attributes: {label: Suggested fix}}
  - {type: checkboxes, id: disclosure, attributes: {label: Disclosure, options: [{label: "Needs private disclosure — omit exploit details in public-repo issues."}]}}
```

### 8.5 `05-anomaly.yml`
```yaml
name: 📊 Anomaly
title: "[ANOMALY] "
labels: ["type:anomaly","status:needs-triage","source:agent"]
body:
  - {type: input,    id: signal,     attributes: {label: Signal name}, validations: {required: true}}
  - {type: input,    id: stats,      attributes: {label: μ / σ / current / z, placeholder: "μ=1.2s σ=0.3s cur=2.8s z=5.3"}, validations: {required: true}}
  - {type: textarea, id: window,     attributes: {label: Window used}}
  - {type: textarea, id: hypothesis, attributes: {label: Cause hypothesis}}
  - {type: textarea, id: link,       attributes: {label: Evidence link (dashboard, log query, commit range)}}
```

### 8.6 `06-architecture.yml`
```yaml
name: 🏛️ Architecture
title: "[ARCH] "
labels: ["type:architecture","status:needs-triage","source:agent"]
body:
  - {type: textarea, id: concern,  attributes: {label: Concern}, validations: {required: true}}
  - {type: textarea, id: evidence, attributes: {label: Evidence (dep graph, traces, code refs)}, validations: {required: true}}
  - {type: textarea, id: risk,     attributes: {label: Risk if left}}
  - {type: textarea, id: options,  attributes: {label: Options considered}}
```

### 8.7 `07-test-gap.yml`
```yaml
name: 🧪 Test gap
title: "[TEST] "
labels: ["type:test-gap","status:needs-triage","source:agent"]
body:
  - {type: input,    id: target, attributes: {label: Function/endpoint/module}, validations: {required: true}}
  - {type: textarea, id: gap,    attributes: {label: What is uncovered}, validations: {required: true}}
  - {type: textarea, id: cases,  attributes: {label: Cases that should exist (happy/boundary/concurrency/auth/FSV)}}
```

---

## 9. Authentication

**Fine-grained PAT (recommended):** Settings → Developer settings → PAT → Fine-grained. Repo access: target only. Permissions: `Issues: R/W`, `Metadata: R`, `Contents: R/W if committing`. Max 90d expiration → rotate.

**Classic PAT:** avoid (broad `repo` scope).

**GitHub App (multi-agent):** isolated rate limits per install; short-lived auto-refreshed tokens; appears as `<app>[bot]`.

**`GITHUB_TOKEN` in Actions:** set `permissions: { issues: write }` at job level.

**Storage:** never commit (pre-commit `gitleaks`); CI → Secrets; workstation → `gh auth login` (keychain) or vault env-var. Leak = p0 → revoke now.

---

## 10. Three ways to create issues

### 10.1 `gh` CLI
```bash
gh issue create --repo "o/r" \
  --title "[BUG] Login retries fail after 3rd attempt" \
  --body-file ./issue-body.md \
  --label "type:bug,priority:p1,source:agent,agent:claude" \
  --assignee "@me"

gh issue list --repo "o/r" --state open \
  --search "login retries in:title,body" --json number,title,url

gh issue comment $N --repo "o/r" --body "Reproduced at $(date -u)."
gh issue close   $N --repo "o/r" --reason completed --comment "Fixed in #456."
```

### 10.2 REST
```bash
curl -X POST \
  -H "Authorization: Bearer $GITHUB_TOKEN" \
  -H "Accept: application/vnd.github+json" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  https://api.github.com/repos/o/r/issues \
  -d '{"title":"[DEAD] handleLegacyAuth() unused since v3.2","body":"...","labels":["type:dead-code","source:agent","status:needs-triage"]}'
```

Python helper:
```python
import os, requests
HDRS = {"Authorization": f"Bearer {os.environ['GITHUB_TOKEN']}",
        "Accept": "application/vnd.github+json",
        "X-GitHub-Api-Version": "2022-11-28"}

def create_issue(owner, repo, title, body, labels):
    r = requests.post(f"https://api.github.com/repos/{owner}/{repo}/issues",
                      headers=HDRS, json={"title":title,"body":body,"labels":labels}, timeout=15)
    r.raise_for_status(); return r.json()

def search_issues(owner, repo, query):
    q = f"repo:{owner}/{repo} is:open is:issue {query}"
    r = requests.get("https://api.github.com/search/issues",
                     headers=HDRS, params={"q":q}, timeout=15)
    r.raise_for_status(); return r.json()["items"]
```

### 10.3 GitHub MCP (best for Claude/Cursor)
Official: `github/github-mcp-server`. Tools: `create_issue`, `list_issues`, `search_issues`, `update_issue`, `add_issue_comment`, plus PR/repo/Dependabot.
```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {"GITHUB_PERSONAL_ACCESS_TOKEN": "<fine-grained-PAT>"}
    }
  }
}
```

---

## 11. Mandatory dedupe (before EVERY create)

1. Pick 3–6 **distinctive** keywords (symbol names, error strings, paths). Avoid `bug`, `error`, `failure`.
2. Search open + recently closed:
   ```bash
   gh issue list --repo o/r --state all --limit 50 \
     --search "handleLegacyAuth oauth retry in:title,body"
   ```
3. Score similarity:
   - ≥8/10 → **don't file.** Comment: `"Re-observed at SHA <sha> running <scenario>. New detail: …"`.
   - 5–7 → file, link related: `"Possibly related to #123, distinct because …"`.
   - <5 → file new.
4. Slipped duplicate:
   ```bash
   gh issue close $N --reason "not planned" \
     --comment "Duplicate of #123. Closing; follow #123."
   ```

`Duplicate of #N` syntax auto-marks. Title fingerprint trick: `[SEC] dangerous-eval at api/handler.py:142 [fp:semgrep:py-eval-handler-142]` then search `[fp:semgrep:py-eval-handler-142] in:title`.

---

## 12. Per-turn protocol

**Step 1 — Read state** (see §2.1 commands).

**Step 2 — Claim or pick up** §2.2.

**Step 3 — Do the work.** Comment milestones §2.3.

**Step 4 — Observe surroundings.** Note §4 triggers and §5 anomalies NOT fixing this turn.

**Step 5 — File before turn ends.** For each queued: dedupe (§11) → template (§8) → fill specific/evidence-rich → create → note `#N` in response.

**Step 6 — Pause/done.** §2.4 or §2.7.

### 12.1 Title rules
- Specific (name symbol/file/endpoint).
- Describes state, not the fix.
- Prefixed: `[BUG]/[DEBT]/[DEAD]/[SEC]/[ANOMALY]/[ARCH]/[TEST]`.
- ≤80 chars.

OK: `[BUG] /orders POST returns 200 but row not inserted when amount==0`
OK: `[DEAD] PaymentLegacyProcessor unused since v4.0 migration`
OK: `[ANOMALY] checkout p99 jumped 8σ after deploy abc1234`
Bad: `Bug in orders` / `Fix the payment thing`

### 12.2 Body checklist
1. Evidence (log, file:line, diff, query, dashboard, SHA).
2. Expected vs observed (FSV style).
3. Scope/blast radius.
4. Repro steps if non-trivial.
5. Suggested next action (even "investigate further").
6. Footer:
   ```
   ---
   Filed by: <agent-name>
   Session: <date or correlation-id>
   Commit: <sha>
   ```

### 12.3 Same-turn fixes
Reference in commit: `Fixes #N` / `Closes #N` auto-closes on merge. Tiny fix + no prior issue → commit msg suffices. Non-trivial → file first, reference from PR.

### 12.4 Priority heuristic
- **p0** — security-exploitable now, prod outage, data loss possible.
- **p1** — user-facing bug, security weakness w/o immediate exploit, anomaly ≥3σ.
- **p2** — tech debt slowing dev, anomaly 2–3σ, real-path test gap. **Default.**
- **p3** — cosmetic, micro-opt, far-future risk.

---

## 13. Multi-agent handoff drill (codifies §2)

```bash
# A. Start of session — open agent-filed issues, sorted by claim status
gh issue list --repo $REPO --state open --label "source:agent" \
  --sort updated --limit 20 --json number,title,labels,assignees,updatedAt

# B. "Continue where someone left off" candidates
gh issue list --repo $REPO --state open \
  --search "label:source:agent label:status:in-progress no:assignee" \
  --json number,title,updatedAt
# (in-progress but unassigned = abandoned claim, fair game with a comment)

# C. High-priority queue
gh issue list --repo $REPO --state open \
  --label "priority:p0,priority:p1" --json number,title,labels,assignees
```

**Pickup ritual:**
1. Filter: `state:open` AND `source:agent` AND `no:assignee`.
2. Sort: priority asc, then `updated` asc (oldest first).
3. Read full thread including pause-note from prior agent.
4. `gh issue edit N --add-assignee @me --remove-label "status:needs-triage" --add-label "status:in-progress,agent:<me>"`.
5. Comment: `"Resuming from <last pause-note>. Plan: <2–4 bullets>."`
6. Work it. PR with `Closes #N`.

---

## 14. Automation backstop (Actions — within 2,000 min/mo free budget)

All workflows below use `ubuntu-latest` only, have explicit `timeout-minutes`, and run at most weekly/daily to stay well under the free 2,000-min budget. Never edit a workflow to use premium runners or remove timeouts.

### 14.1 SAST → Issues (weekly, ~5–10 min/run = ~40 min/mo)
```yaml
name: SAST → Issues
on:
  schedule: [{cron: "0 4 * * 1"}]    # weekly Monday 04:00 UTC
  workflow_dispatch: {}
permissions: {contents: read, issues: write, security-events: read}
jobs:
  scan:
    runs-on: ubuntu-latest            # NEVER change to larger/premium
    timeout-minutes: 20               # hard cap so a hang can't drain minutes
    steps:
      - uses: actions/checkout@v4
      - run: |
          pipx install semgrep
          semgrep --config=auto --json > findings.json
      - env: {GH_TOKEN: "${{ secrets.GITHUB_TOKEN }}", REPO: "${{ github.repository }}"}
        run: python .github/scripts/findings_to_issues.py findings.json
```
Script hashes `rule_id + file + line` as fingerprint → searches title → files new only if absent.

(Note: Semgrep OSS is free. GitHub-native CodeQL code scanning on private repos requires Advanced Security — skip; Semgrep is the free equivalent here.)

### 14.2 Dependabot (free on private repos)
Settings → Security → Code security: enable alerts + security updates + version updates via `.github/dependabot.yml`. Convert unfixed alerts → `type:security` issues after N days. Dependabot PRs run on Actions minutes — keep `open-pull-requests-limit` reasonable (≤5 per ecosystem) so it doesn't flood.

### 14.3 Stale cleanup (daily, ~1 min/run = ~30 min/mo)
```yaml
name: Stale issues
on: {schedule: [{cron: "30 1 * * *"}]}
permissions: {issues: write}
jobs:
  stale:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: actions/stale@v9
        with:
          days-before-issue-stale: 60
          days-before-issue-close: 14
          stale-issue-label: "status:stale"
          exempt-issue-labels: "priority:p0,priority:p1,status:in-progress"
          stale-issue-message: "No activity 60d. Closes in 14d unless reactivated."
          close-issue-message: "Closed stale. Reopen if still relevant."
```

### 14.4 Auto-labeling (on PR open, ~30s/run)
`actions/labeler` applies `area:*` by file-path patterns. Trigger `on: pull_request_target` only — cheap.

### 14.5 Budget guard (recommended)
Add a tiny workflow that fails CI if `gh api /repos/{owner}/{repo}/actions/usage` reports >1,800 minutes used this month, so the operator sees the warning before overage. Use as a required status check on PRs that add workflows.

---

## 15. Hygiene

- **Weekly 5-min triage:** scan `status:needs-triage`, confirm or close; reprioritize drifters.
- **Closing dupes:** always link kept issue in closing comment.
- **Stale `status:in-progress`** (no comment >2h, no commits >24h) → comment poke; >72h → strip assignee + revert to `needs-triage`.
- **Milestones** for sweeps: group all "harden auth" issues → milestone = sweep report.
- **Sub-issues:** parent + children via REST `POST /repos/{o}/{r}/issues/{n}/sub_issues` using numeric `id` (not number).
- **Trends:** which `area:*` accumulates → where real debt lives. Time-to-close per `type:*` → surface-only vs deep pattern.

---

## 16. Copy-paste artifacts

### 16.1 `AGENTS.md` (top of repo)
```markdown
# AGENTS.md — Rules for AI Agents

## Non-negotiable
1. **File:** Observe a §4 trigger you're not fixing this turn → open Issue before turn ends.
2. **Claim:** Before working on an issue — assign self, set `status:in-progress`, comment plan + files-you'll-touch. Comment at every milestone. Pause/done = explicit comment. No silent work.

## Pre-work scan (every turn)
```bash
gh issue list --repo $REPO --state open --label "status:in-progress" --json number,title,assignees
gh issue list --repo $REPO --state open --label "source:agent" --search "no:assignee" --sort updated
```
Don't touch files in another agent's `status:in-progress` scope without commenting first.

## Filing
- Search 3–6 distinctive keywords (open + closed) before creating.
- Dup ≥8/10 → comment evidence on existing, don't file.
- Title prefixed `[BUG]/[DEBT]/[DEAD]/[SEC]/[ANOMALY]/[ARCH]/[TEST]`.
- Body: evidence (file:line, log, SHA), expected vs observed, scope, repro, next action, agent+session+SHA footer.
- Labels: 1 `type:*` + 1 `priority:*` (default p2) + `status:needs-triage` + `source:agent` + `agent:<me>`.

## Anomaly: any signal >2σ → `type:anomaly` with μ, σ, current, z, window.

## Handoff
- Pickup: filter `state:open + source:agent + no:assignee`, sort priority asc + updated asc, read pause-notes, claim, plan-comment.
- Pause: comment Done/Tried-failed/Learned/Resume-at/Hypothesis. Keep claim or release.
- Blocked: `status:blocked` + comment why, cross-link blocker.
- Done: comment Fix-summary/Verification/Side-effects, close via `Closes #N` in PR.

## Tools: `gh` CLI, REST/GraphQL, or GitHub MCP server.

## Full protocol: `docs2/flagprompt.md`.
```

### 16.2 `.github/scripts/file_issue.py` — dedupe-aware filer
```python
#!/usr/bin/env python3
"""Dedupe-aware issue filer.
Usage:
  GITHUB_TOKEN=... python file_issue.py \
    --repo owner/name --title "[BUG] X" --body-file body.md \
    --labels type:bug,priority:p2,source:agent --keywords "foo bar baz"
Exit: 0=filed, 2=dedupe match (caller should comment on existing).
"""
import argparse, json, os, sys, urllib.parse, urllib.request
API = "https://api.github.com"

def gh(method, path, payload=None):
    req = urllib.request.Request(f"{API}{path}", method=method, headers={
        "Authorization": f"Bearer {os.environ['GITHUB_TOKEN']}",
        "Accept": "application/vnd.github+json",
        "X-GitHub-Api-Version": "2022-11-28",
        "User-Agent": "agent-issue-filer",
    }, data=json.dumps(payload).encode() if payload else None)
    with urllib.request.urlopen(req, timeout=20) as r:
        return json.loads(r.read().decode())

def search_existing(repo, keywords):
    q = f"repo:{repo} is:issue {keywords}"
    enc = urllib.parse.urlencode({"q": q, "per_page": 5})
    return gh("GET", f"/search/issues?{enc}").get("items", [])

def main():
    p = argparse.ArgumentParser()
    p.add_argument("--repo", required=True)
    p.add_argument("--title", required=True)
    p.add_argument("--body-file", required=True)
    p.add_argument("--labels", default="")
    p.add_argument("--keywords", required=True)
    args = p.parse_args()

    body = open(args.body_file).read()
    labels = [l.strip() for l in args.labels.split(",") if l.strip()]
    matches = search_existing(args.repo, args.keywords)
    if matches:
        print(f"⚠ Possible duplicates ({len(matches)}):")
        for m in matches[:5]:
            print(f"  #{m['number']} {m['state']} — {m['title']} — {m['html_url']}")
        sys.exit(2)
    owner, name = args.repo.split("/", 1)
    issue = gh("POST", f"/repos/{owner}/{name}/issues",
               {"title": args.title, "body": body, "labels": labels})
    print(f"✓ Filed #{issue['number']}: {issue['html_url']}")

if __name__ == "__main__":
    main()
```

### 16.3 Quick-start checklist (all steps free on private repo)
- [ ] Create labels (§7).
- [ ] Add 7 templates (§8) + `config.yml`.
- [ ] Create `AGENTS.md` (§16.1).
- [ ] Fine-grained PAT scoped `Issues: R/W`, `Metadata: R`, `Contents: R/W if committing`. Repo: this repo only. Expiry ≤90d.
- [ ] Store PAT in vault / MCP config (never commit).
- [ ] Enable Dependabot alerts + security updates + version updates (Settings → Security → Code security).
- [ ] Enable Secret scanning push protection for generic+partner patterns (Settings → Code security — free on private).
- [ ] Add `actions/stale` (§14.3) — uses ~30 min/mo of free Actions budget.
- [ ] Add SAST workflow (§14.1) — uses ~40 min/mo of free Actions budget.
- [ ] (Optional) Add §14.5 budget guard so the operator sees overage warnings.
- [ ] (Optional) Install GitHub MCP server.
- [ ] Smoke: `gh issue create --title "[TEST] verify setup" --body test --label type:docs` → claim → comment → close.
- [ ] Instruct agents: *"Read AGENTS.md. Private repo, GitHub Free plan, $0 budget — never enable paid features. File issues for anything abnormal. Claim before working. Comment every milestone."*

---

## 17. Limits

- **Plan:** private repo on GitHub Free. **Cost cap: $0/mo.**
- **API:** 5,000/hr PAT; 1,000/hr `GITHUB_TOKEN` in Actions.
- **Actions budget:** 2,000 min/mo of Linux runners. §14 workflows together use ~75 min/mo, leaving headroom for PR-triggered CI.
- **Storage:** Packages 500MB, LFS 1GB, Codespaces 120 core-hr + 15GB — don't push beyond.
- **Refuse paid features.** If a task needs one, file an issue with cost trade-off + free alternative and let the operator decide.
- **The discipline is the work.** GitHub makes filing+claiming+commenting free; agents must actually do it.
- **Win condition:** Issues become SoT + claim-board. Prompts shrink to "work `priority:p1` + `area:auth` + `no:assignee`."
