# Software System Hardening — AI Agent Reference

**Audience:** AI agent given the task "harden / improve / optimize this system."
**Promise:** every axis on which a system can be improved, every check to run, every primitive that exists, in a form an agent can grep, scan, and dispatch from.
**Reading mode:** this is a reference, not a tutorial. Jump to the axis you need via §0. Tables and lists are the load-bearing structure; prose is glue.
**Density rule:** acronyms are expanded once in §0.3, never again. Don't reflow as paragraphs — keep tables intact.

---

## 0. Index & conventions

### 0.1 The 10 hardening axes (jump table)

| # | Axis | Failure if neglected | Section |
|---|---|---|---|
| 1 | **Security** | breach, exfil, ransomware, regulatory fine | §3 |
| 2 | **Correctness / accuracy** | silent wrong answers, ghost regressions | §4 |
| 3 | **Performance** | slow UX, lost revenue, infra spend bloat | §5 |
| 4 | **Reliability (SRE)** | outages, missed SLAs, on-call burnout | §6 |
| 5 | **Resilience (fault tolerance)** | cascading failures, blast-radius incidents | §7 |
| 6 | **Scalability** | works at 1× breaks at 10×, costly rewrites | §8 |
| 7 | **Cost efficiency** | runaway cloud bill, no commitment leverage | §9 |
| 8 | **Architecture / maintainability** | velocity collapse, bus factor, dead code | §10–§11 |
| 9 | **Data layer** | N+1, lock contention, runaway storage | §12 |
| 10 | **Observability** | 3am triage = guesswork; SLOs unmeasurable | §13 |
| 11 | **Supply chain integrity** | typosquat, dep confusion, build tampering | §14 |
| 12 | **AI/ML/LLM-specific** | drift, hallucination, prompt injection, calibration loss | §15 |
| 13 | **Benchmarking discipline** | misleading wins, hidden regressions | §16 |
| 14 | **Operational practice** | undisciplined hardening, lost progress | §17–§20 |

Skip none. Each fails differently; their controls don't substitute.

### 0.2 The one rule

> **A return value is a claim. The Source of Truth (SoT) is the verdict. Read the verdict.**

Every check in this doc enforces or surfaces a violation of that rule.

### 0.3 Glossary (single source — used throughout)

| Term | Meaning |
|---|---|
| **SoT** | Source of Truth — the authoritative physical location of state (DB row/file/queue msg/external system record). UI is never SoT. |
| **FSV** | Full State Verification — read SoT before action, execute action, read SoT after, assert delta. §4.1 |
| **RED** | Rate / Errors / Duration — per-service health signals |
| **USE** | Utilization / Saturation / Errors — per-resource health |
| **SLI/SLO/SLA** | Indicator (metric) / Objective (target) / Agreement (contract) |
| **p50/p95/p99** | Latency percentile — never use mean alone |
| **STRIDE** | Spoofing/Tampering/Repudiation/Info-disclosure/DoS/Elevation — threat model |
| **SAST / DAST / SCA / IaC scan** | static / dynamic / dependency / infra-as-code scanners |
| **SBOM** | Software Bill of Materials (CycloneDX / SPDX) |
| **SLSA** | Supply-chain Levels for Software Artifacts (1→4) |
| **FMEA** | Failure Modes & Effects Analysis |
| **RCA** | Root Cause Analysis (§17) |
| **MTTR / MTBF** | Mean Time To Recovery / Between Failures |
| **PSI / KS / KLD / ECE** | Population Stability Index / Kolmogorov-Smirnov / KL Divergence / Expected Calibration Error |
| **PII** | Personally Identifiable Information |
| **BVA / ECP** | Boundary Value Analysis / Equivalence Class Partitioning |
| **CB** | Circuit Breaker (§7) |
| **N+1** | the query anti-pattern: 1 list query + N detail queries |

### 0.4 Decision trigger map (agent-grep this when you read symptoms)

| If you see / are told… | Jump to |
|---|---|
| "it's slow" / "P95 spiked" | §5, §12, §13 |
| "it crashed" / "outage" | §6, §7, §17 |
| "bill is too high" | §9 |
| "we got pwned" / CVE filed | §3, §14 |
| "model is wrong now" / drift | §15 |
| "tests pass but prod broke" | §4 (FSV), §17 |
| "new engineer can't ship" | §10, §11 |
| "deploy is scary" | §10, §19 |
| "I don't know if X happened" | §13, §4 |
| "benchmark says we're 2× faster" | §16 (verify before believing) |

---

## 1. Universal mental models (install before tools)

### 1.1 Trigger → Process → Outcome
```
[Trigger]  ──►  [Process]  ──►  [Outcome]
 observable    measurable     verifiable @ SoT
```
If you can't point at all three, you don't understand the feature. Hardening = make each leg cheap to inspect.

### 1.2 First-principles decomposition
1. What is *literally* happening at byte / SQL / HTTP / syscall level?
2. What invariant is being violated?
3. What single fact, if changed, makes the symptom impossible?
4. Why is that fact currently false?
5. What is the smallest structural change that makes it permanently true?

### 1.3 Symptom vs cause vs root cause
Symptom (user-facing) → Cause (immediate trigger) → Root cause (structural reason it could happen). Symptom fix = patch. Cause fix = fix. Root cause fix = **hardening**. Stop only at root.

### 1.4 Fail-closed not fail-open
Default path must be the safe path. Fail-open into undefined state = correctness + security bomb. Includes: auth failures, validation errors, downstream timeouts, schema mismatches, deserialization failures.

### 1.5 Defense in depth
Never trust a single control. Input validation **and** parameterized queries **and** least-priv DB user **and** WAF **and** logging. Layers survive partial compromise.

### 1.6 Asymmetry of risk
Reversible local edits — proceed. Hard-to-reverse / shared-state / destructive — confirm first. Match cost of acting wrongly against cost of pausing.

### 1.7 80/20 in hardening
Most issues cluster: missing indexes, N+1, no timeouts, no SLOs, no SBOM, no MFA, no FSV. Hit those before chasing edge cases.

---

## 2. Hardening session — top-level workflow

```
Phase 1 DISCOVER (read-only)   → Phase 2 RANK (likelihood × impact × effort)
Phase 3 PLAN (safety net first, PR-sized) → Phase 4 EXECUTE-LOOP per increment
Phase 5 RE-VERIFY against Phase 1 baselines
```

### 2.1 Phase 1 — Discover (read only; never edit)
- [ ] Asset inventory (hosts, repos, queues, buckets, secrets, certs, identities, 3rd-parties, domains).
- [ ] Dep graph (`madge`, `pydeps`, `cargo-modules`, `go mod graph`).
- [ ] 90-day metrics dashboard (RED + USE).
- [ ] Incidents + PIRs read.
- [ ] Scanner baselines (SAST/SCA/secret/IaC/container/cloud — §3 + §11).
- [ ] Codebase walk noting smells / dead code / anti-patterns / SoTs.
- [ ] Cost dashboard (last 90d, by service/tag/owner).
- [ ] Threat model status (STRIDE per data flow exists?).

### 2.2 Phase 2 — Rank
For each finding: likelihood × impact × ease-of-fix → must / should / nice / accept.
Document accepted risks with owner + review date.

### 2.3 Phase 3 — Plan
- Establish **safety net first**: characterization tests around current behavior (Feathers).
- Slice into PR-sized increments — each shippable, reversible, independently FSV-able.
- Define FSV evidence required per increment.

### 2.4 Phase 4 — Execute (per increment)
1. Snapshot SoT + metrics (BEFORE).
2. Make smallest change.
3. Run linters / types / SAST / SCA / unit / integration / property / mutation.
4. Manual FSV with synthetic data: happy + ≥3 edge classes from §4.5.
5. Stage deploy + repeat FSV.
6. Prod canary behind flag; monitor RED + USE.
7. Post-deploy verify SoTs (read-only).
8. Update docs + runbooks + risk register.

### 2.5 Phase 5 — Re-verify
Re-run §2.1 baselines; quantify delta; record residual risk + re-review schedule.

### 2.6 Stop conditions
- Test fails unexpectedly → §17 RCA before fixing.
- Assumption proves false → replan increment.
- Findings deeper than expected → re-rank, don't sneak scope.
- Prod metric shifts post-deploy → investigate; rollback if unclear.

### 2.7 Definition of Done (per task)
Change merged · scanners green · tests added/updated · **FSV evidence captured** · ≥3 edge cases tested · docs/runbooks updated · observability updated · deployed + verified · risk register updated.

---

## 3. Security hardening

Defer to OWASP Top 10:2025 + CIS Benchmarks + NIST CSF 2.0 + ASVS 5.0 (§21).

### 3.1 Threat model & asset inventory (must precede controls)
- [ ] Full asset inventory (can't harden the unknown).
- [ ] Data classification: public / internal / confidential / regulated. Priority follows.
- [ ] Trust boundaries explicit — every crossing needs authN + authZ + validation + logging.
- [ ] STRIDE per data flow, or PASTA for richer models.
- [ ] SBOM per build artifact.

### 3.2 OWASP Top 10:2025

| # | Category | Core controls |
|---|---|---|
| A01 | Broken Access Control | deny-by-default, server-side authZ every request, RBAC/ABAC, IDOR tests, no client-side checks |
| A02 | Security Misconfiguration | repeatable hardening, minimal platform, dev=stage=prod, CSP/HSTS, no default creds |
| A03 | **Software Supply Chain Failures** (new) | SBOM, signed artifacts (Sigstore), pinned deps + hashes, SCA in CI, SLSA provenance |
| A04 | Cryptographic Failures | TLS 1.2+, AES-GCM / ChaCha20-Poly1305, Argon2id passwords, no homemade crypto |
| A05 | Injection (SQL/NoSQL/LDAP/OS/template/log/**prompt**) | parameterized queries, allow-list inputs, output encode |
| A06 | Insecure Design | threat modeling, abuse cases, secure-by-design patterns |
| A07 | AuthN & Identity Failures | MFA, session mgmt, no credential stuffing exposure |
| A08 | Software & Data Integrity Failures | signed updates, integrity checks, no insecure deserialization, CI/CD hardening |
| A09 | Security Logging & Monitoring Failures | central logs, auth/authZ alerts, tamper-evident, retention |
| A10 | **Mishandling of Exceptional Conditions** (new) | no fail-open, timeouts everywhere, fuzz malformed input, race tests, structured error handling |

### 3.3 OS hardening (CIS L1 general / L2 sensitive / STIG gov)
- Disable/remove unused services & daemons.
- Mount `nodev,nosuid,noexec` on `/tmp`, `/var/tmp`, `/dev/shm`, `/home`.
- ASLR on; `kernel.randomize_va_space=2`, `kptr_restrict=2`, `rp_filter=1`.
- MAC: SELinux enforcing or AppArmor profile enforce.
- Patch SLA: critical ≤7d, high ≤30d.
- auditd → SIEM, ≥90d retention.
- Auth NTP (chrony + NTS); secure boot; LUKS/BitLocker on non-datacenter.

### 3.4 Identity / IAM
- Remove default accounts. MFA everywhere (FIDO2 > TOTP > SMS).
- Passwords per NIST SP 800-63B (≥12 chars, no rotation, breach-check via HIBP).
- Least privilege quarterly-reviewed. JIT elevation (sudo+log / PAM).
- Service accounts: no human login, scoped, rotated, not shared.
- Off-boarding ≤1hr; weekly orphan audit; break-glass vault.

### 3.5 Network
- Default deny ingress + egress + east-west.
- Segmentation: prod ≠ non-prod, app ≠ data, DMZ ≠ internal.
- WAF tuned per public endpoint; DDoS protection (Shield/Cloudflare).
- Admin planes behind VPN/zero-trust proxy — never public.
- mTLS service-to-service; **egress allow-list** (kills C2/exfil).
- DNS: DNSSEC, CAA pinned, registrar locked + MFA.

### 3.6 Cloud (AWS/Azure/GCP — CIS Cloud Foundations)
- Root: hardware MFA, no API keys, alarms on use.
- CloudTrail/Audit Log all regions, log validation on, immutable separate-account bucket.
- Block Public Access at account level; CMK encryption for sensitive data.
- IAM: no inline policies, no `*` in prod, permission boundaries, org SCPs, region restrictions.
- Tag everything (owner/env/data-class/cost-center). Untagged = quarantined.
- CSPM continuous scan (Wiz/Prisma/Security Hub/Defender).

### 3.7 Containers / Kubernetes
- Base images: distroless/Alpine/Chainguard, scanned + signed.
- USER directive (non-root); read-only rootfs; drop caps; CPU/mem limits+requests.
- Pod Security Admission `restricted` for non-system ns.
- NetworkPolicies default-deny.
- Secrets via Vault/SealedSecrets — never raw `Secret` in git.
- Image admission (Kyverno/OPA Gatekeeper/Connaisseur): block unsigned + over-threshold CVEs.
- kube-apiserver: no anonymous, RBAC on, audit log on; etcd encrypted at rest.
- Runtime threat detection (Falco/Tetragon); service mesh mTLS for multi-tenant.

### 3.8 Application
- Allow-list input validation at every boundary.
- Context-aware output encoding (HTML/attribute/JS/URL/SQL).
- Parameterized queries everywhere.
- CSRF tokens on cookie-auth state-changing endpoints.
- CORS: explicit origins, no `*` with credentials.
- Security headers: CSP, HSTS preload, X-Content-Type-Options nosniff, Referrer-Policy strict-origin-when-cross-origin, Permissions-Policy, X-Frame-Options DENY.
- Session cookies: HttpOnly, Secure, SameSite=Lax|Strict.
- Per-endpoint rate limit (esp. auth, password reset, search).
- Lockout / progressive delay on auth failures.
- File upload: MIME + magic byte + size cap + AV + out-of-webroot, never serve same-origin.
- SSRF defenses: block link-local (`169.254.169.254`) + private CIDRs by default in URL fetchers.
- No `pickle`/`unserialize` on untrusted input; vetted crypto libs only.

### 3.9 API
- OpenAPI is the contract; reject outside-spec.
- Per-endpoint authN + authZ. Resource-level authZ (BOLA/IDOR).
- Bounds on array / string / depth / batch size; pagination required on lists.
- Version + sunset policy; short token lifetime + rotation.
- Replay protection (nonce + timestamp) on sensitive ops.
- Webhook HMAC-SHA256 outbound; verify inbound.

### 3.10 Secrets
- Centralized store (Vault/Cloud Secret Mgr/Doppler/1Password Connect).
- None in code/Dockerfile/CI logs/chat.
- Pre-commit `gitleaks`/`trufflehog`; CI + historical scans.
- Short-lived dynamic creds where possible; static rotated.
- Owner / purpose / cadence / last-rotated metadata per secret.

### 3.11 Compliance mapping (when in scope)
NIST CSF 2.0 (Identify/Protect/Detect/Respond/Recover/**Govern**); NIST SP 800-53/-171; ISO 27001:2022; PCI DSS 4.0; HIPAA Security Rule; SOC 2 Type II; GDPR/CCPA. CIS Benchmarks map cleanly to most.

### 3.12 Incident response readiness
- Documented IR plan (detect/triage/contain/eradicate/recover/learn).
- On-call rota + pager tested; escalation defined.
- Forensic snapshot capability rehearsed (host image, mem, logs, freeze creds).
- Pre-approved comms templates (internal/customer/regulator).
- Blameless PIR ≤5 business days; action items tracked.

### 3.13 Backup / DR / BCP
- **3-2-1**: 3 copies, 2 media, 1 off-site/account/cloud.
- Encrypted backups; **restore drilled ≥ quarterly** (untested = doesn't count).
- RPO + RTO documented per system.
- Immutable backups (S3 Object Lock / Azure immutable) for ransomware defense.
- DR runbook rehearsed annually.

---

## 4. Correctness & accuracy hardening

### 4.1 Full State Verification (FSV) — the non-negotiable
> *Returns lie. Logs lie. SoT does not lie.*

Four steps:
1. **Define SoT** — what state, *where* (table.col / file / queue / S3 key / metric / external ID), *how* you'll read it, *expected* (exact / range / schema / count delta).
2. **Capture BEFORE** — read SoT, log value.
3. **Execute trigger** — capture response (response = evidence of *attempt*, not success).
4. **Capture AFTER, assert** — re-read SoT, compare to expected, record delta.

`200 OK` + unchanged row = **failed test**, no errors required. Most common silent bug class in modern systems.

### 4.2 Verification chain (every feature usually writes multiple SoTs)
Example *submit order*:
- `orders` row inserted with correct fields ✓
- `order_items` count matches cart ✓
- `inventory.available` decremented ✓
- queue `order.created` event emitted ✓
- external (Stripe) charge created at correct amount ✓
- `email_outbox` row queued ✓
- metric `orders_created_total` incremented ✓
- log entry w/ order ID + user ID ✓

Skip any → prod bug waiting.

### 4.3 FSV evidence artifact (required deliverable)
```
=== FSV Run: feature_x — 2026-05-12T22:15:03Z ===
[Test 1 happy] PASS
  SoT: orders.status
  Before: NULL → After: 'paid'  (latency 230ms)
  Side effects: 2 order_items, inv SKU-42 50→48, queue +1, stripe ch_xyz @ 100, email +1
[Test 2 empty cart] PASS
  Trigger: POST /orders {items:[]}
  Expected: 400 + no row written
  Response 400 ✓; orders count unchanged ✓
```

### 4.4 Test pyramid + complementary techniques

| Layer | Catches | Tools |
|---|---|---|
| SAST | obvious bugs/anti-patterns | Semgrep, CodeQL, SonarQube, mypy --strict, clippy, ruff, ESLint, tsc --noEmit, staticcheck |
| Strict types | nullability, contract drift | mypy strict, TS strict, no `interface{}` leakage |
| Unit | function logic | xUnit family |
| Property-based | edge inputs you didn't think of | Hypothesis, fast-check, Quickcheck, proptest |
| Mutation | weak tests passing on bug-injected code | Stryker, PIT, mutmut, cargo-mutants |
| Integration | wiring / SoT writes | real DB / queue / API |
| Contract | inter-service drift | Pact, OpenAPI diff |
| Fuzzing | parsers / deserializers / boundaries | libFuzzer, AFL++, go-fuzz, Atheris, cargo-fuzz, Jazzer |
| DAST | running-app attack surface | OWASP ZAP, Burp, Nuclei |
| Race/concurrency | shared-state bugs | Go `-race`, ThreadSanitizer, Loom (Rust), pytest-xdist repeated |
| Memory | UAF / leaks / OOB | ASan, MSan, Valgrind |
| E2E / FSV | full user journey + SoT | manual + Playwright/Cypress + DB readback |

### 4.5 Mandatory edge-case audit (≥3 categories per code path)
1. Empty (`""`, `[]`, `{}`, null, missing field)
2. Single item (off-by-one surfaces)
3. Max allowed
4. Max+1 (must reject cleanly)
5. Min allowed (0, 1, documented lower)
6. Min−1
7. Wrong type
8. Malformed (invalid JSON/UTF-8/email)
9. Unicode edges (emoji, RTL, combining, NUL bytes, very long)
10. Duplicate / replay (idempotency)
11. Out-of-order events (B before A)
12. Concurrent (two actors same instant)
13. AuthZ variants (owner / non-owner / admin / anonymous)
14. Tenant scope (A can't see B)
15. Time edges (DST, leap second, negative offset, end-of-month, clock skew)
16. Resource exhaustion (full disk / OOM / conn pool / rate-limit)

Per case log: input, SoT BEFORE, action, SoT AFTER, PASS/FAIL with expected vs actual.

### 4.6 Synthetic test data properties
Deterministic seed · distinguishable (`synthetic_user_2026_05_12_X`) · representative · boundary-rich · privacy-safe (generated, never prod-copy) · cleanup-tagged.

### 4.7 When a test fails
**Stop.** Don't rerun-and-hope. RCA first (§17). Then fix, add regression test pinned to bug ID, re-run **all** adjacent FSV scenarios. Fixes break neighbors.

---

## 5. Performance hardening

### 5.1 Target metric stack
| Metric | Always report |
|---|---|
| Latency | p50, **p95, p99**, p99.9 (never mean alone) |
| Throughput | req/s, qps, RPS — peak + sustained |
| Resource | CPU <70% avg / <90% peak; mem <80%; conn pool <80% |
| Error rate | by-endpoint, not just aggregate |
| Saturation | queue depth, lock wait, GC pause |

### 5.2 The performance loop (observe → profile → fix → verify)
1. **Observe** — RED + USE dashboards; identify p95/p99 SLO breach.
2. **Profile** — sampling profiler (perf, py-spy, pprof, async-profiler), flame graph, trace.
3. **Hypothesize** — bottleneck location: DB / cache / network / CPU / memory / GC / lock.
4. **Fix** — smallest change that targets identified bottleneck.
5. **Verify** — A/B compare p50/p95/p99 + throughput before/after; check regressions elsewhere.

### 5.3 Latency budget (allocate per hop, sum ≤ SLA)
```
DB query   20ms
Cache      5ms
App logic  30ms
External   30ms
Serial.    15ms
Total     100ms  ←  user SLA p95 budget
```
If any hop exceeds → that's the next fix.

### 5.4 Top backend bottlenecks (in order of frequency)
1. **N+1 queries** (§12.2) — detect via query log / APM; fix via JOIN / eager-load / DataLoader batch
2. **Missing / wrong indexes** — full table scans; `EXPLAIN ANALYZE` (§12.1)
3. **Over-fetching** (SELECT *, full row when only 3 cols) — column-select + projection
4. **Synchronous blocking** in request path — push email/AV scan/PDF gen to queue
5. **No caching** — recomputing identical results (§5.6)
6. **Unoptimized serialization** — JSON deepcopy / N+1 in serializer
7. **No connection pool** (or unbounded) — use PgBouncer/HikariCP; size = CPU+disk-spindles
8. **Lock contention** / long transactions
9. **GC pressure** — high-allocation paths; pool buffers
10. **Network round-trips** — batch APIs, HTTP/2, gRPC, response compression

### 5.5 CPU-bound vs IO-bound
- **CPU-bound** → algorithmic complexity, vectorize, SIMD, parallelize, native lib, reduce alloc, lower precision (f32 vs f64 if safe).
- **IO-bound** → async/await, pipelining, batching, prefetching, locality (cache, CDN, edge).

### 5.6 Caching strategy ladder

| Layer | Use when | Watch out for |
|---|---|---|
| Compiler / inline | hot inner loop | premature opt |
| CPU cache (data layout) | SIMD / large arrays | false sharing |
| Process memory (LRU) | per-request memoization | unbounded growth, staleness |
| Redis / Memcached | shared, cross-process | stampede, cold start, invalidation |
| HTTP / CDN edge | static assets, public read | cache poisoning, vary header |
| Read replica | read-heavy | replication lag (alert < 1s) |
| Materialized view | aggregation reads | refresh strategy |

**Cache pitfalls**: stampede (singleflight + jittered TTL), thundering herd, **invalidation is hard** (event-driven preferred), cache != SoT.

### 5.7 Other levers
- HTTP/2 + HTTP/3, gRPC for internal RPC
- Response compression (gzip/br/zstd)
- Persistent connections (keep-alive)
- Async + non-blocking I/O on request path
- Worker thread / job queue for CPU-bound or slow work
- Edge compute for geographically distributed users
- ETag / Last-Modified for client-side caching
- Server timing headers for client-side perception
- Profiling-guided optimization (PGO, LTO, BOLT)

### 5.8 Performance regression gating
- Bench in CI on every PR for hot paths.
- Compare to baseline; fail PR on >X% regression with statistical significance (§16).
- Track p95/p99 of key endpoints over time; alert on baseline drift.

---

## 6. Reliability / SRE hardening

### 6.1 SLI / SLO / Error Budget
- **SLI**: ratio of good events to total events (success / total, fast / total).
- **SLO**: target on the SLI (e.g. 99.9% requests successful, 99% < 500ms).
- **Error budget** = 1 − SLO over a window (month/quarter). When spent → freeze feature work, fix reliability.
- **Burn-rate alerts**: multi-window, multi-burn-rate (Google SRE Workbook). Beats raw threshold alerts.

| Window | Burn rate | Alert |
|---|---|---|
| 1h | 14.4× (2% budget in 1h) | page |
| 6h | 6× | page |
| 3d | 1× | ticket |

### 6.2 Four golden signals (per service)
1. **Latency** (p50/p95/p99)
2. **Traffic** (request rate)
3. **Errors** (rate + category)
4. **Saturation** (queue depth, pool usage, GC, disk)

### 6.3 RED (request-oriented) + USE (resource-oriented)
- **RED** per service: Rate, Errors, Duration
- **USE** per resource: Utilization, Saturation, Errors

### 6.4 Alert rules of thumb
- Alert on **symptom**, not cause (user-impacting, not CPU spike alone).
- Tie alerts to SLO budget burn, not raw metric thresholds.
- Every page must have a runbook.
- Page-or-ticket-not-both per alert; reduce false positives ruthlessly (paging = costly).

### 6.5 On-call hygiene
- ≤25% time on toil (Google SRE rule); the rest = engineering reliability away.
- Pager rotation: humane; primary+secondary; no single-human dep.
- Blameless postmortems (Dekker); action items tracked + verified at 30/60d.
- Tabletop exercises ≥2×/yr.

### 6.6 Capacity / scaling (read with §8)
- Load test at 2× current peak; identify next bottleneck.
- Headroom ≥2× peak for stateful tiers.
- Cache stampede protection; backlog alerts.

---

## 7. Resilience patterns (fault tolerance)

### 7.1 Pattern map

| Pattern | Protects against | When | Cost |
|---|---|---|---|
| **Timeout** | hung calls, thread exhaustion | every external call | zero |
| **Retry + exp backoff + jitter** | transient errors | idempotent ops | low; can amplify load — cap retries 2-3 |
| **Circuit Breaker** | cascading failures, repeated calls to dead dep | per external dep | low |
| **Bulkhead** | one slow dep exhausting all threads/conns | per dependency or tenant | low |
| **Rate limit** | overload, noisy neighbors | every public + internal API | low |
| **Idempotency key** | retry-induced duplicates | mutating ops | medium (dedup store) |
| **Fallback / graceful degradation** | dep unavailable | non-critical paths | varies |
| **Hedging** | tail latency from slow replicas | read-heavy, replicated | extra load |
| **Load shedding** | overload protection | system at capacity | drops some users |
| **Backpressure** | unbounded queue growth | producer-consumer | requires upstream cooperation |
| **Dead letter queue** | poison messages | async workers | storage |

### 7.2 Ordering (matters)
Bulkhead → Circuit Breaker → Retry → Fallback. Don't retry when CB open. Don't retry without idempotency key for mutations.

### 7.3 Retry policy
- Exponential backoff + **jitter** (mandatory — prevents synchronized retry storm).
- Retry only on clearly transient errors (network blip, 503, lock contention).
- Cap retries (2-3) and total retry budget.
- Never retry non-idempotent mutations without idempotency key.
- Honor `Retry-After` header.

### 7.4 Circuit breaker tuning
- States: Closed → Open (after N failures or slow-call rate) → HalfOpen (probe) → Closed.
- Open on **both** error rate AND slow-call rate.
- Threshold: typical defaults are 50% error rate over 20-100 calls.
- Half-open probe: small percentage; revert on failure.

### 7.5 Timeout policy
- Set timeouts **below** parent budget; cascade `context.WithTimeout` down call chain.
- Connect / read / write / total — set each.
- Default: never infinite.

### 7.6 Idempotency
- Mutating endpoints accept `Idempotency-Key` header.
- Store key → result for window (24h typical).
- Replay returns stored result; concurrent same-key requests block-and-wait or 409.

### 7.7 Cascading failure prevention
- Drop traffic upstream when total load > total capacity (load shed retries first).
- Health checks (readiness vs liveness — distinguish).
- Graceful shutdown: drain → stop accepting new → finish in-flight → exit.
- Avoid synchronous chains ≥4 deep (p99 multiplies; failure prob multiplies).

---

## 8. Scalability hardening

### 8.1 Six dimensions to score (1-5 each; ≤2 = priority)
1. **Scalability** — 10× traffic / 100× data plan?
2. **Security** — explicit boundaries, layered controls?
3. **Maintainability** — new dev ships in week 1?
4. **Performance** — p95/p99 budget met under realistic load?
5. **Deployability** — roll out, roll back, multi-version safe?
6. **Observability** — 3am page → root cause in 5 min?

### 8.2 Horizontal vs vertical
- **Vertical** = bigger box. Quick but ceilinged + SPOF.
- **Horizontal** = more boxes. Needs statelessness, partitioning, distributed coordination.
- Stateful tiers (DB) hardest to scale; design for read-replicas + sharding early.

### 8.3 Sharding / partitioning
- Choose shard key carefully — must distribute load evenly + locate related data together.
- Hot keys destroy throughput; consider key hashing or composite keys.
- Resharding is expensive; plan growth headroom 10×.

### 8.4 Caching levels (§5.6 expanded for scale)
- Read replicas for read-heavy DBs (alert replication lag <1s).
- CDN for static + cacheable dynamic.
- Edge KV (Vercel Edge / Cloudflare KV) for low-TTL global state.

### 8.5 Async / queue patterns
- Move slow work out of request path.
- Idempotent consumers + DLQ.
- Backpressure to producers.
- Saga/orchestration for cross-service workflows.

### 8.6 Statelessness
- HTTP layer should be horizontally stateless.
- Session in Redis or signed cookie (not local memory).
- Sticky sessions = scalability anti-pattern unless required.

---

## 9. Cost efficiency / FinOps

### 9.1 FinOps maturity ladder
1. **Visibility** — tag everything (owner/env/data-class/cost-center), enable Cost Explorer / Cost & Usage Report.
2. **Optimization** — rightsize, eliminate waste, storage tiers.
3. **Automation** — auto-scaling, scheduled shutdowns, continuous rightsizing.

### 9.2 Levers and typical savings

| Lever | Expected savings | Risk |
|---|---|---|
| Right-sizing (Compute Optimizer + CloudWatch 14-30d) | 15-30% | minor — wrong call hurts perf |
| Reserved Instances / Savings Plans (1-3yr) | 30-72% (steady-state only) | lock-in if usage drops |
| Spot / preemptible (fault-tolerant) | 60-90% | interruption — design for it |
| Graviton / ARM compute | 10-40% (if compatible) | requires test for parity |
| Storage tiering (S3 IA / Glacier; warm/cold blob) | 30-60% on cold data | retrieval latency + cost |
| Egress reduction (CDN / VPC endpoints / cross-AZ awareness) | 30-80% on egress | rearchitecture |
| Orphan cleanup (unattached EBS, stale snapshots, idle ELB, no-traffic Lambdas) | 5-15% | low if tagged |
| Non-prod shutdown after hours | 30-50% on non-prod | requires schedule + tag |
| Container density (K8s requests/limits tuned) | 20-40% | OOM if too aggressive |
| Cache hit-rate improvement | reduces compute + egress | invalidation complexity |

**Sequencing**: tag → CUR → rightsize → buy commitments (RIs/SPs) → automate. Buying commitments before rightsizing locks in waste.

### 9.3 K8s specifically
- Karpenter / Cluster Autoscaler.
- Separate system + workload node groups; taints/tolerations.
- Requests/limits enforced; OpenCost for showback by namespace/workload.
- Spot for stateless/fault-tolerant; on-demand for stateful.

### 9.4 Anti-patterns
- Buying RIs on un-rightsized fleet.
- No tagging → no chargeback → no incentive.
- Manual dashboards in 2026 — use AIOps recommendations.
- Treating commitments as one-time buys vs portfolio.

### 9.5 KPIs
Cost per customer / transaction · % automated cost actions · reserved-to-on-demand ratio · forecast accuracy · gross margin improvement.

---

## 10. Architecture hardening

### 10.1 Anti-patterns watchlist (find one → found hardening work)
- **Big Ball of Mud** · **Distributed monolith** · **Shared DB across services**
- **God service / God class** (>60% of logic or traffic)
- **Stovepipe** (point-to-point integrations, no shared contract)
- **Missing circuit breakers** · **Synchronous chains ≥4 deep**
- **No bulkheads** (one tenant starves all)
- **The Blob** · **Spaghetti** · **Lava flow**
- **Golden hammer** · **Accidental vendor lock-in**
- **Overengineering / Inner-platform** · **Premature abstraction**
- **Reinvented wheel** (handwritten crypto/retry/queue/ORM)
- **Cargo cult** (Netflix-does-it w/o matching constraints)
- **Anemic Domain Model** (data bags, logic in service classes)

### 10.2 Coupling & cohesion
- High cohesion (module = one thing well).
- Low coupling (minimal, stable-interface deps).
- Acyclic dep graph at module/package/service.
- Stable Dependencies Principle (depend toward stability).
- Stable Abstractions Principle (stable = abstract).

### 10.3 FMEA per external dep
For each (DB/queue/3rd-party/DNS/time/network): what if **slow**? **unavailable**? **wrong data**? **50% errors**? **1% errors**?
Each answer = control (timeout / CB / retry / bulkhead / fallback) OR documented accepted risk.

### 10.4 SPOFs to interrogate
Single DB primary? Single AZ/region? Single LB? Single CI server? Single human knows X (bus factor)? Single shared cache? Single auth provider?

Document accepted SPOFs; runbook each.

### 10.5 Data ownership
- One service per table (no shared write).
- Cross-service reads via API/event, not direct DB.
- Schema evolution: backward-compatible migrations, expand-contract pattern.
- Consistency model documented (strong / eventual, txn boundaries).
- Idempotency keys on every retryable mutation.
- PII map: where stored, who accesses, deletion path.

### 10.6 Deployability
- Reproducible build · automated deploys (manual = bugs).
- Blue/green / canary / rolling; never big-bang.
- Rollback <5 min, tested.
- DB migrations forward+backward compatible across ≥1 version.
- Feature flags for risky changes; retirement tracked (flags >90d on/off = decide + delete).

---

## 11. Code quality & maintainability hardening

> **Maintainability is a measurable property, not a vibe.** It is the cost (time + risk + cognitive load) of the next change. Hardening = reduce that cost. A maintainable system is one where a new engineer can ship a correct change in week 1; an unmaintainable one needs 6 months and a tribal-knowledge sherpa.

### 11.1 Tech debt taxonomy
1. Intentional/strategic (documented payback).
2. Unintentional (inexperience/oversight — most debt).
3. Outdated design.
4. Environmental (aged frameworks/runtimes).
5. Architectural (structural — expensive).

### 11.2 Complexity targets
- Cyclomatic per function: ≤10 target, alarm >15.
- Cognitive (SonarQube): captures readability.
- Function ≤30 lines; file ≤500 lines; class ≤200 lines / ≤7 public methods.
- Coupling: afferent (Ca) / efferent (Ce); instability I = Ce/(Ca+Ce).
- Coverage: line < branch < mutation score (strongest).

### 11.3 Smell → refactoring (Fowler)

| Smell | Refactoring |
|---|---|
| Long method | Extract Method |
| Long parameter list (>4) | Parameter Object / Builder |
| Large class / God Object | Extract Class |
| Feature envy | Move Method |
| Data clumps | Extract Class |
| Primitive obsession | Value Object |
| Switch/if cascades on type | Polymorphism |
| Shotgun surgery | Consolidate via Move Method/Field |
| Divergent change | Extract Class |
| Message chains `a.b().c().d()` | Hide Delegate |
| Middle man | Remove Middle Man |
| Comments explaining *what* | Rewrite code |
| Magic numbers/strings | Named Constant |
| Mutable shared state | Immutable / encapsulate / channels/locks |
| Speculative generality | Inline / collapse |
| Temporary field | Replace with Method / Null Object |

### 11.4 Dead code (delete, don't comment)
- Unused-symbol tools: `ts-prune`, `vulture`, `deadcode` (Go), `cargo-udeps`, `knip`.
- Runtime coverage: 30d zero-hit endpoints → candidates.
- Feature flags >90d on/off → decide, delete flag + branch.
- TODO/FIXME/XXX/HACK >12mo → issue or delete.
- Orphan files / DB columns / indexes (query `pg_stat_user_tables`/`pg_stat_user_indexes` for zero r/w).
- Caveat: reflection / DI / dynamic dispatch / generated SDKs — verify before delete.

### 11.5 Duplication
Detectors: `jscpd`, `simian`, SonarQube duplication, `dupl`. Distinguish coincidental (looks same now, will diverge) from semantic (same intent — extract). 3+ occurrences proven shared intent → extract.

### 11.6 Boy-Scout rule (per PR)
Rename one bad name · delete one stale comment · extract one too-long method · tighten one type · replace one magic number. Compounded across team = debt repayment.

### 11.7 Minimum tooling stack
Linter + formatter pre-commit + CI · type checker (strict) in CI · SAST · SCA · secret scan · test runner + branch coverage + periodic mutation · quality dashboard.

### 11.8 Maintainability metrics (track these like SLOs)

| Metric | How to measure | Healthy target |
|---|---|---|
| **Onboarding time** | days from clone to merged real PR | ≤5 business days |
| **Bus factor** | min # devs whose absence blocks each subsystem | ≥2 per critical area |
| **Change failure rate** (DORA) | % of deploys causing incident/rollback | <15% |
| **Lead time for change** (DORA) | commit → prod | <1 day (elite); <1 week (high) |
| **MTTR** (DORA) | incident → recovery | <1 hr (elite); <1 day (high) |
| **Deploy frequency** (DORA) | shipped per dev per week | ≥1 (high) |
| **Avg PR review time** | open → merged | <24h |
| **Cognitive complexity** (Sonar) | per function | <15 |
| **Doc coverage** | public APIs with usage example | ≥90% |
| **Stale-doc rate** | docs untouched + tests failing | trend down |
| **% of files no single dev "owns"** | git blame distribution | rising = good |

DORA metrics are research-validated predictors of org performance (Forsgren / Humble / Kim, *Accelerate*). Measure them.

### 11.9 Ousterhout principles (*A Philosophy of Software Design*)
- **Deep modules, shallow interfaces.** A module's value = functionality − interface complexity. Big functionality + small API = deep (good). Small functionality + big API = shallow (bad — pass-through layers, leaky abstractions).
- **Information hiding** is the most powerful tool. Each module hides design decisions from others.
- **Pulling complexity downward** — the module's author absorbs complexity so its many users don't.
- **Different is different** — don't make slightly-different things look the same (confuses readers).
- **Comments should describe what code can't** — invariants, intent, "why," not "what." The "what" should be readable from names + types.
- **Strategic vs tactical programming** — invest in design even when it slows the immediate task. The interest rate on bad design is high.

### 11.10 Cognitive load reduction
- **Locality of behavior** — code that changes together lives together. Maximize how much of a feature is in one file/module.
- **Symmetry** — similar things look similar; different things look different.
- **Linearity** — top-to-bottom reading order in functions; minimize jumps.
- **Naming** — names should be predictive. If a reader has to read the body to know what a function does, the name is wrong.
- **Single Level of Abstraction** per function — don't mix high-level orchestration with low-level details.
- **Magic numbers / strings** → named constants with explanatory names.
- **Reduce indirection** — every hop (function call, layer, factory) costs a context switch. Don't add them speculatively.
- **Limit "spooky action at a distance"** — avoid globals, mutable singletons, monkey-patching, dynamic dispatch where static works.

### 11.11 Naming discipline (highest-leverage refactor)
- **Predictive**: `cleanupOrphanedTokens()` not `process()`.
- **Pronounceable**: `customerInfo` not `custInfo`.
- **Searchable**: avoid single letters except very short scopes (`i` in tight loop OK).
- **No type encoding** (`strName`, `iCount`): types already say so.
- **Consistent vocabulary**: pick `fetch` vs `get` vs `retrieve` and use one.
- **Domain language alignment**: if business calls it `Customer`, code calls it `Customer`, not `User`/`Account`/`Party`.
- Rename ruthlessly; modern IDEs make it safe.

### 11.12 Domain-Driven Design — ubiquitous language & bounded contexts
- **Ubiquitous language** — one term per concept, used identically in code, tests, docs, conversation. Glossary lives in the repo (`docs/glossary.md`).
- **Bounded context** — a model is consistent inside one context; across contexts, translate explicitly (context map).
- **Anti-corruption layer** when integrating with a system using foreign vocabulary.
- DDD prevents the "what is a User?" debate from re-recurring in every meeting.

### 11.13 Documentation as code

| Doc type | What | Where | Update cadence |
|---|---|---|---|
| **README.md** | what + quickstart + dev setup | repo root | every major change |
| **ADRs** (Architecture Decision Records) | why a choice was made, alternatives, consequences | `docs/adr/NNNN-title.md` | one per significant decision; never delete, supersede |
| **Runbook** | how to respond to alert X | `docs/runbooks/` linked from alert payload | when alert added or incident occurs |
| **Onboarding guide** | new-hire week-1 path | `CONTRIBUTING.md` / `docs/onboarding.md` | quarterly |
| **API docs** | endpoints, schemas, examples | OpenAPI / generated | on every API change |
| **Glossary** | ubiquitous-language terms | `docs/glossary.md` | when terms emerge/change |
| **Architecture overview** | C4 diagrams (context/container/component) | `docs/architecture/` | on structural change |
| **CHANGELOG.md** | user-facing changes per release | repo root | per release |
| **CODEOWNERS** | who reviews what | `.github/CODEOWNERS` | on team change |
| **Incident postmortems** | timeline + root cause + actions | `docs/incidents/` | per incident |

**ADRs are the single highest-leverage doc** — they capture the *reasoning* that's invisible from code. Future-you will not remember why; ADR tells you. Format: Title / Status (proposed/accepted/superseded) / Context / Decision / Consequences / Alternatives considered.

### 11.14 Doc anti-patterns
- Docs in a separate repo nobody reads.
- Auto-generated API docs with no examples.
- Tutorials that lie because nobody re-runs them on dependency bumps.
- "Architecture" diagrams locked in Visio / Confluence / Figma — drift instantly. Use Mermaid / PlantUML / Diagrams-as-Code committed to repo.
- Comments duplicating obvious code (`i++; // increment i`).
- Multi-paragraph docstrings on trivial functions.

### 11.15 Comment discipline
**Write a comment when** (and only when):
- It encodes a non-obvious *why* (constraint, bug, invariant, edge-case rationale).
- It documents a contract beyond what types express (preconditions, error semantics).
- It warns ("not thread-safe", "callers must hold X lock", "O(n²) — fine because n<100").

**Don't write** what code already says (`// loops over items`). Don't reference task IDs / PR numbers / "added for X feature" — that's PR description content, rots in code.

### 11.16 Developer experience (DX) as a maintainability vector
- `make` / `just` / `task` recipes for every common dev op (`make test`, `make lint`, `make run-local`).
- One-command local-dev startup (`docker-compose up` or `nix-shell`).
- Pre-commit hooks autofix formatting + lint; don't make humans do bot work.
- Fast feedback: incremental compile, watch-mode tests, hot reload.
- Reproducible env (lockfiles, Nix, devcontainers).
- IDE config in repo (`.vscode/settings.json`, `.idea/`) for shared standards.
- Time-to-first-PR measured for new hires; <5 days = healthy.

### 11.17 Testing as documentation
- Tests are the only docs that fail when wrong → trust them more than prose.
- Name tests as specifications: `test_returns_400_when_email_invalid`, not `test_email_1`.
- Arrange-Act-Assert (or Given-When-Then) structure makes intent obvious.
- One assertion-cluster per test; multiple tests > one mega-test.
- Snapshot tests for stable output shapes (with care — they decay when over-broad).
- Example tests as living tutorials for libraries.

### 11.18 Refactoring time budget
- **Allocate explicit time**: e.g., "20% of each sprint" or "Friday afternoon" or "one tech-debt PR per developer per week."
- Boy-Scout rule per PR (§11.6) is the smallest unit.
- Refactor in code-being-changed-anyway (no separate "big refactor" PRs unless safety-net allows).
- Track debt explicitly: tag issues `tech-debt`; trend over time.

### 11.19 Knowledge sharing (defeats bus factor)
- **Pair / mob programming** for the gnarliest areas — knowledge transfers free.
- **Code review** is a knowledge-sharing event, not just a quality gate. Reviewers learn the area too.
- **Brown-bag talks / RFC reviews / internal docs** for non-trivial decisions.
- **Rotate on-call** so multiple people understand each system.
- **Don't tribally-allow** anyone to be the only person who can deploy / debug / build a system.

### 11.20 Maintainability checklist (per service)
- [ ] README explains what + quickstart + dev setup in <5min read.
- [ ] One-command local-dev startup works on a fresh clone.
- [ ] ADRs exist for non-obvious architectural decisions.
- [ ] CODEOWNERS file current.
- [ ] Glossary / ubiquitous language documented.
- [ ] Runbook exists for every alert.
- [ ] Architecture overview (C4 diagrams) checked into repo as code.
- [ ] CHANGELOG maintained per release.
- [ ] Onboarding guide tested by an actual new hire in the last 6 months.
- [ ] No single-person bus factor on any critical subsystem.
- [ ] Cognitive complexity <15 / function (trend, not gate).
- [ ] Public APIs have at least one runnable example.
- [ ] Refactoring time explicit in sprint planning.
- [ ] DORA metrics measured + reviewed quarterly.
- [ ] No "do not touch" lava-flow code without ADR explaining why.

---

## 12. Database hardening

### 12.1 EXPLAIN ANALYZE — read it
| Node | Verdict |
|---|---|
| `Index Only Scan` | Best |
| `Index Scan` | Good |
| `Bitmap Heap Scan` | OK |
| `Hash Join` | Usually good |
| `Seq Scan` | Bad on large tables |
| `Nested Loop` | Bad on large tables (good for small) |

### 12.2 N+1 queries — the #1 backend killer
Detect: query log in dev/test fails build on >2× expected query count.
Fix: JOIN (eager-load via ORM `include`/`with`); DataLoader batch (WHERE id IN); denormalize for read-heavy.

### 12.3 Indexing strategy
- B-tree: equality + range (default).
- GIN: full-text, JSONB, arrays.
- GiST: ranges, geo.
- Hash: equality only.
- Composite: leftmost-prefix rule — column order matters.
- Partial: filter subset (`WHERE deleted_at IS NULL`).
- Covering / INCLUDE: index-only scans.
- Watch unused indexes (write overhead) — `pg_stat_user_indexes`.

### 12.4 Query patterns
- `SELECT specific_cols` not `SELECT *`.
- Pagination: keyset (cursor) not OFFSET on large tables.
- `EXISTS` > `IN` for subqueries on large sets (often).
- COUNT(*) optimization: estimated counts (`pg_class.reltuples`) for huge tables.
- Avoid functions on indexed columns (kills index use): `WHERE date(created_at) = ...` → `WHERE created_at >= ... AND created_at < ...`.

### 12.5 Connection pooling
- PgBouncer / pgpool / HikariCP / app-built pool — never raw connections.
- Pool size ≈ (cores × 2) + spindles (Hikari heuristic); tune by load test.
- Idle timeout configured.
- App must release conn promptly (no holding across `await` boundaries unnecessarily).

### 12.6 Replication & HA
- Read replicas for read-heavy; alert replication lag <1s.
- Tested failover; documented runbook.
- Backup encrypted, off-site, **restore tested ≤quarterly**.

### 12.7 Schema discipline
- `NOT NULL` / `CHECK` / `UNIQUE` / `FOREIGN KEY` per business invariant (push integrity to DB).
- Migrations: expand-contract, reversible, online (no lock) for large tables.
- Soft vs hard delete decided per table; retention aligned to legal.
- RLS for multi-tenant.

### 12.8 Ops
- Slow query log on, reviewed weekly.
- Autovacuum (PG) running; `pg_stat_statements` enabled.
- Audit log: DDL + privileged ops + failed logins.
- DB on private subnet only.
- Least-priv app user; separate users migration / app / reporting.
- TLS in transit; encryption at rest with managed keys.

---

## 13. Observability hardening

> **Observability ≠ Monitoring.** Monitoring asks pre-defined questions ("is CPU > 80%?"). Observability lets you ask **new** questions about a system without re-deploying — the property that you can reconstruct any internal state from external outputs. Hardening = make every state change emit a trail rich enough to debug an unknown future failure.

### 13.1 Three pillars + the rising fourth
1. **Metrics** — counters / gauges / histograms; aggregate; low-cardinality; cheap; alertable.
2. **Logs** — structured JSON; high-cardinality; searchable; retained by class.
3. **Traces** — distributed (OpenTelemetry); request-spanning; sampled; span-level latency attribution.
4. **Continuous profiling** (Parca / Pyroscope / Polar Signals / Datadog Continuous Profiler) — flame graphs from prod 24/7; catches CPU/alloc/lock issues invisible to RED+USE.

Modern stack glues these via **exemplars** (a metric data point links to a trace ID that links to spans that link to logs that link to profiles). When the page fires, one click → flame graph at the moment of the spike. Build that linkage in early.

### 13.2 OpenTelemetry as the standard
- **OTel is the vendor-neutral wire format** for metrics + logs + traces. Pick OTel SDKs + OTLP collector regardless of vendor (Datadog/New Relic/Grafana/Honeycomb/Tempo all consume it).
- Avoid vendor-proprietary agents in code; ingest via OTel Collector → vendor.
- Auto-instrumentation for common frameworks (HTTP server, DB driver, queue client, gRPC) → 80% coverage for near-zero code change.
- Manual spans for business-critical paths the framework doesn't know about.

### 13.3 SLI types (don't only measure latency + errors)

| SLI type | Question it answers | Example |
|---|---|---|
| **Availability** | did the request succeed? | 200/total |
| **Latency** | how long? | p95 < 500ms |
| **Throughput** | how many? | qps |
| **Freshness** | how stale is the data? | data ≤ 5min old for 99% of reads |
| **Correctness** | right answer? | reconciliation check pass rate |
| **Coverage** | what % of jobs processed? | batch jobs completed / scheduled |
| **Durability** | data survived? | object exists after replication |
| **Quality** | good-enough response? | personalization served vs fallback |

Latency + availability alone miss most ML, batch, and data systems.

### 13.4 Cardinality discipline (most expensive observability mistake)
- **Don't put unbounded high-cardinality values in metric labels** (user_id, request_id, full URL with IDs, raw IP). Each unique combination = a new time series → memory + cost explosion in Prometheus/Mimir.
- Bucket them: `endpoint=/users/:id` not `endpoint=/users/42`; status_class `2xx/4xx/5xx` not full code (or both at different granularities).
- High-cardinality belongs in **logs + traces**, not metrics.
- Audit existing metric series count regularly; alert on rapid growth.

### 13.5 Logs — structured, redacted, retained by class

**Required fields (every entry):** UTC ISO-8601 timestamp · service · version · env · request_id · trace_id · span_id · user/tenant id · severity · message.

**Log levels — discipline**:
- `ERROR`: action required, exception leaked, SLO threat. Alertable.
- `WARN`: unexpected, degraded path. Aggregate-alertable.
- `INFO`: significant state change. Low volume.
- `DEBUG`: developer diagnostic. Off in prod by default.
- `TRACE`: byte-level. Never in prod default.

**Never log:** secrets, full PII, tokens, full card numbers, raw passwords, full request bodies for auth endpoints. **Redact at the logger layer** (not per-call-site — too easy to miss). Allow-list what gets logged for sensitive fields.

**Retention by class** (typical):
- Hot (queryable, indexed): 7-30d.
- Warm (queryable, slower): 30-90d.
- Cold (archive, compliance): 1-7y per regulatory class.
- Audit / security logs: tamper-evident (S3 Object Lock, write-once); ≥1y minimum.

### 13.6 Tracing — sampling matters
- **Propagate W3C Trace Context or B3** across every hop (HTTP, gRPC, queue, async boundaries).
- **Sampling strategies**:
  - Head-based (decide at request start): cheap, simple; default 1-10%.
  - **Tail-based** (decide after request completes): keep all errors + all slow + N% baseline. Best for debugging, costs more.
  - Dynamic / adaptive: increase rate during incidents.
- Span every external call, DB query, queue op, cache lookup, business decision.
- Span attributes: input sizes, output sizes, cache hit/miss, retry count, error class.
- **Errors sampled at 100%** — a bug affecting 0.1% of requests produces zero traces at 1% baseline sampling.

### 13.7 Metric stack
- **RED** (services): Rate, Errors, Duration — per-endpoint + aggregate.
- **USE** (resources): Utilization, Saturation, Errors — per CPU/mem/disk/net/conn-pool/thread-pool.
- **Four Golden Signals** (Google SRE): Latency / Traffic / Errors / Saturation.
- Latency: **histogram with explicit buckets** for p50/p95/p99 (summary type loses information across instances; use histogram + `histogram_quantile`).
- **Business metrics co-located** (orders_created_total, signups_total, model_predictions_total) — bridges product to engineering.
- **Exemplars** (Prometheus) — attach a trace ID to a histogram bucket sample.

### 13.8 Health checks — distinguish three types

| Check | Answers | Failure action | Frequency |
|---|---|---|---|
| **Liveness** | is the process alive? | restart pod | 10-30s |
| **Readiness** | can it serve traffic right now? | remove from load balancer | 1-10s |
| **Startup** | finished initialization? | delay liveness checks | once at boot |

Conflating liveness with readiness = restart loops; conflating readiness with liveness = traffic to broken pod.

### 13.9 Synthetic + RUM (real user monitoring)
- **Synthetic** — scripted probes from external locations (Datadog Synthetics / Checkly / Pingdom). Catches DNS / CDN / region issues backend metrics miss. Run for every critical journey from ≥3 regions.
- **RUM** — beacon from real user browsers/apps (Sentry / Datadog RUM / Cloudflare Web Analytics). Captures p95 the user actually experiences (not your edge). Core Web Vitals (LCP, INP, CLS) for web.

### 13.10 Continuous profiling
- Run a sampling profiler (eBPF-based: Parca/Pyroscope/Polar Signals) 24/7 in prod with <1% overhead.
- Flame graphs available for any time window; diff between deploys to spot regressions.
- Catches: CPU hot loops, allocation spikes, lock contention, GC pressure, off-CPU time (IO wait).
- Combine with eBPF for syscall-level visibility (Pixie, Cilium Hubble).

### 13.11 Dashboards — discipline
**Three-tier dashboard system per service**:
1. **Service overview** (RED + golden signals; SLO status; deploy markers; recent incidents). On-call landing page.
2. **Resource detail** (USE per resource; pool depths; queue lengths). Diagnostic.
3. **Business outcome** (transactions, conversion, revenue impact). Product/exec view.

Anti-patterns: dashboards with 50+ panels nobody reads; metrics with no SLO context; no deploy markers; no link out to traces/logs.

### 13.12 Alerting principles
- **Symptom not cause.** Alert on user-impacting SLO burn, not CPU spike alone.
- Tied to error budget burn-rate (multi-window, multi-burn-rate; §6.1).
- Page only what requires now-action; ticket otherwise.
- Every page has a runbook URL in the alert payload.
- Alert payload includes: dashboard link, trace exemplar, recent deploys, runbook, on-call rotation.
- Tamper-evident logs for security-relevant alerts.
- **Routinely measure alert quality**: MTTR per alert, page-to-ticket ratio, false-positive rate, alerts-per-on-call-shift (>2 = burnout risk).

### 13.13 Service catalog / dependency map
- Single registry: every service with owner, on-call, repo, runbook, SLOs, dependencies (upstream + downstream).
- Backstage / OpsLevel / Cortex / hand-rolled YAML — pick one, keep it current.
- Dep map exposes blast radius before incidents (chaos engineering input; §18).

### 13.14 Cost-aware observability
- Logs are the budget killer; metrics second. Traces with tail-sampling cheapest.
- **Drop verbose logs at source** (sampling, level config); cheaper than ingest-then-discard.
- Lifecycle policy on log/metric storage (hot/warm/cold).
- High-cardinality offenders: audit + fix.
- Track observability cost-per-request as its own SLI.

### 13.15 Production monitoring checklist
- [ ] OTel instrumented (metrics + logs + traces with shared trace_id).
- [ ] CPU / mem / disk / net per host with alerts (CPU 80%/5min, mem 85%, disk 90%).
- [ ] Pod restart > 3 in 10min → page; OOM kills tracked.
- [ ] Cross-AZ latency baseline + 3× deviation alert.
- [ ] 4xx + 5xx per endpoint (aggregate hides broken endpoints).
- [ ] p50/p95/p99 per endpoint (never just average).
- [ ] Request throughput with anomaly detection (sudden 50% drop → upstream/routing).
- [ ] Memory growth rate (10MB/hour → OOM in days).
- [ ] End-to-end latency for critical journeys (checkout/login/search).
- [ ] Errors sampled at 100% (baseline misses 0.1% bugs).
- [ ] Connection refused / DNS failure / cert expiry alerts.
- [ ] Liveness / readiness / startup probes correctly separated.
- [ ] Synthetic probes for top user journeys from ≥3 regions.
- [ ] RUM beacon installed; Core Web Vitals tracked.
- [ ] Continuous profiler running with deploy-diff capability.
- [ ] Service catalog current; every service has owner + runbook + SLO.
- [ ] Alert MTTR + false-positive-rate tracked.
- [ ] Observability cost-per-request tracked.
- [ ] Log retention policy by class enforced.
- [ ] Metric cardinality audit run monthly.

---

## 14. Supply chain hardening

### 14.1 Three pillars
1. **SBOM** — what's inside (CycloneDX or SPDX).
2. **Signing** — Sigstore (Cosign + Fulcio + Rekor); keyless via OIDC.
3. **SLSA** — provenance + build integrity.

### 14.2 SBOM
- Generate every build (Syft / Trivy / cdxgen).
- Full dependency tree (transitives are where risk hides).
- Store with artifact (release asset or OCI attestation).
- Sign the SBOM too.

### 14.3 SLSA levels

| Level | Requires | Prevents |
|---|---|---|
| 1 | build documented, provenance exists | nothing formal — baseline |
| 2 | signed provenance from trusted service | tampered builds |
| 3 | hardened build platform, access controls, isolated/scripted build, signed tamper-resistant provenance | insider threats, compromised build servers |
| 4 | two-party review, hermetic reproducible builds | nation-state |

Most teams: target SLSA 2 fast (GitHub Actions + `slsa-github-generator`/`actions/attest-build-provenance`); SLSA 3 with Docker Hardened Images or hermetic builders.

### 14.4 Sigstore stack
- **Cosign** — sign/verify container images and artifacts
- **Fulcio** — short-lived signing certs via OIDC identity
- **Rekor** — immutable transparency log
- **Gitsign** — Git commit signing via Sigstore

### 14.5 Verification at deploy
Policy step in deploy pipeline that checks:
- Valid signature (cosign verify).
- Provenance predicate matches expected repo + workflow.
- Vulnerability scan + license scan pass.
- Admission control (Kyverno/Connaisseur/Gatekeeper) enforces.

### 14.6 Dependency hygiene
- Pin by version AND hash (`pip --require-hashes`, npm lockfile integrity, `go.sum`).
- Lockfiles committed.
- Pin GH Actions to SHA (not tag — tags mutable).
- Private registry / dep proxy — no direct public-registry pulls in prod.
- SCA in CI; block PRs with critical/high CVEs.
- Register internal package names on public registries (dep confusion defense).
- Watch typosquatting (`reqeusts`, `colourama`).
- Reproducible builds where possible; hermetic builders.

---

## 15. AI / ML / LLM-specific hardening

### 15.1 Drift detection
| Drift type | Detect with | Threshold |
|---|---|---|
| **Data drift** (input distribution) | KS test (continuous), Chi-squared (categorical), PSI (credit) | PSI > 0.25 = major shift |
| **Concept drift** (P(y\|x) changes) | performance drop on labeled holdout | depends on baseline |
| **Label shift** (P(y) changes) | marginal label distribution | recalibrate threshold |
| **Score drift** (output dist shift) | KS on prediction distribution | KS > 0.10 → check inputs |
| **Calibration drift** | Expected Calibration Error (ECE), reliability diagram | ECE > 0.05 = drifting |

### 15.2 Performance metrics
- **Classification**: Accuracy, Precision, Recall, F1, ROC-AUC, Average Precision, ECE.
- **Regression**: MAE, RMSE, R², MAPE.
- **Ranking**: NDCG, MAP, MRR.
- **Calibration** (often more important than accuracy): ECE, Brier score, reliability diagram.
- **Per-segment** breakdown (model can be 95% overall but 60% on minority class).

### 15.3 Label-delay strategies
When labels arrive late or never:
- **Proxy metrics**: confidence calibration, prediction entropy, OOD detection rate.
- **Confidence-Based Performance Estimation (CBPE)** — NannyML approach.
- **Two-lane monitoring**: confirmed-label lane (full metrics) + proxy lane (ECE, OOD).

### 15.4 Feature-importance-weighted drift
Weight drift severity by SHAP importance computed at training time. Reduces alert noise from low-signal features.

### 15.5 LLM-specific hardening
- **Prompt injection defense** — content filter + hierarchical system-prompt guardrails + response verification + (optional) PromptArmor-style LLM guard with reasoning model. Reported ASR: 73% → <1% with combined stack.
- **MELON / masked re-execution** — re-run agent with masked tool returns; flag agents whose tool calls match the masked run (indicates injection-driven divergence).
- **Multi-agent court / judge** — defense + prosecution + judge LLM; lower FPR than single-detector.
- **Output validation** — schema check, regex, secondary classifier before returning to user.
- **Tool scoping** — least-privilege tool access; allow-list per agent role.
- **Sandbox tool execution** — ephemeral microVM (Vercel Sandbox / Firecracker) for code execution.
- **Rate limit per user / per token-budget**.
- **PII redaction in/out**.
- **Audit log of every prompt + tool call + response** — required for post-incident analysis.
- **Hallucination reduction**: retrieval grounding (RAG), structured output (JSON-schema), self-consistency check, chain-of-verification, fact-checker LLM, confidence calibration.
- **RAG hardening**: chunk-level provenance, query rewriting safety, embedding model versioning, cross-context contamination isolation, content filtering on retrieved chunks.
- **Eval suite**: golden set + adversarial set + drift-monitor set; run on every model bump.
- **Model version provenance**: pin model + temperature + system prompt hash; log per inference.

### 15.6 ML model registry / lineage
- Track: data version, feature engineering version, training code commit, hyperparameters, training metrics, eval metrics, calibration, evaluation set hash.
- Reproducible training: seed, hardware, library versions.

### 15.7 Production ML readiness checklist
- [ ] Baselined evaluation set (gold + adversarial + segment-stratified).
- [ ] Drift monitoring (data + concept + score + calibration).
- [ ] Performance monitoring on confirmed labels (or proxies if delayed).
- [ ] Rollback path to prior model version (instant).
- [ ] Shadow / canary mode for new model.
- [ ] A/B framework with statistical rigor (§16).
- [ ] Cost-per-prediction tracked (esp. LLMs).
- [ ] Fairness / bias check per protected attribute.
- [ ] Explainability hooks (SHAP / LIME / attention).
- [ ] PII / data governance compliance.

---

## 16. Benchmarking discipline

### 16.1 Three benchmark scales
| Scale | Measures | Pitfalls |
|---|---|---|
| **Microbench** | function / method (ns–µs) | dead-code elimination, constant folding, JIT warmup, cache effects, allocation hidden |
| **Mesobench** | subsystem (ms) | dep mocking diverges from prod |
| **Macrobench** | full app (sec) | realism cost; need prod-like data |

Test the actual product the way it's used. Macrobench gives best decision quality.

### 16.2 Statistical rigor (mandatory)
- Run repetitions: **never a single run**.
- Report mean, median, stdev, CoV (coefficient of variation); CoV <5% before trusting.
- Use BenchmarkDotNet / google/benchmark / JMH / criterion-rs — they auto-determine iteration count for stability (standard error threshold).
- Statistical comparison: Wilcoxon non-parametric test, Cliff's Delta effect size (small/medium/large thresholds: 0.147/0.33/0.474).
- Discard warm-up iterations; benchmark in separate process (isolation).
- Single invocation must run ≥100ms for stable measurement (JIT/branch predictor).
- Stoppage criteria: Relative Confidence Interval Width (RCIW) <2-3%, KLD <threshold, or fixed sample count after stability.

### 16.3 Common JMH/BenchmarkDotNet bad practices to avoid
1. **Forgetting Blackhole** — JVM/CLR dead-code-eliminates unused results.
2. **Final fields constant-folded** — use `@State` / volatile.
3. **Warmup not isolated** — first runs ≠ steady state.
4. **Reusing input** — caches warm differently than prod.
5. **No process isolation** — GC/allocator state leaks between runs.

### 16.4 Macrobench / load test pass criteria
SLO-aligned (not "test completed"):

| Metric | Example SLO |
|---|---|
| p95 response time | ≤ 800ms @ 2000 concurrent |
| Error rate | < 0.5% |
| Throughput | ≥ 1200 req/s sustained |
| p99 alert at 150% of baseline (gives ~30s investigation window) |  |

Rule out infra noise first: DNS lookup >100ms = caching missing; packet retransmit >1% = network congestion.

### 16.5 Regression gating
Bench in CI for hot paths. Fail PR on >X% regression at p≤0.05 with effect size ≥ small. Track baseline drift over time.

### 16.6 Reporting honesty
- Distinguish model-dependent metrics from infrastructure metrics.
- Don't celebrate a metric that never changes across runs — it's testing hardcoded behavior, not your change.
- If only one data point: don't claim improvement.
- Always show: hardware, OS, runtime, build flags, dataset.

---

## 17. Root cause analysis

### 17.1 Methods (apply in order: simple → complex)

| Method | When | Output |
|---|---|---|
| **5 Whys** (Toyoda/Ohno) | linear single-cause incidents | causal chain, structural fix |
| **Fishbone (Ishikawa)** | multiple contributing factors | categorized causes (6 Ms or Code/Data/Config/Infra/Process/People) |
| **Fault Tree Analysis** | high-stakes, quantifiable risk | AND/OR gates, basic events with probabilities |
| **First-principles debugging** | unknown failure mode | reasoning from evidence (logs/traces/SoT), not opinion |
| **Causal DAG / counterfactual** | research-grade analysis | model checked against data |

### 17.2 5 Whys discipline
- Use **evidence** (logs/timestamps/code/SoT), not opinion.
- 3-7 Whys typical; stop only when at structural property.
- "Someone forgot" → keep going: why does the system rely on human memory?
- Multiple branches → switch to fishbone.

### 17.3 Anti-patterns
- Stopping at "human error" — ask why system permitted it.
- Stopping at first plausible cause — multiple can coexist.
- Correlation ≠ causation — verify mechanism not just timing.
- No follow-up — check at 30/60d: did fix hold?
- Blame — blameless is not optional; changes whether info surfaces.

### 17.4 RCA output template
1. Timeline (timestamps + evidence sources).
2. Symptoms.
3. Causal chain.
4. Root cause stated as **system property** ("the system allowed X because Y").
5. Action items: immediate fix / root-cause fix / detection improvement / prevention improvement, with owner + due date.
6. Re-verification plan + 30/60d check.

---

## 18. Chaos engineering

### 18.1 Four principles (Netflix)
1. **Build hypothesis around steady state** — define normal first.
2. **Vary real-world events** — node loss, network partition, latency spike, dependency 500.
3. **Run experiments in production** (with safeguards).
4. **Automate continuously**.

### 18.2 Tool ladder
| Tool | Scope |
|---|---|
| Chaos Monkey | random instance termination |
| Chaos Kong | full region failure |
| FIT (Failure Injection Testing) | per-request, per-dependency failure injection |
| ChAP (Chaos Automation Platform) | sticky-routed experiment with control + treatment cell, statistical rigor |
| Gremlin / LitmusChaos / Chaos Mesh | platform-agnostic |
| GameDay | scheduled human-in-loop drill |

### 18.3 Safe-experiment design (ChAP-style)
- Hypothesis stated.
- Smallest blast radius that yields statistical confidence.
- Control + treatment cells; routing isolates failure.
- Abort criteria + auto-rollback.
- Run in business hours with engineers alert (until automated).
- Integrate with CI/CD (Spinnaker / similar) to catch regressions.

### 18.4 What chaos finds (that staging misses)
- Mistuned retry policies (storms).
- CPU-intensive fallbacks (worse than the failure they handle).
- Circuit breaker × load balancer interactions.
- Cascading thread-pool exhaustion.
- Missing timeout in long-tail dep.

---

## 19. Operational practice — continuous hardening

### 19.1 Cadence
- **Per PR**: review checklist (§20.3); scanners green; FSV evidence for behavior changes.
- **Weekly**: dep auto-PRs merged; failed-login / anomalous-access review; cost anomaly review.
- **Monthly**: dead-code sweep; coverage + complexity trend; SLO/error budget review.
- **Quarterly**: backup restore drill; IR tabletop; threat model refresh; access review; chaos gameday.
- **Annually**: full arch review; DR exercise; 3rd-party pentest; capacity plan refresh.

### 19.2 Capability extraction (per component)
Ask four:
1. What does this contribute (user-visible)?
2. What does the system need from it (SLA / throughput / accuracy / extensibility)?
3. What capability was originally intended (vs drift)?
4. What's the max if optimized? Gap = roadmap.

### 19.3 The "is there a more elegant way" pause
For non-trivial changes: pause before merging. If a fix feels hacky → understand the problem, replace with the elegant version. Skip for simple obvious fixes (don't over-engineer).

### 19.4 Boy-Scout per touch (§11.6)
Compounded over time = the dominant debt-repayment mechanism.

---

## 20. Master checklists (copy-paste)

### 20.1 Daily / per-PR security & quality gate
- [ ] No secrets in diff (gitleaks clean)
- [ ] No new high/critical CVEs (SCA clean)
- [ ] No new SAST findings above threshold
- [ ] Tests added/updated
- [ ] **FSV evidence attached** for feature changes
- [ ] ≥3 edge cases tested (§4.5)
- [ ] No new TODO/FIXME without issue link
- [ ] No magic numbers/strings without named constant
- [ ] No `console.log`/`print`/debug stmts left in
- [ ] Errors handled (no bare `catch(e){}` / `except: pass`)
- [ ] Inputs validated; outputs encoded
- [ ] DB changes reversible / forward-compatible
- [ ] Observability signals added where state changed
- [ ] Docs/runbooks updated
- [ ] No regression in perf bench >X% (CI gate)
- [ ] SBOM regenerated; signed; provenance attested

### 20.2 Architecture review (per service)
- [ ] Single clear responsibility documented
- [ ] Public API spec'd; backward-compat policy explicit
- [ ] Owner + on-call documented
- [ ] Data ownership: every table/topic owned by exactly one service
- [ ] No shared write access to another service's data
- [ ] Failure modes per external dep: timeout/retry/CB/fallback/bulkhead/idempotency
- [ ] Latency budget met at p95/p99
- [ ] Horizontal scaling demonstrated
- [ ] Capacity headroom ≥2× peak
- [ ] Deploy: canary or B/G; rollback <5min
- [ ] Observability: RED + structured logs + traces + SLO-aligned alerts
- [ ] Security: authN+authZ every endpoint, secrets in vault, least priv
- [ ] Backup+restore drill within 90d
- [ ] Runbook for top 5 alerts

### 20.3 Code review
- [ ] Behavior matches spec
- [ ] Names communicate intent
- [ ] Functions ≤~30 lines; cyclomatic ≤10
- [ ] No duplication of existing utilities
- [ ] No dead code added
- [ ] Errors propagated meaningfully; no silent swallowing
- [ ] Edge cases covered by tests
- [ ] No race conditions in shared state
- [ ] No N+1 queries; pagination on lists
- [ ] No PII/secrets in logs
- [ ] No breaking API changes w/o versioning plan
- [ ] No new deps w/o justification + license + SBOM impact

### 20.4 Database hardening
- [ ] Private network only
- [ ] TLS in transit; encryption at rest with managed keys
- [ ] App user has min grants; separate users migration/reporting
- [ ] Constraints PK/NOT NULL/CHECK/FK/UNIQUE per business invariant
- [ ] Indexes match query patterns; no unused indexes
- [ ] Slow query log on, reviewed
- [ ] Backup encrypted, off-site, restore tested ≤90d ago
- [ ] Audit log for DDL + privileged ops
- [ ] Connection pooling; no app-side conn leaks
- [ ] Migration: expand-contract, reversible, online for big tables
- [ ] No N+1 in API response paths
- [ ] EXPLAIN ANALYZE clean for all top 20 queries

### 20.5 Pre-production deploy
- [ ] All tests green
- [ ] FSV evidence attached
- [ ] Migration plan + rollback plan reviewed
- [ ] Canary: percentage, duration, success metrics
- [ ] Alerts in place for new behavior
- [ ] Feature flag default correct
- [ ] Comms sent (if needed)
- [ ] Rollback button verified (not assumed)
- [ ] Build attested (SBOM + signature + SLSA provenance)

### 20.6 IR readiness
- [ ] On-call schedule current; pager tested
- [ ] Runbooks current for top 10 alerts
- [ ] Forensic snapshot capability tested
- [ ] Customer comms templates pre-approved
- [ ] Regulator comms templates pre-approved (if regulated)
- [ ] Tabletop in last 6mo
- [ ] Last-incident action items closed

### 20.7 ML / AI agent system
- [ ] Drift monitoring on inputs (PSI/KS), outputs (KS), calibration (ECE)
- [ ] Performance lane (confirmed labels) + proxy lane (ECE, OOD)
- [ ] Model version + system prompt hash logged per inference
- [ ] Rollback path to prior model tested
- [ ] Shadow / canary path for new model
- [ ] Eval suite: gold + adversarial + drift-set; run on every bump
- [ ] Prompt injection defense: filter + guard + response verify
- [ ] Tool calls scoped (least privilege)
- [ ] PII redaction in/out
- [ ] Cost-per-prediction tracked
- [ ] Per-segment fairness check
- [ ] Audit log of every prompt + tool call + response

### 20.8 FinOps quick-win sweep
- [ ] Tagging coverage ≥95% (owner/env/data-class/cost-center)
- [ ] Compute Optimizer / Azure Advisor / GCP Recommender reviewed in last 30d
- [ ] Unattached EBS / orphaned snapshots / idle ELB / no-traffic Lambda deleted
- [ ] Non-prod auto-shutdown after hours
- [ ] EBS gp2 → gp3 migrated
- [ ] Storage lifecycle policies on S3/GCS/Blob (IA / Glacier / archive)
- [ ] Savings Plans / RIs covering steady-state baseline (after rightsizing)
- [ ] Spot for fault-tolerant workloads
- [ ] CDN reducing egress
- [ ] OpenCost / Kubecost showing per-namespace cost
- [ ] Cost anomaly alerts configured

---

## 21. References (canon — defer to these over blogs)

### 21.1 Security
- OWASP Top 10:2025 — https://owasp.org/Top10/
- OWASP ASVS 5.0 — https://owasp.org/www-project-application-security-verification-standard/
- OWASP Cheat Sheets — https://cheatsheetseries.owasp.org/
- CIS Benchmarks — https://www.cisecurity.org/cis-benchmarks
- CIS Critical Security Controls v8 — https://www.cisecurity.org/controls
- NIST CSF 2.0 — https://www.nist.gov/cyberframework
- NIST SP 800-53 / 800-171 / 800-63B — https://csrc.nist.gov/publications
- DISA STIGs — https://public.cyber.mil/stigs/
- CISA Secure by Design — https://www.cisa.gov/securebydesign
- ASD Essential Eight — https://www.cyber.gov.au/
- NSA/CISA K8s Hardening Guidance

### 21.2 Supply chain
- SLSA spec — https://slsa.dev/
- Sigstore — https://www.sigstore.dev/
- SBOM specs: CycloneDX (https://cyclonedx.org/) and SPDX (https://spdx.dev/)
- in-toto — https://in-toto.io/

### 21.3 Architecture & code quality
- Fowler — *Refactoring* (smells + refactorings)
- Feathers — *Working Effectively with Legacy Code* (characterization tests, seams)
- Martin — *Clean Architecture*
- Ousterhout — *A Philosophy of Software Design*
- Ford/Parsons/Kua — *Building Evolutionary Architectures*
- Microsoft Application Architecture Guide
- AWS Well-Architected — https://aws.amazon.com/architecture/well-architected/
- Azure Well-Architected — https://learn.microsoft.com/en-us/azure/well-architected/
- Google SRE Books — https://sre.google/books/
- SonarQube rules — https://rules.sonarsource.com/

### 21.4 Reliability / SRE / resilience
- Google SRE Book + Workbook — https://sre.google/
- Netflix TechBlog Chaos Engineering series — https://netflixtechblog.com/
- Principles of Chaos Engineering — https://principlesofchaos.org/
- Hystrix / Resilience4j / Polly docs

### 21.5 Performance
- Brendan Gregg — *Systems Performance*
- Scott Oaks — *Java Performance*, 2nd ed.
- google/benchmark — https://github.com/google/benchmark
- JMH — https://github.com/openjdk/jmh
- BenchmarkDotNet — https://github.com/dotnet/BenchmarkDotNet

### 21.6 Testing / FSV
- Meszaros — *xUnit Test Patterns*
- Humble & Farley — *Continuous Delivery*
- ISTQB Foundation (BVA, ECP)
- Hypothesis docs; fast-check docs; OSS-Fuzz; ClusterFuzz; AFL++

### 21.7 RCA / human factors
- Lean 5 Whys — https://www.lean.org/lexicon-terms/5-whys/
- ASQ Five Whys — https://asq.org/quality-resources/five-whys
- Dekker — *The Field Guide to Understanding Human Error*; *Drift into Failure*
- Toyota Production System literature (Ohno)

### 21.8 ML / AI
- *Designing Machine Learning Systems* — Chip Huyen
- *Reliable Machine Learning* — Cathy Chen et al.
- Azure ML monitoring docs — https://learn.microsoft.com/azure/machine-learning/concept-model-monitoring
- Evidently AI / NannyML / WhyLogs docs
- OWASP Top 10 for LLM Applications — https://owasp.org/www-project-top-10-for-large-language-model-applications/
- MLCommons / MLPerf benchmarks
- Anthropic responsible scaling policy + model cards
- AgentDojo / TensorTrust / Open Prompt Injection benchmarks

### 21.9 FinOps
- FinOps Foundation — https://www.finops.org/
- AWS Cost Optimization Hub
- Azure Cost Management docs
- OpenCost — https://www.opencost.io/

---

## 22. The single rule (again, on purpose)

> **A return value is a claim. The Source of Truth is the verdict. Read the verdict.**

Every section here is built around that discipline. Scanners lie. Tests pass on stale data. Logs go missing. Benchmarks lie under DCE. Models lie when calibration drifts. The row in the database — or its absence — does not lie.

**Harden = make the system harder to break, easier to understand, faster to fix, cheaper to run, and provably correct at the SoT, every time, forever.**
