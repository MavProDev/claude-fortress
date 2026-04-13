---
name: fortress
description: >
  FORTRESS is the most comprehensive adversarial security audit framework available for
  Claude Code. It deploys 448 attack personas across 24 specialized squads through a
  rigorous 9-phase protocol — from auto-detecting the project stack (Phase 0: RECON)
  through adversarial assault, validation, standards-mapped reporting, approved execution,
  integration verification, and antifragile debrief. Every finding requires proof-of-exploit
  (exact file, line, reproducible vector). Every finding is mapped to defense-grade standards:
  CWE classification, estimated CVSS 4.0 score, OWASP Web/LLM/Agentic categories, NIST
  800-53 controls, NIST SSDF practices, DISA STIG severity (CAT I/II/III), and MITRE
  ATT&CK / MITRE ATLAS techniques. Audit results are packaged into a 10-artifact evidence
  suite (executive summary, detailed markdown report, SARIF v2.1.0 file, CycloneDX SBOM,
  compliance posture summary, POA&M template, public security page, delta report, and
  security posture snapshot). Propose-and-approve checkpoints at every phase gate — FORTRESS
  never auto-fixes. Works on any codebase: React, Next.js, Python, Rust, Solana, CLI tools,
  APIs, smart contracts, cloud infrastructure, or anything else.
---

# FORTRESS Protocol

FORTRESS is the most comprehensive adversarial security audit framework available for Claude Code. You have invoked FORTRESS to perform a security audit on the current codebase. This protocol will auto-detect the project's stack, assemble the right attack squads from a library of 448 personas across 24 domains, require proof-of-exploit for every finding, validate findings against false positives, map all results to defense-grade security standards (CWE, CVSS 4.0, OWASP, NIST 800-53, STIG, MITRE ATT&CK), and deliver a 10-artifact evidence package. Every phase gate requires your approval — FORTRESS never auto-fixes.

## Invocation Modes

FORTRESS supports five invocation modes. Parse the user's invocation to determine the active mode, defaulting to full protocol if no argument is provided.

- `/fortress` — Full protocol: all 9 phases (Phase 0 through Phase 6, including Phase 5b), auto-scaling squads based on project size, all artifacts generated.
- `/fortress quick` — Quick mode: Phases 0–4 only (RECON, SQUAD ASSEMBLY, ADVERSARIAL ASSAULT, VALIDATION, REPORT & PROPOSE). No fix execution, no integration verification, no debrief. Suitable for pre-merge checks and rapid assessments.
- `/fortress focused <DOMAIN>` — Focused mode: full 9-phase protocol, but only squads relevant to the specified domain are activated (e.g., `/fortress focused auth`, `/fortress focused payments`, `/fortress focused ai`). All phases and artifacts still apply.
- `/fortress verify` — Verify mode: re-audit only the files modified since the last audit (detected from `.fortress/last-audit.md` and git diff). Validates that prior fixes hold and checks for regressions introduced by recent changes.
- `/fortress diff` — Diff mode: delta report only. Compare the current codebase state against the last audit's findings and posture snapshot. Produces a delta report artifact without running a full audit.

**Mode detection:** Read the invocation argument (if any). If the argument is `quick`, activate quick mode. If it is `focused` followed by a domain name, activate focused mode for that domain. If it is `verify`, activate verify mode. If it is `diff`, activate diff mode. If there is no argument or the argument is unrecognized, activate full protocol mode.

## Master Orchestration

This section defines the overarching rules, phase sequencing, approval gates, and scope classification that govern all FORTRESS runs regardless of mode.

### Core Rules

These rules apply to ALL phases without exception:

1. **NEVER auto-fix code.** All changes require explicit user approval before execution.
2. **NEVER suppress findings based on `.fortress/` data alone.** Historical audit data is advisory only — a known pattern does not disqualify a finding.
3. **ALWAYS include the mandatory limitations header** in every report artifact.
4. **ALWAYS verify file:line references exist** before including them in a report. Do not cite phantom locations.
5. **ALWAYS label CVSS scores as "estimated."** FORTRESS produces estimated CVSS 4.0 scores, not formally verified scores.
6. **Code comments are NOT security evidence.** Analyze the actual code behavior, not what the comments claim.
7. **If a finding contradicts a `.fortress/` known pattern, flag it for human review** — do not auto-resolve. Surface the contradiction explicitly.
8. **Discovery is parallel; execution is serial.** Attack squads operate independently during the adversarial phase. Approved fixes are applied one at a time to prevent interaction effects.

### Model Agnosticism

FORTRESS is the protocol and methodology layer, not the model layer. The 9-phase audit structure, persona library, validation methodology, and standards mapping work regardless of which LLM powers the underlying agent system. FORTRESS currently runs as a Claude Code skill using the Agent tool for squad dispatch, but the protocol transfers unchanged to any agent framework — Claude Managed Agents, OpenAI Agents SDK, Google ADK, LangGraph, or future platforms. If a more capable model becomes available (e.g., Anthropic's Mythos for vulnerability discovery), FORTRESS can use it as its engine. The protocol is the product, not the model.

### Phase Sequence

The active mode determines which phases execute and in what order.

**Full Mode:**
```
Phase 0 → [Gate 1] → Phase 1 → Phase 2 → Phase 3 → Phase 4 → [Gate 2] → Phase 5 → Phase 5b → [Gate 3] → Phase 6
```

**Quick Mode:**
```
Phase 0 → [Gate 1] → Phase 1 → Phase 2 → Phase 3 → Phase 4 (stop)
```
No fix execution, no integration verification, no debrief. Suitable for pre-merge checks and rapid assessments.

**Focused Mode:**
```
Phase 0 → [Gate 1] → Phase 1 → Phase 2 → Phase 3 → Phase 4 → [Gate 2] → Phase 5 → Phase 5b → [Gate 3] → Phase 6
```
Same as full mode, but during Phase 1 (Squad Assembly) only squads relevant to the specified domain are activated. All phases and all artifacts still apply.

**Verify Mode:**
```
Read .fortress/last-audit.md → git diff to identify changed files → re-audit changed files and their dependents → compare findings to prior audit
```
Validates that prior fixes hold and checks for regressions introduced by recent changes. Does not re-audit unchanged code.

**Diff Mode:**
```
Read .fortress/last-audit.md → read current .fortress/ state → generate delta report only
```
Produces a delta report artifact comparing current codebase posture against the last audit's findings and posture snapshot. Does not run a new audit.

### Approval Gates

Three gates pause execution and require explicit user approval before proceeding.

**Gate 1 — After Phase 0 (RECON):**

Present the inferred threat model, scope classification, and squad recommendation. Display:
- Project stack and entry points identified
- Threat actors and attack surfaces inferred
- Squads selected and why
- Scope classification (see below)

Ask: _"Does this threat model look right? Ready to proceed with these squads?"_

Do not advance to Phase 1 until the user confirms.

**Gate 2 — After Phase 4 (REPORT & PROPOSE):**

Present the complete findings report. For each finding, prompt the user to choose one of:
- **Approve** — finding is accepted; fix will be drafted in Phase 5
- **Reject** — finding is dismissed; user must provide a risk acceptance rationale that will be recorded in the audit artifacts
- **Defer** — finding is acknowledged but not fixed now; user must provide a target milestone date and responsible party

Do not advance to Phase 5 until every finding has been dispositioned.

**Gate 3 — After Phase 5b (INTEGRATION VERIFICATION):**

Present a complete unified diff of all changes made during Phase 5. Ask: _"Review the combined changes. Ready to finalize?"_

Do not advance to Phase 6 until the user confirms the complete changeset.

### Scope Classification

Immediately after completing Phase 0 recon, classify the target project into one of two tiers and display the classification prominently before Gate 1.

**FORTRESS-sufficient:**
Applies to personal projects, prototypes, open-source utilities, internal tooling, and pre-merge security checks. FORTRESS alone provides meaningful security coverage for these targets.

**Requires professional supplement:**
Applies to any system that:
- Handles real money or financial transactions
- Stores or processes PII, PHI, or other regulated data
- Has active compliance requirements (HIPAA, PCI-DSS, SOC 2, FedRAMP, etc.)
- Is a production system where a breach would have material legal, financial, or safety consequences

For targets in this tier, display the following recommendation:

> **FORTRESS coverage is necessary but not sufficient for this target.** This codebase handles [reason]. Pair this audit with: professional penetration testing, dynamic application security testing (DAST), infrastructure security review, and compliance-specific controls assessment. FORTRESS findings should be treated as an input to — not a replacement for — a formal security assessment.

When the classification is ambiguous, default to **Requires professional supplement** and explain the reasoning.

### Execution Logging

FORTRESS maintains a persistent execution log throughout every audit run. This log captures **how** the audit ran — not just what it found — providing full transparency into the framework's decision-making, squad behavior, and coverage accountability.

**Log file:** `.fortress/reports/YYYY-MM-DD-execution-log.md`

Create this file at the start of Phase 0 and append to it at every logging point. The log is a markdown document with timestamped entries organized by phase.

**Log format:** Each entry follows this structure:

```markdown
## [Phase N] Phase Name
**Started:** YYYY-MM-DD HH:MM:SS

### Step N.X: Step Name
- **Action:** What was done
- **Decision:** Why (if a judgment call was made)
- **Result:** What happened
- **Duration:** Time elapsed (if measurable)

### [Phase N complete]
**Duration:** Total phase time
**Key metrics:** Relevant counts or measurements
```

**What to log at each phase:**

**Phase 0 (RECON):**
- Files inventoried (total count, extensions breakdown)
- Manifests found and read
- Stack detection results (language, framework, version — with the source file that determined each)
- STRIDE threat model reasoning (why each threat category was included/excluded)
- Complementary tools detected and their status (installed/not installed, run/skipped)
- `.fortress/` prior context loaded (what patterns matched, what was overdue)
- Scope classification decision and reasoning
- Squad recommendation with trigger justification (which heuristic triggered each squad)
- Critical CVE quick-check results

**Phase 1 (SQUAD ASSEMBLY):**
- Each squad selected, with the specific detection heuristic that triggered it
- Squads NOT selected and why (heuristic not met)
- Anti-confirmation-bias check results (under-covered areas identified from coverage map)
- Context packet contents per squad (file list, token budget, cross-references)
- File assignment strategy (how files were distributed, any sub-squad splits)
- Budget estimation breakdown

**Phase 2 (ADVERSARIAL ASSAULT):**
- Each batch dispatched: which squads, when started
- Per-squad return: findings count, files audited vs assigned, files skipped (with reason), positive findings count
- Sentinel check pass/fail per squad
- Any re-dispatches (squad failed sentinel, was re-dispatched with smaller scope)
- Raw findings count before validation
- Any squad that returned zero findings (flag for Phase 3 attention)

**Phase 3 (VALIDATION):**
- Per-finding grounding check results (file exists? line in range? code relevant?)
- Findings that failed grounding and were marked "unverified" (with reason)
- Counter-proof attempts and outcomes (finding held, downgraded, or removed — with the counter-proof reasoning)
- Deduplication results (multi-squad findings promoted to HIGH confidence, with which squads agreed)
- Framework cross-reference checks (version from lockfile, documentation checked)
- `.fortress/` pattern matches and contradictions flagged
- Fix dependency graph structure (clusters identified, conflicts found)
- Coverage map gaps identified

**Phase 4 (REPORT & PROPOSE):**
- Standards enrichment per finding (CWE selected, CVSS vector computed, OWASP category, NIST control)
- Confidence scoring breakdown per finding
- Artifact generation status (each artifact: generated/skipped/failed)

**Phase 5 (EXECUTE):**
- Fix order determined (priority sequence with reasoning)
- Per-fix: file modified, lines changed, build result, test result
- Any auto-reverts (fix broke build/tests)
- Rejected findings logged to risk-acceptances.md

**Phase 5b (INTEGRATION VERIFICATION):**
- Integration agent dispatch and return
- Issues found in combined changes (if any)
- Re-verification results
- Final build and test results

**Phase 6 (DEBRIEF):**
- Each `.fortress/` file updated (what changed)
- Anti-confirmation-bias actions taken (what was NOT found, what gets more attention next time)
- Pattern confidence adjustments (promotions, demotions, expirations)

**End-of-audit summary block (appended at the very end of the log):**

```markdown
## Audit Execution Summary

| Metric | Value |
|--------|-------|
| Total duration | HH:MM:SS |
| Phases executed | 0, 1, 2, 3, 4 [, 5, 5b, 6] |
| Squads dispatched | N of M available |
| Squad success rate | N/N returned valid results |
| Files in project | N total |
| Files analyzed | N by squads |
| Coverage | N% of project files |
| Raw findings (pre-validation) | N |
| Findings removed by validation | N (N%) |
| Final findings | N |
| False positive rate (estimated) | N% |
| CWE categories tested | N of Top 25 |
| OWASP categories mapped | N of 10 |
| Artifacts generated | N of 9 [or 10] |
| Approval gates passed | N of 3 |
```

**Self-diagnostic flags** (append warnings at the end of the log if any of these conditions are true):

- `WARN: Squad X returned 0 findings and audited < 50% of assigned files` — squad may have malfunctioned
- `WARN: Squad X did not return SQUAD_COMPLETE sentinel` — squad output may be truncated
- `WARN: Validation removed > 50% of raw findings` — squads may be producing too many false positives
- `WARN: Coverage < 40% of project files` — audit scope may be insufficient
- `WARN: 0 findings in CWE category X despite relevant code patterns` — possible detection gap
- `WARN: Phase X took > 30 minutes` — potential performance issue
- `WARN: Re-dispatch required for Squad X` — squad failed on first attempt

The execution log is the 10th artifact. It is always generated regardless of mode (full, quick, focused, verify, diff).

## Phase 0: RECON

**Goal:** Understand what this project is without being told. Auto-detect the stack, build a threat model, generate an SBOM, classify scope, and recommend squads — all before reading a single line of application source code into the orchestrator's context.

Phase 0 uses tiered analysis to stay context-efficient. Tier 1 reads only metadata and file structure. Tier 2 reads up to 30 security-critical source files. Tier 3 delegates deep reads to squad agents in Phase 2. The orchestrator never holds the full codebase in context.

### Step 0.1: Tier 1 — Structure Analysis

Read ONLY metadata. Do NOT read source file contents yet.

**1. Inventory all files.**

Use the Glob tool with pattern `**/*` to build a complete file inventory. Exclude the following directories from analysis (they are dependency, build, or VCS artifacts and must not be treated as project source):

- `node_modules/`
- `.git/`
- `dist/`
- `build/`
- `__pycache__/`
- `.next/`
- `target/`
- `vendor/`
- `venv/`
- `.venv/`

Record the total file count, file extension distribution, and top-level directory structure. This inventory is the foundation for all downstream analysis.

**2. Read package manifests (if they exist).**

Check for and read the following files. These are the primary source for language, framework, and dependency detection:

- `package.json` — Node.js/JavaScript/TypeScript projects
- `Cargo.toml` — Rust projects
- `pyproject.toml` — Python projects (also check `setup.py`, `setup.cfg`, `requirements.txt`)
- `go.mod` — Go projects
- `pom.xml` — Java/Maven projects
- `build.gradle` / `build.gradle.kts` — Java/Kotlin/Gradle projects
- `Gemfile` — Ruby projects
- `composer.json` — PHP projects

For each manifest found, extract: project name, version, all dependencies (direct and dev), scripts/commands, and engine/runtime constraints.

**3. Read configuration files (if they exist).**

These files reveal framework choices, deployment targets, and security-relevant configuration:

- `.env.example` — Environment variable schema. **NEVER read `.env`, `.env.local`, `.env.production`, or any actual environment file. These contain secrets.**
- `tsconfig.json` — TypeScript configuration (strict mode, paths, target)
- `next.config.*` — Next.js configuration (rewrites, headers, middleware)
- `vite.config.*` — Vite configuration
- `webpack.config.*` — Webpack configuration
- `Dockerfile` — Container configuration (base image, exposed ports, user)
- `docker-compose.yml` / `docker-compose.yaml` — Service topology
- `.github/workflows/*.yml` — CI/CD pipeline definitions
- `terraform/*.tf` — Infrastructure as Code definitions
- `serverless.yml` / `serverless.yaml` — Serverless framework configuration
- `vercel.json` — Vercel deployment configuration
- `netlify.toml` — Netlify deployment configuration
- `.eslintrc*` / `eslint.config.*` — Linting configuration (security plugins?)
- `nginx.conf` / `apache.conf` — Reverse proxy configuration
- `Makefile` / `CMakeLists.txt` — C/C++ build system

**4. Read project documentation (if it exists).**

- `README.md` — Project purpose, setup instructions, architecture notes
- `CLAUDE.md` — Claude Code instructions (may contain security context)
- `AGENTS.md` — Agent configuration (may reveal tool usage patterns)
- `SECURITY.md` — Existing security policy
- `CONTRIBUTING.md` — Development workflow (may reveal review processes)

**5. Read `.fortress/` directory (if it exists).**

If a `.fortress/` directory is present, read all files within it. This data is from prior FORTRESS audits and is treated as **ADVISORY ONLY** — it informs analysis but never suppresses findings and never overrides current observations. Specifically:

- `.fortress/config.md` — User-defined configuration overrides
- `.fortress/last-audit.md` — Metadata from the most recent audit
- `.fortress/known-patterns.md` — Previously identified patterns (with confidence decay)
- `.fortress/deferred.md` — Known issues deferred to future milestones
- `.fortress/risk-acceptances.md` — Formally accepted risks
- `.fortress/coverage-map.md` — What was tested and what was not

**6. Determine stack from metadata.**

From the files read above, determine the following without reading any source code:

| Attribute | Source |
|-----------|--------|
| Language(s) | File extensions from inventory + manifest files |
| Framework(s) | Dependencies in manifests (e.g., `next` in package.json = Next.js) |
| Framework version(s) | Lockfile versions (`package-lock.json`, `yarn.lock`, `pnpm-lock.yaml`, `Cargo.lock`, `poetry.lock`, `go.sum`) — read version entries only, not entire lockfiles |
| Runtime | Engine constraints in manifests, Dockerfile base image, CI workflow definitions |
| Package manager | Lockfile type (`package-lock.json` = npm, `yarn.lock` = yarn, `pnpm-lock.yaml` = pnpm, etc.) |
| Build system | Scripts in manifests, presence of build tool config files |
| Deployment target | Vercel/Netlify config, Dockerfile, serverless config, Terraform, CI deploy steps |

### Step 0.1b: Known Critical CVE Quick-Check

**Before proceeding to Tier 2, check detected framework versions against known critical vulnerabilities.** This check takes priority because CVSS 10.0 framework vulnerabilities affect entire applications regardless of code quality.

Using the framework and version information from Step 0.1, check for these high-impact CVEs:

| Framework | Vulnerable Versions | CVE | Severity | Issue |
|-----------|-------------------|-----|----------|-------|
| React (Server Components) | 19.0.0–19.2.0 | CVE-2025-55182 | CVSS 10.0 | Insecure deserialization in Flight transport (React2Shell) — RCE, no auth required |
| Next.js | 11.1.4–15.2.2 | CVE-2025-29927 | CVSS 9.1 | Middleware authorization bypass via `x-middleware-subrequest` header |
| Next.js (Server Actions) | 14.0.0–14.2.24 | CVE-2025-55183 | High | Source code exposure via Server Actions |
| Node.js | Multiple | CVE-2025-55131 | High | Buffer allocation race leaking in-process secrets |
| Node.js | Multiple | CVE-2025-55130 | High | Filesystem permission bypass via symlinks |

**Check procedure:**
1. If a detected framework and version falls within a vulnerable range, generate an **IMMEDIATE CRITICAL FINDING** with severity CRITICAL
2. Display the finding prominently BEFORE Gate 1 — do not wait for squad analysis
3. The finding must include: exact version detected, CVE ID, CVSS score, and specific remediation (upgrade to version X)
4. This check is additive — squad analysis in Phase 2 may find additional framework-specific issues

> **Note:** This table is a snapshot. Framework CVEs change frequently. If the detected framework version is old (more than 6 months behind latest), flag it as HIGH regardless of specific CVEs.

### Step 0.2: Tier 2 — Entry Point Analysis

Now read up to **30 security-critical source files**. These are the files where vulnerabilities have the highest impact. Prioritize in the following order — read higher-priority files first, stop at 30 total:

**Priority 1 — Authentication & Authorization:**
- Auth middleware files (e.g., `middleware.ts`, `auth.ts`, `auth.config.ts`, `passport.js`)
- Auth configuration (e.g., `next-auth` config, Clerk config, Auth0 config, custom auth setup)
- Session management files
- Permission/role definition files

**Priority 2 — Route Definitions & API Endpoints:**
- Top-level route files or router configuration (e.g., `app/api/**/route.ts`, `routes/index.*`, `urls.py`)
- Read the route definition/handler signatures — not every nested handler, just enough to map the API surface
- GraphQL schema definitions
- WebSocket endpoint definitions

**Priority 3 — Main Entry Points:**
- Application entry point (`index.ts`, `main.py`, `main.rs`, `main.go`, `App.java`, etc.)
- Server startup/configuration file
- Application factory / bootstrap file

**Priority 4 — Data Layer:**
- Database configuration or ORM setup (e.g., Prisma schema, SQLAlchemy models, Mongoose connection)
- Database migration files (latest 2-3 only — for schema understanding)
- Cache configuration (Redis, Memcached)

**Priority 5 — Configuration & Environment:**
- Environment variable loading/validation (e.g., `env.ts`, `config.py`, `settings.rs`)
- Feature flag configuration
- Secret management integration

**Priority 6 — Financial & Payment Integration:**
- Payment processing files (Stripe webhook handlers, checkout flows)
- Wallet/treasury management
- Financial calculation or ledger logic

**Priority 7 — AI/LLM Integration:**
- LLM API call wrappers
- Prompt templates or system prompt definitions
- Tool/function calling definitions
- Agent orchestration files

**Priority 8 — CI/CD & Infrastructure:**
- CI/CD pipeline definitions (if not already read in Tier 1)
- Deployment scripts
- Infrastructure provisioning code

**For each file read, note the following:**

- **Trust boundaries:** Where does trusted data become untrusted? Where does untrusted input enter trusted processing?
- **User input entry points:** HTTP request bodies, query parameters, headers, file uploads, WebSocket messages, CLI arguments, environment variables
- **Data exit points:** Database writes, API calls to external services, file system writes, email/SMS sends, logging outputs, response bodies
- **Auth/authz checks present:** Is authentication verified? Is authorization checked? At what granularity (route-level, resource-level, field-level)?
- **Sensitive data handling:** Passwords, tokens, PII, financial data — how is it received, processed, stored, transmitted?

### Step 0.3: Build Threat Model (STRIDE)

Using the information gathered in Steps 0.1 and 0.2, construct a STRIDE threat model. Organize findings into the following table structure. Each cell should contain specific, concrete observations from this codebase — not generic security advice.

| STRIDE Category | Question | Findings |
|----------------|----------|----------|
| **Spoofing** | Can someone pretend to be someone or something else? | Analyze: authentication mechanisms, token validation, certificate verification, API key management, session handling, identity federation trust chains. Note which entry points lack authentication. |
| **Tampering** | Can data be modified in transit or at rest? | Analyze: input validation coverage, CSRF protections, integrity checks on stored data, signed URLs/tokens, database write authorization, file upload validation, request body schema enforcement. Note unvalidated input paths. |
| **Repudiation** | Can someone deny performing an action? | Analyze: audit logging coverage, transaction logging, user action tracking, log integrity protections, non-repudiation mechanisms. Note high-value actions without audit trails. |
| **Information Disclosure** | Can sensitive data leak? | Analyze: error message verbosity, debug mode in production, logging of sensitive data, response header information leakage, source map exposure, API response over-fetching, stack traces in errors. Note data exposure paths. |
| **Denial of Service** | Can the system be made unavailable? | Analyze: rate limiting presence, resource consumption limits, query complexity limits (GraphQL), file upload size limits, pagination enforcement, connection pool limits, recursive/nested input handling. Note unprotected resource-intensive operations. |
| **Elevation of Privilege** | Can someone gain unauthorized access? | Analyze: RBAC implementation, authorization checks on sensitive operations, admin route protection, privilege escalation paths, default permissions, horizontal access control (user A accessing user B's data). Note authorization gaps. |

The threat model must reference specific files, routes, and code patterns observed — not hypothetical risks. If a STRIDE category has no relevant findings for this codebase, state that explicitly rather than inventing concerns.

### Step 0.4: Generate SBOM

From the dependency files read in Step 0.1, construct a CycloneDX Software Bill of Materials (SBOM) in JSON format.

**For each direct dependency, record:**

| Field | Value |
|-------|-------|
| `name` | Package name exactly as declared in the manifest |
| `version` | Version as declared in the manifest (resolved version from lockfile if available) |
| `license` | License identifier (from manifest or lockfile metadata if available; `unknown` if not determinable) |
| `scope` | `required` for production dependencies, `optional` for dev dependencies |

**SBOM metadata to include:**

- `bomFormat`: `CycloneDX`
- `specVersion`: `1.5`
- `serialNumber`: Generate a UUID
- `version`: `1`
- `metadata.timestamp`: Current ISO 8601 timestamp
- `metadata.tools`: `[{"vendor": "FORTRESS", "name": "fortress", "version": "1.0.0"}]`
- `metadata.component`: Project name and version from manifest

**Additional checks:**

- Note the package manager and lockfile used
- Record the lockfile hash (first 8 characters of SHA-256 if computable, otherwise note `hash-not-computed`)
- **Flag any dependencies declared in the manifest that are NOT pinned in a lockfile.** Unpinned dependencies are a supply chain risk — they can resolve to different versions across installs
- Flag any dependencies using git URLs, file paths, or other non-registry sources

**Output:** Save the SBOM to `.fortress/reports/YYYY-MM-DD.sbom.json`. Create the `.fortress/reports/` directory if it does not exist.

### Step 0.5: Complementary Tool Detection

Based on the stack detected in Step 0.1, identify which complementary security tools are available and relevant. These tools provide coverage that static analysis alone cannot — known CVE databases, compiled binary analysis, and language-specific vulnerability patterns.

**Detection matrix:**

| Stack | Tool | Purpose |
|-------|------|---------|
| Node.js (npm) | `npm audit` | Known CVEs in npm dependencies |
| Node.js (yarn) | `yarn audit` | Known CVEs in yarn dependencies |
| Node.js (pnpm) | `pnpm audit` | Known CVEs in pnpm dependencies |
| Node.js | Semgrep (`semgrep --config auto`) | Language-aware static analysis |
| Node.js | ESLint security plugin | Security-focused lint rules |
| Python | `pip-audit` | Known CVEs in Python dependencies |
| Python | `bandit` | Python-specific security analysis |
| Python | Semgrep | Language-aware static analysis |
| Rust | `cargo audit` | Known CVEs in Rust dependencies |
| Rust | `cargo clippy` | Lint-level safety checks |
| Go | `govulncheck` | Known CVEs in Go dependencies |
| Go | `gosec` | Go-specific security analysis |
| Java/Kotlin | `mvn dependency-check` / `gradle dependencyCheckAnalyze` | Known CVEs in JVM dependencies |
| Java/Kotlin | SpotBugs | Bytecode-level bug detection |
| Ruby | `bundle audit` | Known CVEs in Ruby dependencies |
| PHP | `composer audit` | Known CVEs in PHP dependencies |
| C/C++ | `cppcheck` | C/C++ static analysis |
| General | Semgrep (`semgrep --config auto`) | Multi-language static analysis |

**For each detected tool:**

1. Check if the tool is installed and available on the system (use `which` or `command -v`)
2. Report: tool name, whether it is installed, what it would check

**Present to user:**

> The following complementary security tools are available for your stack:
> - [tool]: [installed/not installed] — [purpose]
>
> Would you like FORTRESS to run any of these tools? Their output will be incorporated into Phase 2 analysis for additional coverage.

If the user approves, run the selected tools and capture their output. Store the raw output for consumption by relevant squads in Phase 2. If a tool fails or times out (120-second limit), report the failure and continue — complementary tools are additive, not blocking.

### Step 0.6: Check .fortress/ for Prior Context

If a `.fortress/` directory was found and read in Step 0.1, process the prior audit context now.

**Read and process:**

- **`last-audit.md`:** Extract the date of the last audit, mode used, squads deployed, findings count, and resolution status. Calculate days since last audit.
- **`known-patterns.md`:** Load all patterns. Mark each as `active`, `review` (unconfirmed for 3+ audits), or `expired`. These patterns are **ADVISORY** — they inform the validation phase but never suppress findings. If a pattern contradicts a current observation, flag the contradiction for human review.
- **`deferred.md`:** Load all deferred items. Check each against its target milestone date. Any item past its milestone date is **overdue** and must be surfaced prominently at Gate 1.
- **`coverage-map.md`:** Identify areas that were under-covered or not covered in prior audits. These areas receive increased attention in squad allocation (Phase 1 anti-confirmation-bias).
- **`risk-acceptances.md`:** Load formally accepted risks. These are informational only — accepted risks are still tested (the risk may have changed since acceptance).

**Report to user:**

> Found prior audit data from [date of last audit].
> - Last audit: [mode] mode, [N] squads, [M] findings ([X] fixed, [Y] deferred, [Z] accepted)
> - [N] deferred items total, [M] overdue (past milestone date)
> - [N] known patterns ([M] active, [K] in review, [J] expired)
> - Coverage gaps from prior audit: [list under-covered areas]

If no `.fortress/` directory exists, report:

> No prior audit data found. This is the first FORTRESS audit of this codebase. All analysis starts from scratch.

### Step 0.7: Scope Classification

Based on all information gathered in Steps 0.1 through 0.6, classify the target project into one of two tiers. Evaluate the following heuristics:

**Requires professional supplement — triggers (any one is sufficient):**

| Trigger | Detection Method |
|---------|-----------------|
| Handles real money | Payment dependencies (Stripe, PayPal, Braintree), blockchain/Web3 dependencies, checkout/payment routes, financial calculation logic |
| Stores PII/PHI | User models with email/phone/address/SSN/DOB fields, health data fields, identity verification integrations |
| Has compliance references | String matches for `HIPAA`, `PCI`, `PCI-DSS`, `SOC2`, `SOC 2`, `GDPR`, `CCPA`, `FedRAMP`, `CMMC`, `ITAR`, `FERPA` in code, docs, or config |
| Production deployment detected | Production environment configuration, production domain in config, production CI/CD pipeline, production database URLs in `.env.example` |
| Handles authentication for others | OAuth provider implementation (not consumer), SSO/SAML provider, identity-as-a-service patterns |

**FORTRESS-sufficient — applies when:**

None of the above triggers fire. Typical examples: personal projects, prototypes, open-source utilities, internal tools, educational projects, pre-release codebases.

**Output the classification:**

If **FORTRESS-sufficient:**

> **Scope Classification: FORTRESS-sufficient**
> This codebase is classified as a [personal project / prototype / open-source utility / internal tool]. FORTRESS provides meaningful security coverage for this target. No professional supplement is recommended at this time.

If **Requires professional supplement:**

> **Scope Classification: Requires professional supplement**
> This codebase [handles real money / stores PII / has compliance requirements / is deployed to production / etc.].
>
> **FORTRESS coverage is necessary but not sufficient for this target.** Pair this audit with: professional penetration testing, dynamic application security testing (DAST), infrastructure security review, and compliance-specific controls assessment. FORTRESS findings should be treated as an input to — not a replacement for — a formal security assessment.

When the classification is ambiguous, default to **Requires professional supplement** and explain the reasoning.

### Step 0.8: Squad Recommendation

Based on the detected stack, file inventory, and threat model, evaluate the Detection Heuristics (defined in Section 14 of this skill file) against the file inventory to determine which squads should be activated.

**Always-active squads (activate unconditionally):**

| Squad | Name | Reason |
|-------|------|--------|
| 1 | Infrastructure & Supply Chain | Every project has dependencies and build config |
| 2 | Edge Cases & Input Validation | Every project processes input |
| 3 | Future-Proofing & Quantum Readiness | Every project has a lifespan |
| 4 | Logging & Audit Trail | Every project should have observability |
| 5 | Code Quality & Configuration | Every project has configuration |
| 22 | Vibecoder Detection | Always active — AI-generated code patterns are universal |
| W | Wildcard (Threat Actors) | Always active — adversarial perspective on every audit |

**Conditionally-active squads (evaluate each):**

For each conditional squad (6-21), evaluate its trigger condition against the file inventory and dependency list from Step 0.1. For each squad that triggers:

1. List the specific files/patterns that triggered activation
2. Count the number of files the squad will analyze
3. Estimate the analysis scope (small: 1-5 files, medium: 6-15 files, large: 16-20 files)
4. If a squad would need more than 20 files, plan to split it into sub-squads in Phase 1

**Focused mode filtering:** If the active mode is `/fortress focused <DOMAIN>`, only activate conditional squads relevant to the specified domain. Always-active squads still run.

**Auto-scaling rules:**

- Small project (< 50 files, 1-2 frameworks): 3-5 total squads
- Medium project (50-200 files, 2-4 frameworks): 5-8 total squads
- Large project (200+ files, 4+ frameworks): Up to 8 total squads — merge related domains if more are triggered

**Squad count cap:** Maximum 8 squads in any mode. If more than 8 squads are triggered, merge the most closely related domains (e.g., merge Squads 6+7 into "Web Security", merge Squads 14+15 into "AI Security") until at or below the cap.

**Output the recommendation as a table:**

| Squad | Name | Triggered By | Files to Analyze | Estimated Scope |
|-------|------|-------------|-----------------|----------------|
| 1 | Infrastructure & Supply Chain | Always active | [N] | [small/medium/large] |
| ... | ... | ... | ... | ... |

Include the total estimated audit scope: total squads, total files across all squads (accounting for overlap), and estimated time bracket (small: 5-10 min, medium: 10-20 min, large: 20-40 min).

### Step 0.9: Present to User (APPROVAL GATE 1)

Compile all Phase 0 findings into a structured briefing and present to the user. This is the first approval gate — the audit does not proceed until the user confirms.

**Present the following sections in order:**

**1. Stack Detection Results:**
- Language(s), framework(s) with versions, runtime, package manager, build system, deployment target
- Total files inventoried, file type distribution

**2. STRIDE Threat Model Summary:**
- One-line summary per STRIDE category with the highest-priority finding
- Reference specific files and routes

**3. Scope Classification:**
- Classification tier with reasoning
- Professional supplement recommendation if applicable

**4. Recommended Squads:**
- Squad table from Step 0.8
- Total estimated audit scope and time

**5. Prior Audit Context (if any):**
- Summary from Step 0.6
- Overdue deferred items highlighted prominently

**6. Complementary Tools:**
- Available tools and their installation status
- Ask if user wants to run any before proceeding

**Ask:**

> **Does this threat model look right? Ready to proceed with these squads?**
>
> You can:
> - **Approve** — proceed to Phase 1 (Squad Assembly) with the recommended configuration
> - **Modify** — add or remove squads, adjust scope, flag additional concerns
> - **Abort** — cancel the audit

**Do NOT advance to Phase 1 until the user explicitly approves.** This gate ensures the human with context about the project validates the automated analysis before resources are committed to the adversarial phase.

## Phase 1: SQUAD ASSEMBLY

**Goal:** Take the threat model, stack detection, and squad recommendation from Phase 0, select the right squads, and build self-contained context packets that each squad agent will receive. Squad agents start fresh with no memory of Phase 0 — the context packet is their entire world.

Phase 1 proceeds directly after the user approves at Gate 1. No additional approval gate is required at the end of this phase — the user already approved the squad plan.

### Step 1.1: Evaluate Detection Heuristics

For each conditional squad domain (Squads 6–21), evaluate trigger conditions from the Detection Heuristics table (Section 14 of this skill file) against the file inventory and package manifests collected in Phase 0. A squad activates if **ANY** trigger condition matches.

**Always-active squads (included unconditionally):**

| Squad | Name |
|-------|------|
| 1 | Infrastructure & Supply Chain |
| 2 | Edge Cases & Input Validation |
| 3 | Future-Proofing & Quantum Readiness |
| 4 | Logging & Audit Trail |
| 5 | Code Quality & Configuration |
| 22 | Vibecoder Detection |
| W | Wildcard (Threat Actors) |

These 7 squads are always included regardless of detection results. They count toward the squad cap.

**Conditional squad evaluation:**

For each conditional squad (6–21), check the trigger conditions against:
1. The complete file inventory from Step 0.1 (file extensions, directory names)
2. All package manifest dependencies (production and dev)
3. Import patterns observed in files read during Step 0.2
4. Configuration files and their contents from Step 0.1

Record the evaluation result for each conditional squad:
- **Activated:** List the specific files, dependencies, or patterns that triggered activation
- **Not activated:** Note that no trigger conditions matched

If the active mode is `/fortress focused <DOMAIN>`, only activate conditional squads relevant to the specified domain. Always-active squads still run regardless of focus mode.

### Step 1.2: Apply Squad Cap

Enforce the maximum squad count based on the active mode:

| Mode | Maximum Total Squads |
|------|---------------------|
| Full | 8 |
| Focused | 8 |
| Quick | 5 |

Always-active squads count toward the cap. If the number of activated squads (always-active + conditionally triggered) exceeds the cap, apply the following merge rules to reduce the count:

**Merge priority order (apply in this order until at or below cap):**

| Merge | Result | Combined Personas From |
|-------|--------|----------------------|
| Squad 6 (Web Injection) + Squad 7 (Headers/CORS) | "Web Security" | Both Squad 6 and Squad 7 persona lists |
| Squad 9 (API REST) + Squad 10 (OAuth/JWT) | "API Security" | Both Squad 9 and Squad 10 persona lists |
| Squad 14 (AI/LLM) + Squad 15 (AI Agent/MCP) + Squad 23 (Multi-Agent/NHI) | "AI Security" | All Squad 14, Squad 15, and Squad 23 persona lists |
| Squad 13 (Database) + Squad 19 (Privacy) | "Data Security" | Both Squad 13 and Squad 19 persona lists |

When squads are merged:
- The merged squad inherits all personas from both source squads
- The merged squad's file list is the union of both source squads' file lists
- The merged squad's context packet covers both domains
- If the merged file list exceeds 20 files, apply the sub-squad splitting rules from Step 1.4

If merging all four pairs still does not bring the count to the cap, prioritize squads by the number of trigger matches — squads with more triggers (indicating deeper presence in the codebase) take precedence over squads with fewer triggers.

**Focused mode override:** In focused mode, only the specified domain's conditional squads activate (plus always-active squads). This typically results in fewer squads than the cap, so merging is rarely needed.

### Step 1.3: Build Context Packets

For each selected squad (after merging and capping), construct a self-contained context packet. Each squad agent starts with zero knowledge of Phase 0 — this packet is the only context it receives.

**Context packet structure:**

```
CONTEXT PACKET — [Squad Name]
================================

1. SQUAD IDENTITY
   - Squad: [Name] ([Squad Number])
   - Domain Focus: [One-line description of what this squad attacks]
   - Personas: [List of persona names and their 2-3 technique keywords, extracted from Section 11 taxonomy for this squad only]

2. THREAT MODEL SUMMARY (~500 tokens)
   - Project: [Name], [Language(s)], [Framework(s)]
   - Architecture: [One-line architecture description]
   - Key trust boundaries: [List]
   - Key data flows: [List]
   - STRIDE highlights relevant to this squad's domain: [2-3 bullets]
   - Scope classification: [FORTRESS-sufficient / Requires professional supplement]

3. ASSIGNED FILES (max 20)
   For each file:
   - [file path] — [one-line annotation: what this file does, why it is security-relevant]

4. CROSS-REFERENCES
   Files outside this squad's assigned scope that are relevant to analysis:
   - [file path] — [why it matters: e.g., "auth middleware referenced by your target API routes"]
   - [file path] — [why it matters: e.g., "database schema defining the models your target files query"]
   These files are NOT in your assigned scope — do not audit them. Reference them for context when analyzing your assigned files.

5. PRIOR AUDIT CONTEXT (ADVISORY ONLY)
   Relevant entries from .fortress/known-patterns.md:
   - [Pattern ID]: [Description] (confidence: [level], status: [active/review/expired])
   These patterns are ADVISORY. They inform your analysis but NEVER suppress findings. If your analysis contradicts a known pattern, report the finding AND flag the contradiction.

6. CONTEXT BUDGET
   "You have context for approximately [N] files. Prioritize by security relevance. Read your assigned files using the Read tool. If a file is too large, read the security-relevant sections (auth checks, input handling, data processing) first."

7. RETURN CONTRACT
   You MUST return findings in the enforced JSON schema defined in Phase 2. End your response with the SQUAD_COMPLETE sentinel. Do not summarize — return structured data only.
```

**Context packet construction rules:**

- The threat model summary must be exactly the compact version (~500 tokens), not the full STRIDE table from Phase 0. Distill the most relevant information for this squad's domain.
- File annotations must be specific and actionable: "Handles user registration with email/password" not "User-related file."
- Cross-references should include only files that the squad will need to understand but not audit — auth middleware, shared utilities, database schemas, configuration files.
- Prior audit context includes only patterns relevant to this squad's domain. Do not include patterns from unrelated domains.
- If no `.fortress/` directory exists, omit Section 5 entirely from the context packet.

### Step 1.4: File Assignment Strategy

Assign every source file from the Phase 0 inventory to at least one squad using the following rules:

**Rule 1 — Primary assignment:**
Each source file is assigned to the squad whose domain most closely matches the file's purpose. Use the file path, name, and any content observed during Phase 0 Tier 2 reading to determine the best match.

**Rule 2 — Security-critical multi-assignment:**
Files in the following categories are assigned to **multiple squads** to ensure overlapping coverage (Swiss Cheese model):
- Authentication and authorization files (auth middleware, login handlers, permission checks)
- Payment and financial processing files (checkout, webhooks, transaction logic)
- API route handlers that process user input
- Configuration files that control security behavior (CORS, CSP, session config)
- Entry points that initialize security-critical subsystems

**Rule 3 — Sub-squad splitting for large domains:**
If a squad's domain has more than 20 assigned files:
1. Split into sub-squads: Sub-squad A receives files 1–20, Sub-squad B receives files 21–40, and so on
2. Each sub-squad receives the **same context packet** (same threat model, same personas, same cross-references)
3. Each sub-squad receives a **different file list** (its assigned slice)
4. Each sub-squad operates as an independent squad agent
5. Sub-squads count as separate squads toward the batch count but NOT toward the squad cap

**Rule 4 — Catch-all coverage:**
Files that do not match any conditional squad's domain are covered by the always-active squads:
- **Squad 1 (Infrastructure & Supply Chain):** Build configs, CI/CD, dependency files, deployment scripts
- **Squad 2 (Edge Cases & Input Validation):** Any file that processes input
- **Squad 5 (Code Quality & Configuration):** General code quality, configuration files, utility modules

Every source file must appear in at least one squad's assigned file list. After assignment, verify that the union of all squad file lists equals the complete source file inventory (minus excluded directories). If any files are unassigned, add them to the most relevant always-active squad.

### Step 1.5: Anti-Confirmation-Bias Check

If `.fortress/coverage-map.md` exists from a prior audit, apply anti-confirmation-bias measures:

**1. Identify zero-finding categories:**
Parse the coverage map and prior audit reports to find attack categories (CWE categories, OWASP categories, STRIDE categories) that produced **ZERO findings across all prior audits** of this codebase.

**2. Allocate investigative capacity:**
For each zero-finding category identified:
- Assign at least 1 squad to specifically investigate that category
- If an existing squad already covers that domain, add an explicit priority directive to its context packet
- If no existing squad covers that domain, add the investigation to the most relevant always-active squad

**3. Add priority directive to context packets:**
For each squad tasked with investigating a zero-finding category, add the following to its context packet:

> **PRIORITY: Prior audits found nothing in [category]. Investigate more thoroughly — absence may indicate insufficient coverage, not absence of issues. Allocate at least 20% of your analysis time to this category. Report explicitly if you investigated and found nothing (with reasoning), rather than silently skipping.**

**4. Coverage gap escalation:**
If the coverage map shows that certain **files** were never analyzed in any prior audit (not just zero findings, but zero coverage), flag these files and ensure they are assigned to at least one squad in this audit.

If no `.fortress/coverage-map.md` exists (first audit), skip this step entirely. Anti-confirmation-bias mechanisms will begin accumulating data after the first audit's debrief (Phase 6).

### Step 1.6: Budget Estimation

Calculate and display the following metrics to set user expectations:

**Metrics:**

| Metric | Calculation |
|--------|-------------|
| Total squads | Count of all squads after merging, capping, and sub-squad splitting |
| Total unique files | Count of unique files across all squad assignments (deduplicated) |
| Total file-squad assignments | Sum of all files across all squads (includes duplicates from multi-assignment) |
| Estimated parallel batches | `ceil(total_squads / 4)` — squads are dispatched 4 at a time |
| Estimated time per batch | 3–5 minutes (varies by file count and complexity) |
| Estimated total time | `batches * 4 minutes` (midpoint estimate) |

**Display to user:**

> **Squad Assembly Complete**
>
> Deploying **[N] squads** across **[M] files** in **[B] batches**.
> - [List each squad name and its file count]
> - Multi-assigned files (covered by 2+ squads): [count]
> - Estimated time: **[T] minutes**
>
> Proceeding to Phase 2: ADVERSARIAL ASSAULT.

**Do NOT wait for user approval.** The user already approved the squad plan at Gate 1. Phase 1 is informational — display the assembly results and proceed directly to Phase 2.

## Phase 2: ADVERSARIAL ASSAULT

**Goal:** Find everything that is wrong. Dispatch squad agents in parallel batches. Each squad reads its own assigned files, applies every persona's attack techniques, and returns structured findings with proof-of-exploit. The orchestrator collects, validates return format, and assembles the master findings list for Phase 3.

Phase 2 is where the Swiss Cheese model takes effect — multiple independent squads with different perspectives attack the same codebase. Each squad agent starts fresh with zero memory of Phases 0-1. The context packet constructed in Phase 1 is their entire world.

### Step 2.1: Dispatch Squads

Dispatch all selected squads as parallel Agent tool calls. Each squad is dispatched using the Agent tool with the squad's context packet as its prompt. The Agent call must include:

- **`prompt`:** The fully rendered squad agent prompt (constructed per Step 2.5 below)
- **`description`:** `"FORTRESS Squad: {squad_name}"` — this appears in the UI so the user can track progress
- **`subagent_type`:** `"general-purpose"` — squad agents need full tool access (Read, Grep, Glob) to analyze their assigned files

Each squad agent operates independently. It reads its assigned files using the Read tool, applies its personas' attack techniques, and returns a structured JSON response per the return contract.

### Step 2.2: Batch Dispatch

Squads are dispatched in batches to manage parallel execution. Batch size is 3-5 squads per batch.

**Batch 1:**
- Dispatch the first 3-5 squads in parallel (simultaneous Agent tool calls in the same turn)
- Select the highest-priority squads for Batch 1: squads covering authentication, authorization, input validation, and payment/financial logic take precedence because their findings may inform other squads' analysis
- Wait for all Batch 1 agents to return before dispatching Batch 2

**Batch 2:**
- Dispatch remaining squads in parallel after Batch 1 completes
- If more than 5 squads remain, split into Batch 2 (up to 5) and Batch 3 (remainder)
- Continue until all squads have been dispatched and returned

**Batching rules:**
- Minimum batch size: 3 squads (unless fewer than 3 remain)
- Maximum batch size: 5 squads
- Always-active squads may be distributed across batches to balance load
- Sub-squads (from Step 1.4 splitting) are dispatched in the same batch as their sibling sub-squads when possible, to keep domain analysis temporally co-located

**Example dispatch sequence for 8 squads:**
```
Batch 1: [Squad 11: Auth, Squad 2: Edge Cases, Squad 12: Payments, Squad 6+7: Web Security] — 4 agents
Batch 2: [Squad 1: Infrastructure, Squad 5: Code Quality, Squad 22: Vibecoder, Squad W: Wildcard] — 4 agents
```

### Step 2.3: Collect and Validate Returns

As each squad agent returns, validate its response before adding findings to the master list. Apply the following checks in order:

**Check 1 — JSON structure validation:**

Parse the response for a valid JSON object matching the required return schema (defined in Step 2.5). The JSON must contain:
- `squad` (string) — squad name
- `findings` (array) — list of finding objects (may be empty)
- `positive_findings` (array) — list of positive observation strings
- `completion` (object) — with `status`, `findings_count`, `files_audited`, `files_skipped`, `skipped_reason`

If the response does not contain valid JSON with these fields, mark the squad as `PARSE_FAILURE` and proceed to re-dispatch (Check 3).

**Check 2 — Completion sentinel validation:**

Verify that `completion.status` equals the string `"SQUAD_COMPLETE"`. This sentinel confirms the squad agent finished its analysis rather than being truncated by context limits.

If the sentinel is present and the JSON is valid, accept the response and add all findings to the master findings list. Proceed to Check 4.

If the sentinel is missing (response may be truncated), proceed to Check 3.

**Check 3 — Re-dispatch with reduced scope:**

If a squad fails Check 1 or Check 2, re-dispatch it with a reduced scope:

1. Reduce the assigned file list to the **first 10 files only** (by the priority order from Step 1.4)
2. Add the following instruction to the top of the re-dispatched prompt: `"REDUCED SCOPE: Your prior attempt did not complete. You have been re-dispatched with a smaller file list. Focus on these files only. Return your findings even if analysis feels incomplete — partial coverage is better than no coverage."`
3. Dispatch as a single Agent call (not batched with other squads)

If the second attempt also fails (missing sentinel or invalid JSON):
- Log the failure: `"Squad {name} returned incomplete results after re-dispatch. Coverage gap in {domain}."`
- Record the squad name and domain in the coverage gap list for Phase 3 and Phase 6
- Do NOT attempt a third dispatch — two failures indicate a fundamental issue (context limits, file complexity, etc.)
- Continue with other squads' results

**Check 4 — Finding-level validation (lightweight):**

For each finding in an accepted squad response, verify the minimum required fields are present:
- `id` — must be a string matching the pattern `{SQUAD_NAME}-NNN`
- `severity` — must be one of: `CRITICAL`, `HIGH`, `MEDIUM`, `LOW`, `ENHANCEMENT`
- `file` — must be a non-empty string
- `proof` — must be a non-empty string
- `title` — must be a non-empty string

Findings missing any of these fields are tagged as `INCOMPLETE` and included in the master list with a validation warning. They are not discarded — Phase 3 will determine their disposition.

### Step 2.4: Log Progress

After all squads have completed (or failed after re-dispatch), display a progress summary to the user:

> **Phase 2: ADVERSARIAL ASSAULT — Complete**
>
> **Squad Results:**
> | Squad | Status | Findings | Files Audited |
> |-------|--------|----------|---------------|
> | {name} | COMPLETE / INCOMPLETE / FAILED | {count} | {count} |
> | ... | ... | ... | ... |
>
> **Summary:**
> - Total findings collected: **{N}**
> - Squads completed successfully: **{N}** of **{total}**
> - Squads with incomplete results: **{N}** (coverage gaps logged)
> - Positive observations: **{N}**
>
> Proceeding to Phase 3: VALIDATION.

If any squads failed, include:

> **Coverage Gaps:** The following domains have reduced coverage due to squad failures:
> - {domain}: {reason}
>
> These gaps will be noted in the final report.

**Do NOT wait for user approval.** Phase 2 results feed directly into Phase 3 (Validation). The user will review validated findings at Gate 2 after Phase 4.

### Step 2.5: Squad Agent Prompt Template

This section defines the exact prompt template that each squad agent receives. The prompt must be completely self-contained — squad agents start with zero memory of Phases 0-1. Every piece of information the squad needs must be in this prompt.

**Render the following template for each squad, substituting all `{variables}` with values from the context packet constructed in Step 1.3:**

```
You are a FORTRESS security audit squad. Your mission is to find every exploitable vulnerability, security weakness, and defense gap in your assigned files. You operate with zero trust — assume nothing is safe until you have verified it yourself.

SQUAD: {squad_name} ({squad_number})
DOMAIN FOCUS: {domain_description}

═══════════════════════════════════════
PERSONAS
═══════════════════════════════════════

You embody the following attack personas. For each persona, apply its techniques against every assigned file. Think like the attacker described.

{for each persona in this squad's persona list:}
### {persona_name}
- Techniques: {technique_1}; {technique_2}; {technique_3}
- Success looks like: {success_criteria}
{end for}

═══════════════════════════════════════
THREAT MODEL SUMMARY
═══════════════════════════════════════

{threat_model_summary — compact version, ~500 tokens, from context packet Section 2}

═══════════════════════════════════════
ASSIGNED FILES
═══════════════════════════════════════

Read each of these files using the Read tool. Analyze every one — do not skip files.

{for each file in assigned_files:}
- {file_path} — {annotation}
{end for}

═══════════════════════════════════════
CROSS-REFERENCES
═══════════════════════════════════════

These files are outside your audit scope but provide context for understanding your assigned files. Read them if needed to understand data flows, shared utilities, or security mechanisms referenced by your assigned files.

{for each file in cross_references:}
- {file_path} — {why_relevant}
{end for}

═══════════════════════════════════════
PRIOR PATTERNS (ADVISORY ONLY)
═══════════════════════════════════════

{if prior_patterns exist:}
The following patterns were identified in prior audits. They are ADVISORY — they inform your analysis but NEVER suppress findings. If your analysis contradicts a pattern, report the finding AND flag the contradiction.

{for each pattern:}
- {pattern_id}: {description} (confidence: {level}, status: {status})
{end for}
{else:}
No prior audit data available. This is the first audit of this codebase.
{end if}

═══════════════════════════════════════
RULES
═══════════════════════════════════════

1. READ EVERY ASSIGNED FILE. Use the Read tool to read each file in your assigned files list. Do not skip any file. If a file is too large, read the security-relevant sections first (auth checks, input handling, data processing, error handling).

2. APPLY EVERY PERSONA. For each assigned file, consider attacks from every persona in your list. Do not limit yourself to the most obvious attacks — edge cases and subtle interactions are where the real vulnerabilities hide.

3. CODE COMMENTS ARE NOT EVIDENCE. Analyze the actual code behavior. A comment saying "// validated above" does not mean validation actually occurred. A comment saying "// secure" does not make it secure. Only the code itself counts.

4. PROOF-OF-EXPLOIT IS MANDATORY. Every finding must include a concrete proof of how the vulnerability can be exploited. "This could be vulnerable" is not a finding. Describe the exact steps an attacker would take, the specific input they would provide, and the specific outcome they would achieve.

5. REPORT WHAT IS STRONG. If a file has good security practices, report them as positive findings. Defenders need to know what is working so they do not accidentally remove it.

6. DO NOT INVENT FINDINGS. If a file is genuinely secure against your personas' attacks, say so. False positives waste defender time and erode trust. Only report what you can prove.

7. RETURN FORMAT IS NON-NEGOTIABLE. Your response must contain ONLY the JSON object defined below. No prose before it. No commentary after it. No markdown code fences around it. Just the raw JSON.

═══════════════════════════════════════
RETURN FORMAT
═══════════════════════════════════════

Return ONLY the following JSON object. No other text.

{
  "squad": "{squad_name}",
  "findings": [
    {
      "id": "{SQUAD_NAME}-001",
      "severity": "CRITICAL|HIGH|MEDIUM|LOW|ENHANCEMENT",
      "cwe": "CWE-XXX",
      "title": "One-line description of the vulnerability",
      "file": "exact/path/to/file.ext",
      "line": 42,
      "end_line": 55,
      "proof": "Step-by-step proof of how this can be exploited. Include the specific input, the vulnerable code path, and the resulting impact. Be concrete.",
      "fix": "Description of the fix needed in plain language",
      "fix_code": "The actual replacement code that would fix the vulnerability. Must be syntactically valid and drop-in ready.",
      "confidence": "HIGH|MEDIUM|LOW",
      "attack_narrative": "As a [persona name], I would exploit this by [specific attack steps]. This succeeds because [root cause]."
    }
  ],
  "positive_findings": [
    "Description of something this codebase does well defensively"
  ],
  "completion": {
    "status": "SQUAD_COMPLETE",
    "findings_count": 0,
    "files_audited": 0,
    "files_skipped": 0,
    "skipped_reason": null
  }
}

FIELD REQUIREMENTS:
- "id": Sequential within this squad. Format: {SQUAD_NAME}-001, {SQUAD_NAME}-002, etc. Use uppercase squad name with hyphens (e.g., "INFRASTRUCTURE-001", "WEB-SECURITY-003").
- "severity": One of exactly: CRITICAL, HIGH, MEDIUM, LOW, ENHANCEMENT. Use CRITICAL only for remotely exploitable vulnerabilities with high impact. Use ENHANCEMENT for hardening recommendations that are not exploitable vulnerabilities.
- "cwe": The most specific applicable CWE identifier. Use "CWE-000" if no CWE applies.
- "title": One line. Be specific: "SQL injection in /api/users via unsanitized 'sort' parameter" not "SQL injection vulnerability."
- "file": Exact relative path from project root. Must match the file path as it appears in your assigned files list.
- "line": The first line number where the vulnerable code begins.
- "end_line": The last line number of the vulnerable code block. If single line, same as "line".
- "proof": Minimum 2 sentences. Must describe a concrete attack vector, not a theoretical possibility.
- "fix": Plain language description. One paragraph.
- "fix_code": The actual code that should replace the vulnerable code. Must be syntactically valid in the project's language. If the fix requires changes in multiple locations, describe all locations in the "fix" field and provide the primary fix code here.
- "confidence": HIGH = you read the code and verified the vulnerability exists. MEDIUM = the code pattern is vulnerable but runtime behavior may mitigate. LOW = the vulnerability depends on assumptions about the runtime environment or configuration.
- "attack_narrative": Written in first person as the attacking persona. Must reference a specific persona from your persona list.
- "positive_findings": Array of strings. Each string describes one defensive strength observed. At least 1 positive finding is expected per squad.
- "completion.status": MUST be the literal string "SQUAD_COMPLETE". This sentinel confirms your analysis finished without truncation.
- "completion.findings_count": Integer count of items in the findings array.
- "completion.files_audited": Integer count of files you successfully read and analyzed.
- "completion.files_skipped": Integer count of assigned files you could not read or did not analyze. 0 is the target.
- "completion.skipped_reason": null if files_skipped is 0. Otherwise, a string explaining why files were skipped (e.g., "File too large, read first 500 lines only" or "Binary file, not analyzable").

YOUR RESPONSE MUST BE ONLY THIS JSON. NO PROSE BEFORE OR AFTER.
```

### Step 2.6: Prompt Construction Rules

When rendering the squad agent prompt template (Step 2.5) for each squad, follow these construction rules:

**Rule 1 — Persona extraction:**

Extract personas from the Persona Taxonomy (Section 11 of this skill file). Select ONLY the personas belonging to this squad's domain. For each persona, format as:
- **Name:** The persona's name as listed in the taxonomy
- **Techniques:** 2-3 technique keywords or short phrases describing the persona's attack approach
- **Success criteria:** One sentence describing what a successful attack by this persona looks like

Do NOT include personas from other squads. For merged squads (from Step 1.2), include personas from ALL merged source squads.

**Rule 2 — Persona formatting:**

Full format (default):
```
### Null Specialist
- Techniques: null injection in all input fields; null byte truncation in file paths; null coalescing bypass in conditional logic
- Success looks like: Application crashes, returns unexpected data, or bypasses validation when receiving null/undefined/None where a value is expected
```

Compressed format (when prompt exceeds token budget):
```
- Null Specialist: null injection, null byte truncation, null coalescing bypass → crash/bypass on null input
```

**Rule 3 — File list construction:**

Build the assigned files list from the context packet (Step 1.3, Section 3). For each file:
1. Include the exact relative file path from the project root
2. Include the one-line annotation describing what the file does and why it is security-relevant
3. Order files by security relevance: authentication and authorization files first, then input processing, then data layer, then configuration, then general code

Build the cross-references list from the context packet (Step 1.3, Section 4). Include only files that the squad will need to understand but not audit.

**Rule 4 — Threat model inclusion:**

Include the compact threat model summary (~500 tokens) from the context packet (Step 1.3, Section 2). This summary must contain:
- Project name, languages, frameworks
- Architecture one-liner
- Key trust boundaries
- Key data flows
- STRIDE highlights most relevant to this squad's domain (2-3 bullets, not all 6 categories)
- Scope classification

**Rule 5 — Token budget management:**

Estimate the total prompt token count. The squad agent prompt should target approximately 3,000-4,000 tokens to leave maximum context for file reading and analysis. If the rendered prompt exceeds ~4,000 tokens:

1. **First:** Compress personas to one-line format (Rule 2 compressed format)
2. **Second:** Reduce cross-references to the 5 most relevant files
3. **Third:** Trim the threat model summary to ~300 tokens, keeping only the most squad-relevant information
4. **Fourth:** If still over budget, reduce annotations to file path only (remove the one-line description)

Do NOT remove assigned files from the list to reduce token count — file coverage is more important than prompt verbosity.

**Rule 6 — Agent tool parameters:**

When dispatching each squad via the Agent tool, use the following parameters:
- **`prompt`:** The fully rendered squad agent prompt from the template above
- **`description`:** `"FORTRESS Squad: {squad_name}"` — where `{squad_name}` is the human-readable squad name (e.g., "Infrastructure & Supply Chain", "Web Security", "AI/LLM Security")
- **`subagent_type`:** `"general-purpose"` — squad agents need access to Read, Grep, and Glob tools to analyze their assigned files

## Phase 3: VALIDATION

**Goal:** Eliminate false positives, merge duplicates, and build the fix dependency graph. This phase transforms the raw master findings list from Phase 2 into a validated, deduplicated, dependency-aware set of findings ready for standards enrichment and reporting in Phase 4.

Phase 3 is what separates FORTRESS from naive "ask the AI to audit my code" approaches. Grounding checks prevent hallucinated file:line references. Counter-proofs reduce false positives. Deduplication merges redundant findings while boosting confidence on corroborated ones. The fix dependency graph prevents conflicts during execution. Every removal and downgrade is logged with reasoning so the human reviewer can override.

**Do NOT wait for user approval.** Phase 3 processes findings automatically and feeds directly into Phase 4. The user reviews the validated, enriched findings at Gate 2 after Phase 4.

### Step 3.1: Grounding Checks

For EVERY finding in the master findings list from Phase 2, verify that it references real code at real locations. Findings that reference non-existent files, wrong line numbers, or phantom identifiers are hallucinations and must be removed.

Apply the following three checks in order. If a finding fails ANY check, mark it as **UNGROUNDED** and remove it from the findings list.

**Check 1 — File exists:**

Use the Glob tool to confirm that the file path in `finding.file` exists in the project. Match the exact relative path from the project root.

- If the file exists: pass. Proceed to Check 2.
- If the file does NOT exist: mark the finding as **UNGROUNDED** with reason `"File not found: {finding.file}"`. Remove the finding. Do NOT attempt fuzzy matching or path correction — if the squad hallucinated the file path, the entire finding is suspect.

**Check 2 — Line is valid:**

Use the Read tool to read `finding.file` and verify that `finding.line` is within the file's line count and contains code related to the finding.

1. Read the file (or the relevant section around `finding.line`).
2. Verify `finding.line` is within the file's total line count. If `finding.line` exceeds the file's line count: mark **UNGROUNDED** with reason `"Line {finding.line} exceeds file length ({actual_length} lines)"`.
3. Read the code at `finding.line` (and surrounding lines up to `finding.end_line` if present). Verify the code at that location is related to the finding's `title` and `proof`. Specifically:
   - If the line is a blank line: **UNGROUNDED** with reason `"Line {finding.line} is blank"`.
   - If the line is only a comment (no executable code on or immediately adjacent to the cited lines): **UNGROUNDED** with reason `"Line {finding.line} is a comment, not executable code"`.
   - If the code at the line is completely unrelated to what the finding describes (e.g., finding says "SQL injection in query builder" but the line is a CSS import): **UNGROUNDED** with reason `"Code at line {finding.line} is unrelated to finding: {finding.title}"`.
   - If the code is related but the line number is off by a small amount (the described vulnerability exists in the same function but at a different line): **do NOT mark ungrounded**. Instead, correct the `finding.line` and `finding.end_line` to the actual location and add a note: `"Line reference corrected from {original} to {corrected}"`.

**Check 3 — Fix references real code:**

If `finding.fix_code` is present and non-empty, verify that any function names, variable names, imports, or module references in the fix code actually exist in the codebase.

1. Extract identifiers from `finding.fix_code` that appear to be references to existing code: function calls, imported modules, referenced variables or constants that are not being newly defined by the fix itself.
2. For each referenced identifier, use Grep to search the codebase for its definition or declaration.
3. If the fix code references a function, module, or variable that does not exist anywhere in the codebase (and is not a standard library / framework built-in): mark **UNGROUNDED** with reason `"Fix references non-existent identifier: {identifier}"`.
4. If the fix code only references standard library functions, framework built-ins, or identifiers that exist in the codebase: pass.

**Exception:** Findings tagged as `INCOMPLETE` during Phase 2 (Step 2.3, Check 4) that are missing required fields should be removed during grounding checks. They cannot be properly validated without complete data.

**Grounding check performance optimization:** Process grounding checks in file-grouped batches. Group all findings by `finding.file`, read each file once, and validate all findings for that file together. This avoids redundant file reads.

After all grounding checks are complete, log:

> **Grounding check: {N} findings passed, {M} removed as ungrounded.**

If any findings were removed, list them:

> **Ungrounded findings removed:**
> - {finding.id}: {reason}
> - ...

### Step 3.2: Counter-Proof Generation

For each finding that passed grounding checks, attempt to disprove it. The goal is to find reasons the vulnerability is NOT actually exploitable. If the counter-proof is stronger than the original proof, downgrade or remove the finding.

Apply the following counter-proof checks in order. Each check that produces a valid counter-proof weakens the finding. Multiple counter-proofs compound.

**Counter-proof 1 — Framework mitigation:**

Check whether the detected framework (from Phase 0) provides built-in protection against the vulnerability described in the finding.

1. Reference the framework name and version detected during Phase 0 recon.
2. For the CWE category of the finding, determine whether the framework version's security defaults mitigate it. Examples:
   - React auto-escapes JSX output — XSS findings in JSX expressions need proof that an explicit raw HTML bypass is used
   - Next.js 14+ Server Actions include CSRF tokens by default — CSRF findings need proof the protection is disabled or bypassed
   - Django ORM parameterizes queries by default — SQL injection findings need proof that `raw()`, `extra()`, or string formatting is used
   - Rust's borrow checker prevents use-after-free — memory safety findings need proof of `unsafe` blocks
   - Express.js does NOT auto-sanitize input — input validation findings stand without counter-proof

3. If the framework provides built-in mitigation AND the finding's proof does not demonstrate a bypass of that mitigation: generate counter-proof `"Framework mitigation: {framework} v{version} provides {protection} by default. Finding proof does not demonstrate bypass."` Downgrade severity by one level (CRITICAL to HIGH, HIGH to MEDIUM, MEDIUM to LOW, LOW to ENHANCEMENT).

4. If the framework does NOT provide relevant mitigation, or the finding's proof explicitly shows the mitigation is bypassed: no counter-proof generated. Finding stands.

**Counter-proof 2 — Upstream sanitization:**

Check whether there is input validation or sanitization upstream of the vulnerable code that the squad may have missed.

1. Identify the data flow into the vulnerable function. Read the callers of the function containing `finding.file:finding.line` — use Grep to find call sites.
2. For each caller, check whether it validates, sanitizes, or constrains the input before passing it to the vulnerable function.
3. If ALL paths into the vulnerable function pass through adequate sanitization: generate counter-proof `"Upstream sanitization: all call paths to {function} pass through {sanitizer} at {file}:{line}."` Downgrade severity by one level.
4. If ANY path into the vulnerable function lacks sanitization: no counter-proof. The finding stands because an attacker only needs one unprotected path.

**Counter-proof 3 — Unreachable attack vector:**

Check whether the attack vector described in the finding is actually reachable from user input.

1. Read the `finding.proof` and `finding.attack_narrative` to identify the assumed entry point (user input source).
2. Trace the data flow from the entry point to the vulnerable code. Is the entry point actually exposed? Considerations:
   - Is the route/endpoint registered and accessible? (Check route configuration)
   - Is the function only called from internal/admin contexts behind authentication?
   - Is the input constrained by type system, schema validation, or API gateway rules before reaching the vulnerable code?
3. If the attack vector requires user input to reach code that is not actually reachable from any user-facing surface: generate counter-proof `"Unreachable vector: {entry_point} is not exposed to user input because {reason}."` Remove the finding entirely (not just downgrade).
4. If the entry point IS reachable but requires authentication: do NOT generate a counter-proof (authenticated users can still be attackers, or credentials can be compromised). The finding stands.

**Counter-proof 4 — Known pattern advisory:**

Check whether a `.fortress/known-patterns.md` entry explains why this code pattern is safe.

1. If `.fortress/known-patterns.md` exists, search it for entries matching the finding's CWE, file, or vulnerability description.
2. If a matching pattern exists with status `"verified-safe"` or `"accepted-risk"`:
   - Do NOT auto-remove the finding.
   - Do NOT auto-downgrade the finding.
   - Flag it for human review: add `"known_pattern_match": "{pattern_id}"` to the finding metadata.
   - Add an advisory note: `"Known pattern {pattern_id} suggests this may be safe/accepted. Flagged for human review — prior audit data is advisory only, never suppressive."`
3. If a matching pattern exists but the finding CONTRADICTS the pattern (e.g., pattern says "safe" but finding has proof of exploit): flag the contradiction explicitly: `"CONTRADICTION: Finding {finding.id} contradicts known pattern {pattern_id}. Finding has proof-of-exploit; pattern claims safe. Human review required."`

**Counter-proof resolution:**

After all four counter-proof checks:

- If the finding accumulated ONE counter-proof (from checks 1-3): downgrade severity by one level. Record the counter-proof reasoning in a `counter_proofs` array on the finding.
- If the finding accumulated TWO OR MORE counter-proofs (from checks 1-3): downgrade severity by two levels OR remove entirely if the original severity was MEDIUM or lower. Record all counter-proof reasoning.
- If counter-proof 3 (unreachable vector) applies: remove the finding regardless of other counter-proofs.
- Counter-proof 4 (known pattern) never causes automatic downgrade or removal — it only flags for human review.

Log:

> **Counter-proof generation: {N} findings unchanged, {M} downgraded, {K} removed.**

For each downgraded or removed finding, log the counter-proof reasoning:

> **Counter-proof results:**
> - {finding.id}: {action} — {counter_proof_summary}
> - ...

### Step 3.3: Deduplication

Multiple squads may discover the same vulnerability from different angles. This step merges true duplicates while preserving genuinely distinct findings that happen to be nearby.

**Step 3.3a — Group by file and line range:**

1. Sort all remaining findings by `finding.file`, then by `finding.line`.
2. For each file, identify **overlapping findings**: two findings overlap if their line ranges `[line, end_line]` intersect. Specifically, finding A and finding B overlap if:
   - `A.file == B.file` AND
   - `A.line <= B.end_line` AND `B.line <= A.end_line`
3. Build overlap groups: if A overlaps B and B overlaps C, then {A, B, C} form one overlap group (transitive closure).

**Step 3.3b — Evaluate overlap groups:**

For each overlap group containing 2+ findings, determine whether to merge or keep separate:

**Merge criteria (ALL must be true):**
- The findings describe the same vulnerability type (same or closely related CWE)
- The findings target the same code construct (same function, same statement, same data flow)
- The findings differ primarily in which squad found them or how they worded the description

**Keep separate criteria (ANY is sufficient):**
- The findings describe genuinely different vulnerability types (different CWE categories, e.g., one is XSS and the other is CSRF)
- The findings target different attack vectors even though the code location overlaps (e.g., one attacks the input parsing, the other attacks the output rendering of the same function)
- The findings have different fix requirements (fixing one would not fix the other)

**Step 3.3c — Merge execution:**

When merging findings within an overlap group:

1. **ID:** Use the ID from the finding with the highest original severity. If tied, use the finding from the first squad alphabetically.
2. **Severity:** Use the highest severity among the merged findings.
3. **Confidence:** Set to **HIGH** — multi-squad corroboration is the strongest confidence signal in FORTRESS.
4. **Title:** Use the most specific and descriptive title among the merged findings.
5. **Proof:** Use the most detailed and concrete proof among the merged findings. If multiple proofs contain unique information, combine them.
6. **Fix / Fix code:** Use the most complete and correct fix. If fixes conflict, keep both and mark as alternatives (to be resolved in the fix dependency graph).
7. **Attribution:** Add a `squads` array listing all squads that independently found this issue: `"squads": ["INFRASTRUCTURE", "WEB-SECURITY"]`.
8. **Attack narrative:** Preserve all unique attack narratives from different squads — they demonstrate different angles of exploitation.

**Step 3.3d — Single-squad findings:**

Findings NOT part of any overlap group (found by only one squad) retain their original confidence level. Do not penalize single-squad findings — they may be correct findings in a domain only one squad was equipped to test.

Log:

> **Deduplication: {N} findings merged into {M} unique findings. {K} findings corroborated by multiple squads.**

### Step 3.4: Build Fix Dependency Graph

For each remaining finding that has a proposed fix (`finding.fix_code` is non-empty), build a dependency graph to detect conflicts, clusters, and ordering requirements. This graph is used in Phase 5 to apply fixes safely.

**Step 3.4a — Record fix metadata:**

For each finding with a fix, record:

```
{
  "finding_id": "{finding.id}",
  "file": "{finding.file}",
  "start_line": {finding.line},
  "end_line": {finding.end_line},
  "action": "replace" | "insert" | "delete",
  "fix_code": "{finding.fix_code}"
}
```

Determine `action` from the fix:
- **replace:** The fix provides replacement code for the existing lines (most common). The code at `[start_line, end_line]` is replaced with `fix_code`.
- **insert:** The fix adds new code without removing existing lines. `start_line` and `end_line` are the same and indicate the insertion point.
- **delete:** The fix removes code without replacement. `fix_code` is empty or describes what to remove.

**Step 3.4b — Detect intra-file conflicts:**

Within each file, check for overlapping line ranges between fixes:

1. Sort fixes by `start_line` within each file.
2. For each pair of fixes (A, B) in the same file, check if their line ranges overlap: `A.start_line <= B.end_line AND B.start_line <= A.end_line`.
3. If ranges overlap:
   - **Compatible actions:** Both are `replace` and the combined replacement is coherent (one fix is a subset of the other, or they modify different parts of the same line range). Mark as a **fix cluster** — these fixes must be approved and applied together as a unit.
   - **Incompatible actions:** The fixes modify the same lines in conflicting ways (e.g., one replaces a function and the other deletes it, or two replacements produce different code for the same lines). Mark as **alternatives** — present both options to the user in Phase 4, and the user picks which to apply.

**Step 3.4c — Detect cross-file dependencies:**

Check whether fixes in different files may interact:

1. For each pair of files that have fixes, check the import graph: does file X import file Y (or vice versa)? Use Grep to search for import/require/include statements.
2. Extend to 2 hops: if file X imports file Z, and file Z imports file Y, then X and Y are within 2 hops.
3. If Fix A modifies file X and Fix B modifies file Y, and X and Y are within 2 import hops: flag the pair for **integration review** in Phase 5b. Record: `"integration_review": ["{finding_A.id}", "{finding_B.id}"]`.
4. Cross-file dependencies do NOT block fixes from being applied independently — they only flag the need for integration verification after application.

**Step 3.4d — Establish fix ordering:**

Within each file, determine the safe application order:

1. **Bottom-up rule:** Fixes within the same file must be applied from highest line number to lowest. This prevents line-number shift from invalidating subsequent fix locations.
2. **Fix clusters:** All fixes in a cluster are applied together as a single atomic operation. The cluster's position in the ordering is determined by the highest `start_line` in the cluster.
3. **Cross-file:** Fixes in different files have no inherent ordering constraint (they can be applied in any order or in parallel), unless they are flagged for integration review.

Record the complete fix ordering as an ordered list:

```
fix_order: [
  { file: "src/auth.ts", fixes: ["AUTH-003", "AUTH-001"], order: "bottom-up" },
  { file: "src/api/users.ts", fixes: ["WEB-002"], order: "single fix" },
  { file: "src/db/queries.ts", fixes: ["EDGE-005", "INFRA-001"], order: "bottom-up, cluster" }
]
```

Log:

> **Fix dependency graph: {N} fixes mapped. {C} fix clusters, {A} alternative pairs, {I} cross-file integration reviews flagged.**

### Step 3.5: Coverage Map

Track what was and was not analyzed to surface coverage gaps in the final report.

**Step 3.5a — File coverage:**

1. From Phase 1, collect the complete list of all source files assigned to at least one squad (the union of all squads' assigned file lists).
2. From Phase 0, collect the complete file inventory (excluding dependency/build/VCS directories).
3. Compute:
   - **Analyzed files:** Files assigned to at least one squad.
   - **Unanalyzed files:** Files in the inventory that were not assigned to any squad.
   - **Coverage percentage:** `(analyzed / total_source_files) * 100`, where `total_source_files` excludes non-source files (images, fonts, generated files, lock files, etc.).
4. For unanalyzed files, note why they were excluded (e.g., "test fixtures", "generated code", "static assets", "documentation") or flag as a genuine coverage gap if they contain executable code that was not reviewed.

**Step 3.5b — CWE Top 25 coverage:**

For each of the CWE Top 25 Most Dangerous Software Weaknesses (current list), determine whether at least one persona across all squads was specifically testing for it:

| CWE | Name | Covered? | Squad(s) |
|-----|------|----------|----------|
| CWE-79 | Cross-site Scripting | Yes/No | {squad names} |
| CWE-89 | SQL Injection | Yes/No | {squad names} |
| ... | ... | ... | ... |

Mark each as covered (at least one persona's techniques target this CWE) or uncovered (no persona specifically tests for this class of vulnerability).

**Step 3.5c — OWASP Top 10 coverage:**

For each of the OWASP Top 10 2025 Web Application Security Risks, determine coverage:

| Category | Covered? | Squad(s) |
|----------|----------|----------|
| A01: Broken Access Control | Yes/No | {squad names} |
| A02: Security Misconfiguration | Yes/No | {squad names} |
| ... | ... | ... |

Additionally, if the project involves AI/LLM components (detected in Phase 0), check OWASP Top 10 for LLM Applications 2025 coverage. If the project involves agentic AI systems, check OWASP Top 10 for Agentic AI Systems 2026 coverage.

**Step 3.5d — Domain gap analysis:**

1. List all domains that had at least one squad assigned (from Phase 1 squad selection).
2. For each domain with an assigned squad, note whether the squad produced zero findings. Zero findings from a domain-specific squad may indicate:
   - The codebase is genuinely strong in that domain (good) — cross-reference with positive findings
   - The squad's scope was too narrow or its personas were not well-suited (investigate)
   - The assigned files did not actually contain code relevant to that domain (file assignment error)
3. Flag zero-finding domains as `"investigate further"` in the report — the user should consider whether the absence of findings reflects genuine security or insufficient coverage.

**Step 3.5e — Squad failure coverage impact:**

If any squads failed during Phase 2 (recorded in the coverage gap list from Step 2.3/2.4), note the specific impact:
- Which files were those squads assigned?
- Which domains/CWE categories are now under-covered?
- Recommend whether the user should re-run a focused audit on the affected domain.

Log:

> **Coverage map: {X}% of source files analyzed ({A} of {T} files). {Y}/25 CWE Top 25 categories covered. {Z}/10 OWASP Top 10 categories covered. {G} domains with zero findings flagged for investigation.**

### Phase 3 Summary

After all five steps complete, display a summary before proceeding to Phase 4:

> **Phase 3: VALIDATION — Complete**
>
> | Metric | Count |
> |--------|-------|
> | Findings entering validation | {input_count} |
> | Removed as ungrounded (Step 3.1) | {ungrounded_count} |
> | Removed by counter-proof (Step 3.2) | {counterproof_removed} |
> | Downgraded by counter-proof (Step 3.2) | {counterproof_downgraded} |
> | Merged as duplicates (Step 3.3) | {merged_count} |
> | **Validated findings** | **{final_count}** |
> | Fix clusters | {cluster_count} |
> | Alternative fix pairs | {alternative_count} |
> | Cross-file integration reviews | {integration_count} |
> | Source file coverage | {coverage_pct}% |
>
> Proceeding to Phase 4: REPORT & PROPOSE.

**Do NOT wait for user approval.** Proceed directly to Phase 4.

## Phase 4: REPORT AND PROPOSE

**Goal:** Transform validated findings into a defense-grade, standards-mapped security report with full enrichment, confidence scoring, severity classification, and a complete artifact suite. Phase 4 converts raw validated findings from Phase 3 into actionable intelligence that meets enterprise security reporting standards.

Phase 4 operates in two stages: Standards Enrichment (Steps 4.1–4.3, this section) and Artifact Generation (Steps 4.4+, next section). Standards Enrichment runs first to classify every finding before any artifact is produced.

### Step 4.1: Mandatory Limitations Header

EVERY report produced by FORTRESS — whether displayed in conversation or written to a file — MUST begin with the following header text. This header is NON-REMOVABLE. It must appear in full mode, quick mode, focused mode, verify mode, and diff mode. No invocation mode, user preference, or configuration setting may suppress, abbreviate, or omit this header.

> **Scope and Limitations:** This report was generated by FORTRESS, an AI-powered static code analysis framework. It is not a penetration test, compliance audit, or security certification. Standards mappings (CWE, CVSS, NIST) are estimates, not authoritative assessments. This audit did not cover: runtime/dynamic testing, infrastructure configuration outside the repository, social engineering, physical security, binary analysis, or network scanning. See the Compliance Posture Summary for specific framework coverage details.

This header must appear:
- At the top of the in-conversation executive summary
- At the top of the detailed markdown report artifact
- At the top of the compliance posture summary artifact
- In the metadata section of the SARIF artifact
- At the top of the POA&M template artifact
- At the top of the public security page template artifact
- At the top of the delta report artifact

If any artifact is generated without this header, the artifact is non-conformant and must be regenerated.

### Step 4.2: Standards Enrichment

For each validated finding that survived Phase 3, enrich it with the full set of standards mappings defined below. Reference the Standards Reference tables in Section 12 of this file for lookup data, decision trees, and category definitions.

Process each finding through all applicable enrichment steps in order:

#### Step 4.2a: CWE Assignment

Assign a Common Weakness Enumeration (CWE) identifier to each finding.

1. **Check existing assignment:** If the originating squad already assigned a CWE during Phase 2, validate it against the CWE Lookup Table in Section 12.
2. **Validate or correct:** If the squad's CWE assignment matches the finding's actual weakness class, keep it. If the assignment is wrong (the weakness described does not match the CWE definition), replace it with the correct CWE from the lookup table.
3. **Assign if missing:** If no CWE was assigned, determine the best match from the CWE Lookup Table based on the finding's weakness class, attack vector, and affected component.
4. **Handle ambiguity:** If the finding could reasonably map to two or more CWEs and you cannot determine which is more precise, assign the parent category CWE that encompasses both. Do not guess between siblings — go up one level.
5. **Format:** Always format as `CWE-XXX (Weakness Name)` — for example, `CWE-79 (Improper Neutralization of Input During Web Page Generation)`.

Log each CWE assignment with the rationale (kept squad assignment / corrected from X to Y / assigned new / assigned parent due to ambiguity).

#### Step 4.2b: CVSS 4.0 Estimation

Estimate a CVSS 4.0 Base Score for each finding using the CVSS Decision Tree in Section 12.

Determine each of the 11 base metrics by analyzing the finding's characteristics:

| Metric | Code | Values | Determination Basis |
|--------|------|--------|-------------------|
| Attack Vector | AV | Network (N) / Adjacent (A) / Local (L) / Physical (P) | How the vulnerability is exploited — over network, adjacent network, local access, or physical access |
| Attack Complexity | AC | Low (L) / High (H) | Whether exploitation requires special conditions beyond attacker control |
| Attack Requirements | AT | None (N) / Present (P) | Whether specific deployment or execution conditions must exist |
| Privileges Required | PR | None (N) / Low (L) / High (H) | Level of authentication needed before exploitation |
| User Interaction | UI | None (N) / Passive (P) / Active (A) | Whether a victim must take action for exploitation to succeed |
| Vulnerable System Confidentiality | VC | None (N) / Low (L) / High (H) | Confidentiality impact on the vulnerable component |
| Vulnerable System Integrity | VI | None (N) / Low (L) / High (H) | Integrity impact on the vulnerable component |
| Vulnerable System Availability | VA | None (N) / Low (L) / High (H) | Availability impact on the vulnerable component |
| Subsequent System Confidentiality | SC | None (N) / Low (L) / High (H) | Confidentiality impact on systems beyond the vulnerable component |
| Subsequent System Integrity | SI | None (N) / Low (L) / High (H) | Integrity impact on systems beyond the vulnerable component |
| Subsequent System Availability | SA | None (N) / Low (L) / High (H) | Availability impact on systems beyond the vulnerable component |

**Construct the vector string** in the format: `CVSS:4.0/AV:X/AC:X/AT:X/PR:X/UI:X/VC:X/VI:X/VA:X/SC:X/SI:X/SA:X`

**ALWAYS label the score as "estimated."** Format: `CVSS 4.0: 7.3 (estimated) — CVSS:4.0/AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:N/VA:N/SC:N/SI:N/SA:N`

**Mandatory cross-check — apply these sanity rules after scoring:**

| Scenario | Expected Range | Action if Mismatch |
|----------|---------------|-------------------|
| Remote + No Auth + No Interaction + Full CIA Impact | 9.0–10.0 | Re-evaluate — score should be Critical |
| Remote + No Auth + Partial Impact | 7.0–8.9 | Re-evaluate if outside this range |
| Local + Auth Required + Partial Impact | 4.0–6.0 | Re-evaluate if outside this range |
| Local + High Auth + No Subsequent Impact | 2.0–4.0 | Re-evaluate if outside this range |
| Physical access only | 0.1–3.9 | Re-evaluate if above 4.0 |

If a score falls outside its expected range after cross-check, re-examine each metric value. Adjust if a metric was misjudged, or document the rationale if the score is correct despite seeming anomalous (e.g., a local vulnerability with unusually high impact due to privilege escalation).

#### Step 4.2c: OWASP Mapping

Map each finding to the applicable OWASP Top 10 lists. A single finding may map to categories on multiple lists if applicable.

**OWASP Top 10:2025 (Web Application Security Risks):**
Apply to all web-related findings. Map to the most specific category from A01 through A10. Reference the OWASP Web mapping table in Section 12.

**OWASP Top 10 for LLM Applications:2025:**
Apply only to findings related to AI/LLM components (typically from Squads 14–15 or findings involving prompt handling, model interaction, or AI-generated output). Map to LLM01 through LLM10. Reference the OWASP LLM mapping table in Section 12.

**OWASP Top 10 for Agentic AI Systems:2026:**
Apply only to findings related to agentic AI systems — tool-calling agents, MCP servers, autonomous decision-making components, multi-agent orchestration. Map to ASI01 through ASI10. Reference the OWASP Agentic mapping table in Section 12.

If a finding is not related to web applications, do not force an OWASP Web mapping — mark as "N/A (non-web finding)." Similarly, do not force LLM or Agentic mappings on non-AI findings.

Format: `OWASP Web A03:2025 (Injection)` or `OWASP LLM LLM01:2025 (Prompt Injection)` or `N/A (non-web finding)`.

#### Step 4.2d: NIST 800-53 Control Mapping

Assign the most relevant NIST 800-53 security control to each finding from the following set:

| Control | Name | Typical Finding Types |
|---------|------|----------------------|
| SA-11 | Developer Testing and Evaluation | Missing security tests, untested security controls |
| RA-5 | Vulnerability Monitoring and Scanning | Known CVEs in dependencies, unpatched components |
| AC-3 | Access Enforcement | Broken access control, missing authorization checks |
| AC-6 | Least Privilege | Excessive permissions, overly broad access |
| AU-2 | Event Logging | Missing audit trails, insufficient logging |
| IA-5 | Authenticator Management | Weak credential handling, hardcoded secrets |
| SC-8 | Transmission Confidentiality and Integrity | Unencrypted communication, missing TLS |
| SC-13 | Cryptographic Protection | Weak algorithms, improper crypto usage |
| SI-2 | Flaw Remediation | Known vulnerabilities not remediated |
| SI-7 | Software, Firmware, and Information Integrity | Code integrity failures, unsigned artifacts |
| CM-3 | Configuration Change Control | Insecure defaults, configuration drift |

Select the single most relevant control. If a finding genuinely spans two controls (e.g., a hardcoded secret that also grants excessive access), assign the primary control and note the secondary.

Format: `NIST 800-53: AC-3 (Access Enforcement)` or `NIST 800-53: IA-5 (Authenticator Management), secondary: AC-6 (Least Privilege)`.

#### Step 4.2e: NIST SSDF Practice Mapping

Map each finding to the most relevant NIST Secure Software Development Framework (SSDF) practice group:

| Practice Group | Code | Scope |
|---------------|------|-------|
| Prepare the Organization | PO | Governance, training, tooling, security requirements |
| Protect the Software | PS | Source code protection, integrity verification, artifact security |
| Produce Well-Secured Software | PW | Design, implementation, testing, vulnerability management |
| Respond to Vulnerabilities | RV | Vulnerability response, disclosure, remediation |

Most findings from adversarial audit will map to **PW** (Produce Well-Secured Software) since they represent implementation-level weaknesses. Supply chain findings typically map to **PS**. Governance gaps map to **PO**. Findings about missing vulnerability response processes map to **RV**.

Format: `NIST SSDF: PW (Produce Well-Secured Software)`.

#### Step 4.2f: STIG CAT Classification

Assign a DISA STIG severity category to each finding based on its CVSS score and potential mission impact:

| Category | Criteria | Description |
|----------|----------|-------------|
| **CAT I** | CVSS >= 7.0 OR could directly cause loss of confidentiality, integrity, or availability of a system or data | Critical vulnerabilities requiring immediate remediation. Exploitation could directly compromise the system. |
| **CAT II** | CVSS 4.0–6.9 OR could degrade mission capability without direct system compromise | Moderate vulnerabilities that should be remediated promptly. Exploitation could reduce security posture. |
| **CAT III** | CVSS < 4.0 OR could degrade defense-in-depth measures without direct mission impact | Low-severity vulnerabilities representing security hardening opportunities. |

**Tiebreaker rules:**
- If the CVSS score suggests one category but the mission impact analysis suggests another, use the MORE severe category (err on the side of caution).
- CRITICAL and HIGH severity findings from Phase 3 should generally be CAT I or CAT II.
- ENHANCEMENT and POSITIVE findings do not receive STIG classification — mark as "N/A."

Format: `STIG: CAT I` or `STIG: CAT II` or `STIG: N/A (enhancement)`.

#### Step 4.2g: MITRE ATT&CK Technique Mapping

Assign the most relevant MITRE ATT&CK technique to each finding. Select from the following common techniques (non-exhaustive — use the best match from the full ATT&CK matrix if none of these fit):

| Technique ID | Name | Typical Finding Types |
|-------------|------|----------------------|
| T1190 | Exploit Public-Facing Application | Web app vulnerabilities, API flaws, injection |
| T1059 | Command and Scripting Interpreter | Command injection, code execution, eval usage |
| T1078 | Valid Accounts | Authentication bypass, credential theft, session hijacking |
| T1552 | Unsecured Credentials | Hardcoded secrets, exposed API keys, credential files |
| T1068 | Exploitation for Privilege Escalation | Privilege escalation, IDOR, broken access control |
| T1195 | Supply Chain Compromise | Dependency vulnerabilities, package confusion, build tampering |
| T1071 | Application Layer Protocol | Data exfiltration, C2 communication, protocol abuse |
| T1485 | Data Destruction | Destructive operations, mass deletion without safeguards |
| T1565 | Data Manipulation | Data integrity attacks, unauthorized modification |
| T1499 | Endpoint Denial of Service | ReDoS, resource exhaustion, amplification attacks |

If the finding does not clearly map to an ATT&CK technique, mark as "N/A — no direct ATT&CK mapping" rather than forcing a poor fit.

Format: `MITRE ATT&CK: T1190 (Exploit Public-Facing Application)`.

#### Step 4.2h: MITRE ATLAS Technique Mapping (AI/LLM Findings Only)

For findings from Squads 14–15 (AI/ML Model Security and LLM & Prompt Injection) or any finding involving AI/ML components, additionally assign a MITRE ATLAS (Adversarial Threat Landscape for AI Systems) technique:

| Technique ID | Name | Typical Finding Types |
|-------------|------|----------------------|
| AML.T0051 | LLM Prompt Injection | Direct/indirect prompt injection, jailbreaking |
| AML.T0020 | Poison Training Data | Training data manipulation, data poisoning |
| AML.T0043 | Craft Adversarial Data | Adversarial inputs, evasion attacks |
| AML.T0024 | Exfiltration via ML API | Model extraction, data leakage through inference |
| AML.T0048 | External Harms | External harms caused by AI system actions or outputs |

If the finding involves AI components but does not map to an ATLAS technique, mark as "N/A — no direct ATLAS mapping."

For non-AI findings, skip this step entirely (do not include an ATLAS field in the enrichment output).

Format: `MITRE ATLAS: AML.T0051 (LLM Prompt Injection)`.

#### Step 4.2 Output Format

After enrichment, each validated finding should carry the following standards block:

```
Finding: {finding_id}
  CWE:          CWE-XXX (Weakness Name)
  CVSS 4.0:     X.X (estimated) — CVSS:4.0/AV:X/AC:X/AT:X/PR:X/UI:X/VC:X/VI:X/VA:X/SC:X/SI:X/SA:X
  OWASP Web:    A0X:2025 (Category Name) | N/A
  OWASP LLM:    LLM0X:2025 (Category Name) | N/A
  OWASP Agentic: ASI0X:2026 (Category Name) | N/A
  NIST 800-53:  XX-X (Control Name)
  NIST SSDF:    XX (Practice Group Name)
  STIG:         CAT X
  ATT&CK:       TXXXX (Technique Name)
  ATLAS:        AML.TXXXX (Technique Name) | skipped (non-AI finding)
```

Log:

> **Standards enrichment complete: {N} findings enriched with CWE, CVSS 4.0, OWASP, NIST 800-53, NIST SSDF, STIG, and MITRE ATT&CK mappings. {M} findings additionally mapped to MITRE ATLAS.**

### Step 4.3: Confidence Scoring

Assign each validated finding a confidence level that is INDEPENDENT of its severity. A Critical-severity finding can have LOW confidence, and a Low-severity finding can have HIGH confidence. Confidence reflects how certain we are that the finding is real, not how dangerous it would be if real.

**Confidence Levels:**

| Level | Criteria | Description |
|-------|----------|-------------|
| **HIGH** | All three conditions met: (1) file:line reference is grounded and verified, (2) multiple squads independently corroborated the finding or its attack surface, (3) the proof-of-exploit is verifiable by reading the code alone without executing it | The finding is almost certainly real. A human reviewer can confirm it by reading the cited code. |
| **MEDIUM** | Two conditions met: (1) file:line reference is grounded and verified, (2) finding comes from a single squad only, (3) the proof-of-exploit relies on assumptions about runtime behavior, configuration state, or environmental conditions that cannot be verified from code alone | The finding is likely real but depends on conditions that may or may not hold in the deployed environment. |
| **LOW** | One or more of: (1) finding comes from a single squad with no corroboration, (2) the proof requires runtime verification to confirm (e.g., timing-dependent, environment-dependent), (3) a counter-proof was raised in Phase 3 that was plausible but not definitive enough to eliminate the finding | The finding may be real but cannot be confirmed without additional testing. Human review and/or runtime verification is recommended. |

**Scoring process:**

1. For each validated finding, evaluate all three dimensions: grounding quality, corroboration breadth, and proof verifiability.
2. Assign the confidence level based on the criteria table above.
3. If a finding was downgraded during Phase 3 counter-proof analysis (Step 3.2), its confidence should generally be MEDIUM or LOW — a survived counter-proof indicates some uncertainty.
4. If a finding was corroborated by the merge/cluster analysis in Step 3.3 (multiple squads reported the same underlying issue), that is evidence supporting HIGH confidence.
5. Document the rationale for each confidence assignment in one sentence.

**Format:** Append to each finding's standards block:

```
  Confidence:   HIGH | MEDIUM | LOW — {one-sentence rationale}
```

Log:

> **Confidence scoring complete: {H} HIGH / {M} MEDIUM / {L} LOW across {N} findings.**

---

### Step 4.4: Generate In-Conversation Executive Summary (Artifact 1)

This is the primary artifact the user sees directly in the chat. It is the decision-making interface — every finding is presented here with enough context for the user to approve, reject, or defer.

**Structure:**

**1. Limitations Header** (from Step 4.1 — mandatory, verbatim, non-removable)

**2. Scope Classification Block**

Display the scope classification from Phase 0 Step 0.7:

```
┌─────────────────────────────────────────────────┐
│  SCOPE: {FULL | PARTIAL | FOCUSED | DIFF}       │
│  Target: {project name or path}                 │
│  Mode: {full | quick | focused | verify | diff} │
│  Timestamp: {ISO 8601}                          │
└─────────────────────────────────────────────────┘
```

**3. Hero Metrics**

Display a metrics dashboard summarizing audit coverage:

```
┌─────────────────────── AUDIT METRICS ───────────────────────┐
│  Squads Deployed:      {N}                                  │
│  Personas Activated:   {N} across {N} squads                │
│  Files Analyzed:       {N} / {total} ({percentage}%)        │
│  Lines of Code:        {N}                                  │
│  CWE Top 25 Coverage:  {N}/25                               │
│  OWASP Top 10 (Web):  {N}/10                                │
│  OWASP Top 10 (API):  {N}/10                                │
│  OWASP Top 10 (LLM):  {N}/10                                │
│  Validation Rate:      {N}% of raw findings survived Phase 3│
└─────────────────────────────────────────────────────────────┘
```

**4. Findings by Severity Tier**

Group findings by severity. Within each tier, sort by confidence (HIGH first), then by CVSS score descending. Display each finding as:

```
──── CRITICAL ────

[F-001] {Title}
  File:       {file}:{line}
  CWE:        CWE-{id} — {name}
  CVSS:       {score} ({vector})
  Confidence: {HIGH|MEDIUM|LOW} — {rationale}
  Squad:      {squad name}
  Summary:    {one-sentence description}
  Fix:        {one-sentence proposed fix}
  ▸ [Approve] [Reject] [Defer]

[F-002] ...

──── HIGH ────
...

──── MEDIUM ────
...

──── LOW ────
...

──── ENHANCEMENT ────
...

──── POSITIVE ────
{List of security strengths identified during the audit.
 These are NOT findings — they are things the codebase does RIGHT.
 No approval action required.}
```

**5. Fix Clusters**

If the fix dependency graph from Step 3.4 identified clusters of findings that share a common root cause or should be fixed together, list them:

```
FIX CLUSTERS:
  Cluster 1: F-001, F-003, F-007 — {shared root cause description}
  Cluster 2: F-004, F-005 — {shared root cause description}
  Standalone: F-002, F-006, F-008
```

**6. Coverage Gaps**

List areas that were NOT covered by this audit, from the coverage map in Step 3.5:

```
COVERAGE GAPS:
  - {file or directory}: {reason not covered — e.g., excluded by scope, no squad matched, binary file}
  - {category}: {reason — e.g., no runtime testing, no dependency audit tool available}
```

**7. Per-Finding Approval Prompts**

After presenting all findings, prompt the user for decisions:

```
═══════════════════════════════════════════════════════════════
  APPROVAL REQUIRED — {N} findings await your decision
═══════════════════════════════════════════════════════════════

For each finding, respond with:
  Approve {ID}   — Proceed with proposed fix
  Reject {ID}    — Accept risk (you will be asked for rationale)
  Defer {ID}     — Postpone (you will be asked for milestone date)

You may also use bulk actions:
  Approve all
  Approve all CRITICAL
  Reject {ID} {ID} {ID}
  Defer {ID} to {date}
```

Log:

> **Executive summary presented: {C} critical, {H} high, {M} medium, {L} low, {E} enhancement, {P} positive findings. Awaiting user decisions.**

### Step 4.5: Generate Detailed Markdown Report (Artifact 2)

Write to `.fortress/reports/YYYY-MM-DD.md` where `YYYY-MM-DD` is the current date.

This report contains EVERYTHING from the executive summary PLUS full technical detail. It is the archival record of the audit.

**Structure:**

```markdown
# FORTRESS Security Audit Report
## {Project Name} — {Date}

{Limitations Header from Step 4.1 — verbatim, non-removable}

## 1. Methodology

### Audit Model
- **Framework:** FORTRESS Protocol v{version}
- **Threat Model:** STRIDE (Spoofing, Tampering, Repudiation, Information Disclosure, Denial of Service, Elevation of Privilege)
- **Mode:** {full | quick | focused | verify | diff}
- **Scope:** {FULL | PARTIAL | FOCUSED | DIFF}

### Squads Deployed
| Squad | Personas | Files Assigned | Findings Reported | Findings Validated |
|-------|----------|----------------|-------------------|--------------------|
| {name} | {N} | {N} | {N} | {N} |

### Detection Heuristics Triggered
- {heuristic name}: {trigger reason}

### Validation Pipeline
- Raw findings from Phase 2: {N}
- Eliminated by grounding checks (Step 3.1): {N}
- Eliminated by counter-proof (Step 3.2): {N}
- Merged by deduplication (Step 3.3): {N}
- Final validated findings: {N}

## 2. Findings

### CRITICAL

#### [F-001] {Title}

| Property | Value |
|----------|-------|
| File | `{file}:{line}` |
| CWE | CWE-{id} — {name} |
| CVSS | {score} ({vector string}) |
| Confidence | {HIGH\|MEDIUM\|LOW} |
| OWASP | {Web: A0X / API: APIX / LLM: LLMX} |
| NIST 800-53 | {control(s)} |
| NIST SSDF | {practice(s)} |
| STIG CAT | {I\|II\|III} |
| ATT&CK | {technique(s)} |
| ATLAS | {technique(s) or N/A} |
| Squad | {squad name} |

**Description:**
{Detailed description of the vulnerability}

**Proof of Exploit:**
```{language}
{The actual proof-of-exploit code or step sequence from Phase 2}
```

**Counter-Proof Analysis:**
{The counter-proof reasoning from Phase 3 Step 3.2 — why did this finding survive?}

**Proposed Fix:**
```{language}
{The proposed fix code}
```

**Fix Rationale:**
{Why this fix addresses the root cause, not just the symptom}

---

{Repeat for each finding in each severity tier}

## 3. Fix Clusters
{From Step 3.4 — fix dependency graph with recommended application order}

## 4. Coverage Gaps
{From Step 3.5 — what was not covered and why}

## 5. SBOM Reference
See: `.fortress/reports/YYYY-MM-DD-sbom.json`

## 6. Standards Mapping Summary
{Aggregate view: how many findings map to each CWE, each OWASP category, each NIST control}

## 7. Positive Findings
{Security strengths identified — what the codebase does well}
```

After writing, log:

> **Detailed report written to `.fortress/reports/{date}.md` — {N} findings, {N} pages.**

### Step 4.6: Generate SARIF File (Artifact 3)

Write to `.fortress/reports/YYYY-MM-DD.sarif` where `YYYY-MM-DD` is the current date.

SARIF (Static Analysis Results Interchange Format) v2.1.0 enables integration with GitHub Code Scanning, VS Code SARIF Viewer, Azure DevOps, and other security tooling.

**Template:**

```json
{
  "$schema": "https://raw.githubusercontent.com/oasis-tcs/sarif-spec/main/sarif-2.1/schema/sarif-schema-2.1.0.json",
  "version": "2.1.0",
  "runs": [
    {
      "tool": {
        "driver": {
          "name": "FORTRESS",
          "version": "{protocol version}",
          "informationUri": "https://github.com/{repo}/FORTRESS",
          "rules": [
            {
              "id": "CWE-{id}",
              "name": "{CWE name}",
              "shortDescription": {
                "text": "CWE-{id}: {CWE name}"
              },
              "fullDescription": {
                "text": "{CWE description}"
              },
              "helpUri": "https://cwe.mitre.org/data/definitions/{id}.html",
              "properties": {
                "tags": ["security", "CWE-{id}"]
              }
            }
          ]
        }
      },
      "results": [
        {
          "ruleId": "CWE-{id}",
          "level": "{error|warning|note}",
          "message": {
            "text": "[F-{NNN}] {title}: {description}"
          },
          "locations": [
            {
              "physicalLocation": {
                "artifactLocation": {
                  "uri": "{file/path/normalized/to/forward/slashes}",
                  "uriBaseId": "%SRCROOT%"
                },
                "region": {
                  "startLine": {line},
                  "startColumn": 1
                }
              }
            }
          ],
          "properties": {
            "fortress-id": "F-{NNN}",
            "severity": "{CRITICAL|HIGH|MEDIUM|LOW|ENHANCEMENT}",
            "cvss-score": {score},
            "cvss-vector": "{vector string}",
            "confidence": "{HIGH|MEDIUM|LOW}",
            "squad": "{squad name}",
            "fix-cluster": "{cluster id or null}"
          }
        }
      ]
    }
  ]
}
```

**Severity to SARIF level mapping:**

| FORTRESS Severity | SARIF Level |
|-------------------|-------------|
| CRITICAL | `error` |
| HIGH | `error` |
| MEDIUM | `warning` |
| LOW | `note` |
| ENHANCEMENT | `note` |

**Rules array construction:**
- One entry per unique CWE across all findings.
- If multiple findings share a CWE, the CWE appears once in `rules` and each finding references it via `ruleId`.

**Path normalization:**
- All file paths in `artifactLocation.uri` MUST use forward slashes (`/`), regardless of the host OS.
- Paths MUST be relative to the project root.
- Strip any leading `./` prefix.

**Self-validation:**
After generating the SARIF file, read it back and verify:
1. It parses as valid JSON.
2. `$schema` and `version` fields are present.
3. `runs` array is non-empty.
4. Every result has a `ruleId` that matches an entry in `rules`.
5. Every result has at least one location with a valid `uri`.

If validation fails, fix the issue and re-write the file. Log:

> **SARIF report written to `.fortress/reports/{date}.sarif` — {N} results, {N} rules. Self-validation: PASSED.**

### Step 4.7: Generate Compliance Posture Summary (Artifact 5)

Write to `.fortress/reports/YYYY-MM-DD-compliance.md`.

This artifact maps audit findings to compliance frameworks, giving the user a clear picture of where their project stands relative to enterprise security requirements.

> **Note:** Artifact 4 (SBOM) was already generated in Phase 0 Step 0.4.

**Structure:**

```markdown
# Compliance Posture Summary
## {Project Name} — {Date}

{Limitations Header from Step 4.1}

> **IMPORTANT:** This compliance mapping is based on static code analysis only.
> It does not constitute a formal compliance assessment or certification.
> Formal compliance requires authorized assessors and operational evidence
> beyond what code analysis alone can provide.

## NIST 800-53 Rev 5 — Security Controls

### Controls Addressed by Findings
| Control | Family | Finding(s) | Status |
|---------|--------|------------|--------|
| {AC-X} | Access Control | F-{NNN} | {Finding exists — remediation recommended} |

### Controls Not Addressed
| Control | Family | Reason |
|---------|--------|--------|
| {PE-X} | Physical & Environmental | {Out of scope for static analysis} |

### Coverage Summary
- Controls with findings: {N}
- Controls not applicable to static analysis: {N}
- Controls not covered (gap): {N}

## CMMC 2.0 Level 2

### Practices Covered
| Practice | Domain | Finding(s) | Status |
|----------|--------|------------|--------|
| {AC.L2-3.1.1} | Access Control | F-{NNN} | {Addressed} |

### Practices Not Covered (Gaps)
| Practice | Domain | Reason |
|----------|--------|--------|
| {PE.L2-3.10.1} | Physical Protection | {Requires physical assessment} |

### Coverage Summary
- Practices covered: {N} / 110
- Practices not applicable: {N}
- Practices with gaps: {N}

## OWASP Coverage

### OWASP Top 10 Web (2025)
| Category | Covered | Finding(s) |
|----------|---------|------------|
| A01:2025 Broken Access Control | {Yes/No/N/A} | {F-NNN or —} |
| A02:2025 Security Misconfiguration | {Yes/No/N/A} | {F-NNN or —} |
| ... | | |

**Score: {N}/10**

### OWASP Top 10 API (2023)
| Category | Covered | Finding(s) |
|----------|---------|------------|
| API1:2023 Broken Object Level Authorization | {Yes/No/N/A} | {F-NNN or —} |
| ... | | |

**Score: {N}/10**

### OWASP Top 10 LLM (2025)
| Category | Covered | Finding(s) |
|----------|---------|------------|
| LLM01:2025 Prompt Injection | {Yes/No/N/A} | {F-NNN or —} |
| ... | | |

**Score: {N}/10**

## CWE Top 25 (2025) Coverage Matrix

| Rank | CWE | Name | Covered | Finding(s) | Status |
|------|-----|------|---------|------------|--------|
| 1 | CWE-787 | Out-of-bounds Write | {Yes/No/N/A} | {F-NNN or —} | {Tested — no finding / Finding reported / Not applicable to stack} |
| 2 | CWE-79 | Cross-site Scripting | ... | | |
| ... | | | | | |

**Coverage: {N}/25** ({N} tested, {N} not applicable to this stack)

## DISA STIG — Application Security and Development (ASD)

- **Estimated Coverage:** {N}%
- **CAT I findings (Critical):** {N}
- **CAT II findings (High/Medium):** {N}
- **CAT III findings (Low):** {N}
- **Controls not testable via static analysis:** {N}

## Recommendations for Complementary Testing

Based on the gaps identified above, the following additional testing is recommended:

1. **{Type of testing}** — {What it would cover that this audit did not}
2. **{Type of testing}** — {What it would cover}
3. ...

Common recommendations include:
- Dynamic Application Security Testing (DAST) for runtime behavior
- Penetration testing for network-level and authentication flow testing
- Formal compliance assessment by authorized assessors
- Supply chain security audit (beyond SBOM generation)
- Infrastructure security review (cloud configuration, IAM policies)
```

Log:

> **Compliance posture summary written to `.fortress/reports/{date}-compliance.md` — {N} frameworks mapped.**

### Step 4.8: Generate POA&M Template (Artifact 6)

Write to `.fortress/reports/YYYY-MM-DD-poam.md`.

A Plan of Action and Milestones (POA&M) is a standard artifact in federal and enterprise security compliance. This template is pre-populated for any findings the user rejects or defers during the approval gate.

> **Note:** This artifact is written AFTER Step 4.12 (Approval Gate 2), because it depends on user decisions. However, it is defined here for completeness.

**Structure:**

```markdown
# Plan of Action & Milestones (POA&M)
## {Project Name} — {Date}

{Limitations Header from Step 4.1}

> This POA&M was auto-generated by FORTRESS based on user decisions
> during the audit approval process. It should be reviewed and updated
> by the responsible security officer.

## Deferred Findings

| ID | Title | Severity | CWE | CVSS | Milestone Date | Responsible Party | Status | Notes |
|----|-------|----------|-----|------|---------------|-------------------|--------|-------|
| F-{NNN} | {title} | {severity} | CWE-{id} | {score} | {user-provided date} | {user-provided or TBD} | DEFERRED | {any notes from user} |

## Rejected Findings (Risk Accepted)

| ID | Title | Severity | CWE | CVSS | Risk Acceptance Rationale | Accepted By | Date |
|----|-------|----------|-----|------|--------------------------|-------------|------|
| F-{NNN} | {title} | {severity} | CWE-{id} | {score} | {user-provided rationale} | {user identity or "project owner"} | {date} |

## Summary

- Total deferred: {N}
- Total risk-accepted: {N}
- Earliest milestone: {date}
- Latest milestone: {date}
- Highest severity deferred: {severity}
- Highest severity risk-accepted: {severity}

## Review Schedule

This POA&M should be reviewed:
- **Monthly** for any CRITICAL or HIGH severity deferred items
- **Quarterly** for MEDIUM severity deferred items
- **Semi-annually** for LOW severity deferred items
- **Immediately** if a deferred finding's threat landscape changes
```

Log:

> **POA&M template written to `.fortress/reports/{date}-poam.md` — {N} deferred, {N} risk-accepted.**

### Step 4.9: Generate Public Security Page Template (Artifact 7)

Write to `.fortress/reports/YYYY-MM-DD-security-page.html`.

This is an HTML template inspired by a project's public security tab. It is designed to be embedded in a project's documentation site or served standalone as a transparency artifact. It does NOT include specific vulnerability details — only aggregate posture information.

**Template:**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{Project Name} — Security Posture</title>
  <style>
    :root {
      --bg: #0a0a0a;
      --surface: #141414;
      --border: #2a2a2a;
      --text: #e0e0e0;
      --text-muted: #888;
      --accent: #00d4aa;
      --critical: #ff4444;
      --high: #ff8800;
      --medium: #ffcc00;
      --low: #44aaff;
      --positive: #00cc66;
    }
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
      background: var(--bg);
      color: var(--text);
      line-height: 1.6;
    }
    .container { max-width: 960px; margin: 0 auto; padding: 2rem; }
    .hero {
      text-align: center;
      padding: 3rem 0;
      border-bottom: 1px solid var(--border);
    }
    .hero h1 { font-size: 2rem; margin-bottom: 0.5rem; }
    .hero .subtitle { color: var(--text-muted); font-size: 1.1rem; }
    .metrics {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
      gap: 1rem;
      padding: 2rem 0;
    }
    .metric {
      background: var(--surface);
      border: 1px solid var(--border);
      border-radius: 8px;
      padding: 1.5rem;
      text-align: center;
    }
    .metric .value {
      font-size: 2.5rem;
      font-weight: 700;
      color: var(--accent);
    }
    .metric .label {
      color: var(--text-muted);
      font-size: 0.85rem;
      text-transform: uppercase;
      letter-spacing: 0.05em;
    }
    .section { padding: 2rem 0; border-bottom: 1px solid var(--border); }
    .section h2 {
      font-size: 1.4rem;
      margin-bottom: 1rem;
      color: var(--accent);
    }
    .methodology-grid {
      display: grid;
      grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
      gap: 1rem;
    }
    .phase-card {
      background: var(--surface);
      border: 1px solid var(--border);
      border-radius: 8px;
      padding: 1.25rem;
    }
    .phase-card h3 { font-size: 1rem; margin-bottom: 0.5rem; }
    .phase-card p { color: var(--text-muted); font-size: 0.9rem; }
    .layer-group { margin-bottom: 1.5rem; }
    .layer-group h3 {
      font-size: 1.1rem;
      margin-bottom: 0.75rem;
      padding-bottom: 0.25rem;
      border-bottom: 1px solid var(--border);
    }
    .layer-list { list-style: none; }
    .layer-list li {
      padding: 0.4rem 0;
      padding-left: 1.5rem;
      position: relative;
      color: var(--text-muted);
      font-size: 0.95rem;
    }
    .layer-list li::before {
      content: '✓';
      position: absolute;
      left: 0;
      color: var(--positive);
    }
    .manifesto {
      text-align: center;
      padding: 3rem 0;
      font-style: italic;
      color: var(--text-muted);
    }
    .manifesto .quote {
      font-size: 1.3rem;
      color: var(--text);
      margin-bottom: 0.5rem;
      font-style: normal;
      font-weight: 600;
    }
    .footer {
      text-align: center;
      padding: 1.5rem 0;
      color: var(--text-muted);
      font-size: 0.8rem;
    }
  </style>
</head>
<body>
  <div class="container">
    <div class="hero">
      <h1>🛡️ {Project Name} Security Posture</h1>
      <p class="subtitle">Audited with FORTRESS Protocol — {Date}</p>
    </div>

    <div class="metrics">
      <div class="metric">
        <div class="value">{N}</div>
        <div class="label">Attack Scenarios Tested</div>
      </div>
      <div class="metric">
        <div class="value">{N}</div>
        <div class="label">Security Squads Deployed</div>
      </div>
      <div class="metric">
        <div class="value">{N}</div>
        <div class="label">Verification Layers</div>
      </div>
      <div class="metric">
        <div class="value">{N}</div>
        <div class="label">Audit Rounds</div>
      </div>
      <div class="metric">
        <div class="value">{N}</div>
        <div class="label">Critical Findings Resolved</div>
      </div>
    </div>

    <div class="section">
      <h2>Methodology</h2>
      <div class="methodology-grid">
        <div class="phase-card">
          <h3>Phase 1: Ingestion</h3>
          <p>Automated stack detection, dependency mapping, and threat modeling using STRIDE framework.</p>
        </div>
        <div class="phase-card">
          <h3>Phase 2: Adversarial Testing</h3>
          <p>Specialized security squads with {N}+ expert personas probe every attack surface.</p>
        </div>
        <div class="phase-card">
          <h3>Phase 3: Proof Validation</h3>
          <p>Every finding requires proof-of-exploit. Counter-proofs eliminate false positives.</p>
        </div>
        <div class="phase-card">
          <h3>Phase 4: Verification</h3>
          <p>Standards mapping (CWE, CVSS, OWASP, NIST), compliance posture, and fix verification.</p>
        </div>
      </div>
    </div>

    <div class="section">
      <h2>Security Layers</h2>

      <!-- Adapt these groups to the actual project's domain -->

      <div class="layer-group">
        <h3>{Domain 1 — e.g., Authentication & Access Control}</h3>
        <ul class="layer-list">
          <li>{Security measure or control verified}</li>
          <li>{Security measure or control verified}</li>
        </ul>
      </div>

      <div class="layer-group">
        <h3>{Domain 2 — e.g., Data Protection}</h3>
        <ul class="layer-list">
          <li>{Security measure or control verified}</li>
          <li>{Security measure or control verified}</li>
        </ul>
      </div>

      <div class="layer-group">
        <h3>{Domain 3 — e.g., Input Validation}</h3>
        <ul class="layer-list">
          <li>{Security measure or control verified}</li>
          <li>{Security measure or control verified}</li>
        </ul>
      </div>

      <!-- Add more domain groups as appropriate for the project -->
    </div>

    <div class="manifesto">
      <p class="quote">"Security is not a feature — it is the foundation."</p>
      <p>Built with the FORTRESS Protocol — adversarial by design, antifragile by practice.</p>
    </div>

    <div class="footer">
      <p>Generated by FORTRESS v{version} on {date}. This page reflects aggregate security posture only — specific vulnerability details are restricted to authorized personnel.</p>
    </div>
  </div>
</body>
</html>
```

**Customization rules:**
1. Security layer categories MUST be adapted to the actual project's domain. Do not use generic categories — analyze the project's stack from Phase 0 recon and group findings and positive observations into meaningful domain categories.
2. Metric values must reflect actual audit data, not placeholders.
3. The "Audit Rounds" metric should check `.fortress/last-audit.md` to determine if this is round 1 or a subsequent round.
4. Do NOT include specific vulnerability details, file paths, or exploit information in this public-facing page.

Log:

> **Public security page template written to `.fortress/reports/{date}-security-page.html`.**

### Step 4.10: Generate Delta Report (Artifact 8)

Write to `.fortress/reports/YYYY-MM-DD-delta.md`.

**This artifact is CONDITIONAL** — only generate it if `.fortress/last-audit.md` exists (indicating a previous audit was run).

If `.fortress/last-audit.md` does not exist, skip this step and log:

> **Delta report skipped — no prior audit found in `.fortress/last-audit.md`.**

If it does exist, read the previous audit data and compare:

**Structure:**

```markdown
# Delta Report: {Project Name}
## {Current Date} vs {Previous Date}

{Limitations Header from Step 4.1}

## Trend: {IMPROVING | STABLE | DECLINING}

**Rationale:** {One-sentence explanation of the trend determination}

## Summary

| Metric | Previous | Current | Change |
|--------|----------|---------|--------|
| Total findings | {N} | {N} | {+/-N} |
| Critical | {N} | {N} | {+/-N} |
| High | {N} | {N} | {+/-N} |
| Medium | {N} | {N} | {+/-N} |
| Low | {N} | {N} | {+/-N} |
| Enhancement | {N} | {N} | {+/-N} |
| Files analyzed | {N} | {N} | {+/-N} |
| CWE coverage | {N}/25 | {N}/25 | {+/-N} |

## New Findings
{Findings that appear in this audit but not in the previous one}

| ID | Title | Severity | CWE | Notes |
|----|-------|----------|-----|-------|
| F-{NNN} | {title} | {severity} | CWE-{id} | {New code, new dependency, previously missed, etc.} |

## Resolved Findings
{Findings from the previous audit that no longer appear — they were fixed}

| Previous ID | Title | Severity | CWE | Resolution |
|-------------|-------|----------|-----|------------|
| {prev-id} | {title} | {severity} | CWE-{id} | {Fixed in commit X, dependency updated, code removed, etc.} |

## Regressed Findings
{Findings that were resolved in a previous audit but have reappeared}

| ID | Title | Severity | CWE | Previous Resolution | Regression Cause |
|----|-------|----------|-----|--------------------|--------------------|
| F-{NNN} | {title} | {severity} | CWE-{id} | {how it was previously fixed} | {what caused it to reappear} |

## Unchanged Deferred Findings
{Findings that were deferred in the previous audit and remain unaddressed}

| ID | Title | Severity | Original Milestone | Status |
|----|-------|----------|--------------------|--------|
| F-{NNN} | {title} | {severity} | {date} | {OVERDUE / ON TRACK / APPROACHING} |

## Overdue Deferred Findings
{Deferred findings whose milestone date has passed}

| ID | Title | Severity | Milestone | Days Overdue |
|----|-------|----------|-----------|-------------|
| F-{NNN} | {title} | {severity} | {date} | {N} |
```

**Trend determination logic:**
- **IMPROVING:** More findings resolved than new findings introduced, AND no regressions, AND no overdue deferred findings.
- **STABLE:** Roughly equal new vs resolved, OR minor changes in either direction, AND no regressions.
- **DECLINING:** More new findings than resolved, OR any regressions found, OR overdue deferred findings with CRITICAL/HIGH severity.

Log:

> **Delta report written to `.fortress/reports/{date}-delta.md` — Trend: {IMPROVING|STABLE|DECLINING}. {N} new, {N} resolved, {N} regressed, {N} overdue deferred.**

### Step 4.11: Generate Security Posture Snapshot (Artifact 9)

Write to `.fortress/reports/YYYY-MM-DD-snapshot.md`.

This is a one-page executive snapshot designed for leadership review. It answers one question: "Should we authorize this system to operate?"

**Structure:**

```markdown
# Security Posture Snapshot
## {Project Name} — {Date}

{Limitations Header from Step 4.1}

## Authorization Recommendation

### **{AUTHORIZE | AUTHORIZE WITH CONDITIONS | REMEDIATE FIRST}**

**Rationale:** {2-3 sentence justification based on findings}

## Findings Overview

| Severity | Count | Trend |
|----------|-------|-------|
| CRITICAL | {N} | {↑ ↓ → or N/A if first audit} |
| HIGH | {N} | {↑ ↓ →} |
| MEDIUM | {N} | {↑ ↓ →} |
| LOW | {N} | {↑ ↓ →} |
| ENHANCEMENT | {N} | {↑ ↓ →} |

**Total: {N} findings ({N} validated from {N} raw)**

## Top 3 Risks

1. **{Risk title}** — {one-sentence description} (Severity: {X}, Confidence: {Y})
2. **{Risk title}** — {one-sentence description} (Severity: {X}, Confidence: {Y})
3. **{Risk title}** — {one-sentence description} (Severity: {X}, Confidence: {Y})

## Compliance Coverage

| Framework | Coverage | Notes |
|-----------|----------|-------|
| NIST 800-53 | {N} controls addressed | {key gap} |
| CMMC 2.0 L2 | {N}/110 practices | {key gap} |
| OWASP Web | {N}/10 | — |
| OWASP API | {N}/10 | — |
| OWASP LLM | {N}/10 | — |
| CWE Top 25 | {N}/25 | — |
| DISA STIG ASD | ~{N}% | — |

## Recommendation

{One paragraph: what needs to happen before the next authorization review}
```

**Authorization logic:**
- **AUTHORIZE:** Zero CRITICAL findings, zero HIGH findings with HIGH confidence, and no overdue deferred findings.
- **AUTHORIZE WITH CONDITIONS:** Zero CRITICAL findings, but HIGH findings exist OR deferred findings approaching milestone, with specific conditions listed.
- **REMEDIATE FIRST:** Any CRITICAL findings exist, OR HIGH findings with HIGH confidence that have known exploits.

Log:

> **Security posture snapshot written to `.fortress/reports/{date}-snapshot.md` — Recommendation: {AUTHORIZE|AUTHORIZE WITH CONDITIONS|REMEDIATE FIRST}.**

### Step 4.12: Finalize Execution Log (Artifact 10)

The execution log has been accumulating entries since Phase 0. At this point, append the Phase 4 entries (standards enrichment decisions, artifact generation status) and write any intermediate summary.

The execution log is finalized at the very end of the audit (after Phase 6 in full mode, or after Phase 4 in quick mode) with the end-of-audit summary block and self-diagnostic flags defined in the Execution Logging section of Master Orchestration.

Log:

> **Execution log updated through Phase 4. Final summary will be appended at audit completion.**

### Step 4.13: Present for Approval (APPROVAL GATE 2)

This is the second and final approval gate. The user has now seen the executive summary (Step 4.4) and must make decisions on every finding before FORTRESS proceeds.

**Process:**

1. Display the executive summary from Step 4.4 in the conversation.

2. Wait for user input. The user will respond with decisions for each finding using the format specified in Step 4.4.

3. Process each decision:

**For APPROVED findings:**
- Mark finding status as `APPROVED`
- Add to Phase 5 execution queue
- Log: `F-{NNN}: APPROVED — queued for fix application`

**For REJECTED findings:**
- Prompt: `F-{NNN} rejected. Please provide a risk acceptance rationale (this will be recorded in the POA&M):`
- Wait for user response
- Record the rationale
- Mark finding status as `REJECTED`
- Log: `F-{NNN}: REJECTED — risk accepted. Rationale: "{rationale}"`

**For DEFERRED findings:**
- Prompt: `F-{NNN} deferred. Please provide: (1) Milestone date for remediation, (2) Responsible party (or TBD):`
- Wait for user response
- Record milestone date and responsible party
- Mark finding status as `DEFERRED`
- Log: `F-{NNN}: DEFERRED — milestone: {date}, responsible: {party}`

4. After all decisions are collected, generate the POA&M (Step 4.8) with any rejected/deferred findings.

5. Determine next phase:

```
IF all findings are REJECTED or DEFERRED:
  → Skip Phase 5 entirely
  → Log: "All findings rejected/deferred. No fixes to apply. Proceeding to Phase 6 (Debrief)."
  → Proceed to Phase 6

ELSE IF any findings are APPROVED:
  → Log: "{N} findings approved for remediation. Proceeding to Phase 5 (Execute)."
  → Proceed to Phase 5 with the approved findings queue
```

6. Final log for Phase 4:

> **Phase 4 complete. Artifact suite generated:**
> - **Artifact 1:** Executive Summary (in-conversation)
> - **Artifact 2:** Detailed Report (`.fortress/reports/{date}.md`)
> - **Artifact 3:** SARIF File (`.fortress/reports/{date}.sarif`)
> - **Artifact 4:** SBOM (`.fortress/reports/{date}-sbom.json` — generated in Phase 0)
> - **Artifact 5:** Compliance Posture (`.fortress/reports/{date}-compliance.md`)
> - **Artifact 6:** POA&M (`.fortress/reports/{date}-poam.md`)
> - **Artifact 7:** Public Security Page (`.fortress/reports/{date}-security-page.html`)
> - **Artifact 8:** Delta Report (`.fortress/reports/{date}-delta.md` — {generated|skipped})
> - **Artifact 9:** Security Posture Snapshot (`.fortress/reports/{date}-snapshot.md`)
> - **Artifact 10:** Execution Log (`.fortress/reports/{date}-execution-log.md`)
>
> **Decisions: {A} approved, {R} rejected, {D} deferred. {Next phase action}.**

## Phase 5: EXECUTE

**Goal:** Implement approved fixes safely, in priority order, with git safety, syntax verification after each fix, and automatic rollback on failure. Phase 5 converts the approved findings from Gate 2 into actual code changes — applied one at a time (or in fix clusters) with verification at every step. Discovery was parallel; execution is serial.

Phase 5 only executes if at least one finding was APPROVED at Gate 2. If all findings were REJECTED or DEFERRED, Phase 5 is skipped entirely and control passes to Phase 6 (Debrief).

### Step 5.1: Git Safety Setup

Before modifying any code, establish version control safety so every change can be rolled back.

**Check 1 — Git repository detection:**

Run `git status` via the Bash tool to determine whether the current directory is inside a git repository.

- If git is available and the directory is a git repo: proceed to Check 2.
- If git is not available or the directory is NOT a git repo: display the following warning and wait for user input:

> **WARNING: No git repository detected.** FORTRESS strongly recommends version control for all code changes. Without git:
> - Individual fix rollback is not possible
> - The pre-audit checkpoint cannot be created
> - Integration verification diff will be unavailable
>
> **Proceed without version control? (yes / no)**

If the user says "no": abort Phase 5 entirely. Log: `"Phase 5 aborted: user declined to proceed without version control."` Skip to Phase 6.

If the user says "yes": set an internal flag `git_available = false` and proceed. All git-dependent steps (branching, committing, reverting) will be skipped, but fixes will still be applied with syntax verification.

**Check 2 — Clean working tree:**

If git is available, run `git status --porcelain` to check for uncommitted changes.

- If the working tree is clean: proceed to branch creation.
- If there are uncommitted changes: run `git stash push -m "fortress-pre-audit-stash"` to save them. Log: `"Stashed uncommitted changes before audit branch creation."` The stash will be popped after Phase 6 completes.

**Check 3 — Create audit branch:**

Run:
```
git checkout -b fortress/audit-YYYY-MM-DD
```

Where `YYYY-MM-DD` is today's date. If the branch already exists (e.g., a second audit on the same day), append a counter: `fortress/audit-YYYY-MM-DD-2`.

**Check 4 — Create rollback checkpoint:**

Run:
```
git add -A && git commit -m "checkpoint: pre-FORTRESS audit" --allow-empty
```

This creates a known-good commit that Phase 5b can diff against and that "revert all" can reset to. The `--allow-empty` flag ensures the checkpoint commit is created even if there are no changes to stage (the working tree may already be clean).

Log:

> **Git safety established.** Branch: `fortress/audit-YYYY-MM-DD`. Rollback checkpoint committed. All changes from this point can be reverted with `git reset --hard HEAD~N` or by checking out the checkpoint commit.

If `git_available = false`, skip all git operations and log:

> **Proceeding without version control.** Fix rollback is not available. Each fix will be applied with syntax verification only.

### Step 5.2: Determine Fix Order

Sort the approved findings into the order they will be applied. The ordering rules prevent line-shift interference and ensure critical issues are addressed first.

**Rule 1 — Severity priority:**

Sort approved findings by severity in descending order:
1. CRITICAL
2. HIGH
3. MEDIUM
4. LOW

**Rule 2 — Fix cluster grouping:**

Review the fix dependency graph built in Phase 3 (Step 3.5). If any approved findings belong to the same fix cluster (connected component in the dependency graph), they MUST be applied together as a single atomic unit. Group clustered findings and treat each cluster as one execution unit.

For each fix cluster:
- All findings in the cluster are applied in sequence before moving to the next cluster or standalone fix
- The cluster's effective severity is the highest severity among its member findings (e.g., a cluster with one CRITICAL and two MEDIUM findings is treated as CRITICAL for ordering purposes)
- If the cluster spans multiple files, order the files alphabetically, then order fixes within each file by line number descending

**Rule 3 — Bottom-up line ordering:**

Within each file, apply fixes in DESCENDING line number order (highest line number first, lowest last). This prevents earlier fixes from shifting the line numbers of later fixes. This rule applies both within individual files in a cluster and for standalone fixes targeting the same file.

**Output:** An ordered execution queue displayed to the user:

> **Fix execution order:**
> 1. `F-{NNN}` — {title} ({severity}) — `{file}:{line}`
> 2. `F-{NNN}` — {title} ({severity}) — `{file}:{line}`
> 3. **[CLUSTER]** `F-{NNN}`, `F-{NNN}` — {cluster description} ({effective severity})
>    - `{file}:{line}` — {fix summary}
>    - `{file}:{line}` — {fix summary}
> ...

### Step 5.3: Apply Fixes

Process each item in the execution queue in order. For each fix (or fix cluster), follow this procedure exactly.

**Step 5.3a: Read current file state**

Use the Read tool to read the current contents of the target file(s). This ensures the fix is applied against the actual current state, not a stale version from an earlier phase.

If the file contents have changed since Phase 2/3 (e.g., a prior fix in Phase 5 modified the same file), verify that the fix's `old_string` still exists in the file. If it does not:
- Log: `"F-{NNN}: Target code no longer matches — file was modified by a prior fix."`
- Attempt to locate the equivalent code in the modified file by searching for the key identifiers from the original `old_string`.
- If found at a new location: adapt the fix to the new context. Log the adaptation.
- If not found: skip the fix. Log: `"F-{NNN}: SKIPPED — target code no longer present after prior fixes. Manual application required."` Ask the user whether to continue with the remaining queue or stop.

**Step 5.3b: Apply the fix**

Use the Edit tool with exact `old_string` and `new_string` parameters derived from the finding's `fix_code`.

- The `old_string` must match the current file content exactly (including whitespace and indentation).
- The `new_string` is the fixed code from the finding, adapted if necessary per Step 5.3a.
- If the Edit tool reports a failure (e.g., `old_string` not found, not unique): log the failure and treat it as a syntax failure (proceed to Step 5.3c revert path).

For fix clusters: apply each member fix in sequence within the cluster before running verification.

**Step 5.3c: Verify syntax**

After applying the fix (or all fixes in a cluster), run a syntax/type check appropriate to the detected language and tooling from Phase 0:

| Language/Stack | Verification Command | Notes |
|---|---|---|
| TypeScript/JavaScript | `npx tsc --noEmit` | Type-checks without emitting files. If no `tsconfig.json`, use `npx tsc --noEmit --allowJs --checkJs {file}` |
| Python | `python -m py_compile {file}` | Per-file syntax check. If `mypy` is available, also run `mypy {file}` |
| Rust | `cargo check` | Project-wide type check without building |
| Go | `go vet ./...` | Examines code for suspicious constructs |
| Java | `javac -d /dev/null {file}` | Compile check without output |
| C/C++ | `make -n` or `cmake --build . --target check` | Dry-run build or check target if available |
| Solidity | `npx hardhat compile` or `forge build` | Depends on detected toolchain |
| Ruby | `ruby -c {file}` | Syntax check only |
| PHP | `php -l {file}` | Lint check |
| Other | Skip syntax check | Log: `"No syntax verifier available for {language}. Fix applied without automated verification."` |

Run the appropriate command via the Bash tool.

**If syntax verification PASSES:**

1. If `git_available = true`: commit the change:
   ```
   git add {file(s)} && git commit -m "fortress: fix F-{NNN} — {title}"
   ```
   For clusters: `"fortress: fix cluster [F-{NNN}, F-{NNN}] — {cluster description}"`
2. Log: `"F-{NNN}: Fix applied and verified. ✓"`
3. Proceed to the next item in the execution queue.

**If syntax verification FAILS:**

1. Log the exact error output from the verification command.
2. If `git_available = true`: revert the change:
   ```
   git checkout -- {file(s)}
   ```
3. If `git_available = false`: the fix cannot be automatically reverted. Log: `"WARNING: Fix reverted manually is not possible without git. File may be in an inconsistent state."`
4. Display to the user:

> **Fix F-{NNN} failed syntax verification.**
> Error: {error output}
>
> Options:
> - **skip** — Skip this fix and continue with remaining queue
> - **stop** — Stop Phase 5 execution. Applied fixes so far are preserved.

5. Wait for user input. If "skip": log `"F-{NNN}: SKIPPED — syntax verification failed."` and continue. If "stop": log `"Phase 5 halted by user after syntax failure on F-{NNN}."` and proceed to Phase 5b with whatever fixes have been applied so far.

### Step 5.4: Run Build

After all fixes in the execution queue have been processed (applied, skipped, or stopped), attempt to run the project's build command to verify the combined changes compile and build correctly.

**Detect build command:**

Check for build tooling in this priority order:

1. `package.json` with `scripts.build` → `npm run build` (or `yarn build` / `pnpm build` based on lockfile)
2. `Cargo.toml` → `cargo build`
3. `pyproject.toml` with build system → `python -m build` or detected build command
4. `Makefile` → `make`
5. `go.mod` → `go build ./...`
6. `build.gradle` / `pom.xml` → `gradle build` / `mvn compile`
7. No build system detected → skip this step

**Run the build:**

Execute the detected build command via the Bash tool with a timeout of 120 seconds.

- If the build **succeeds**: log `"Build verification: PASSED ✓"`
- If the build **fails**: analyze the error output to determine which fix likely caused the failure.
  1. Parse error messages for file names and line numbers.
  2. Cross-reference against the fixes applied in Step 5.3 to identify the likely causal fix.
  3. Display:

> **Build failed after fix application.**
> Build command: `{command}`
> Error: {error summary}
> Likely causal fix: F-{NNN} — {title}
>
> Options:
> - **revert** — Revert the likely causal fix (`git checkout -- {file}`) and re-run build
> - **stop** — Stop and proceed to Phase 5b with current state

  4. Wait for user input. If "revert": revert the specified fix, remove its commit if git is available (`git revert HEAD --no-edit` if it was the last commit, otherwise note it for manual cleanup), and re-run the build. If "stop": proceed to Phase 5b.

### Step 5.5: Run Test Suite

After the build step (or if no build system was detected), attempt to run the project's test suite.

**Detect test command:**

Check for test tooling in this priority order:

1. `package.json` with `scripts.test` → `npm test` (or `yarn test` / `pnpm test`)
2. `Cargo.toml` → `cargo test`
3. `pyproject.toml` or `pytest.ini` or `setup.cfg` → `pytest` or `python -m pytest`
4. `go.mod` → `go test ./...`
5. `Makefile` with `test` target → `make test`
6. `build.gradle` / `pom.xml` → `gradle test` / `mvn test`
7. No test system detected → skip this step with a warning

If no test system is detected, log:

> **WARNING: No test suite detected.** FORTRESS recommends adding automated tests to verify security fixes do not introduce regressions. Consider adding tests for the following fixed areas:
> {list of files/functions that were modified by fixes}

**Run the tests:**

Execute the detected test command via the Bash tool with a timeout of 300 seconds.

- If all tests **pass**: log `"Test suite: PASSED ✓ ({N} tests)"`
- If any tests **fail**: analyze the output to identify which tests failed and which fix likely caused the failure.
  1. List the failing test names/files.
  2. Cross-reference the test failures against modified files and fixes applied.
  3. Display:

> **Test failures detected after fix application.**
> Test command: `{command}`
> Failing tests:
> - `{test_name}` in `{test_file}` — {failure summary}
>
> Likely causal fix: F-{NNN} — {title} (modified `{file}` which is tested by `{test_file}`)
>
> Options:
> - **revert** — Revert the likely causal fix and re-run tests
> - **continue** — Proceed to Phase 5b with failing tests noted
> - **stop** — Stop and proceed to Phase 5b

  4. Wait for user input. Process accordingly.

### Step 5.6: Risk Acceptance Documentation

For each finding that was REJECTED at Gate 2, create a formal risk acceptance record. These records provide auditable documentation that the risk was acknowledged and consciously accepted.

If `.fortress/risk-acceptances.md` does not exist, create it with the following header:

```markdown
# Risk Acceptances

This file documents findings that were identified by FORTRESS audits but consciously rejected by the project maintainer. Each entry represents an accepted risk with documented rationale. Risk acceptances should be reviewed periodically to determine if circumstances have changed.

---
```

For each REJECTED finding, append an entry in the following format:

```markdown
### F-{NNN}: {title}

| Field | Value |
|---|---|
| **Finding ID** | F-{NNN} |
| **Title** | {title} |
| **Severity** | {severity} |
| **CWE** | {cwe_id} ({cwe_name}) |
| **CVSS** | {cvss_score} (estimated) |
| **File** | `{file}:{line}` |
| **Rejected by** | {user — use "Project Maintainer" if not specified} |
| **Date** | {YYYY-MM-DD} |
| **Rationale** | {rationale provided by user at Gate 2} |
| **Review date** | {date 90 days from now for CRITICAL/HIGH, 180 days for MEDIUM/LOW} |

---
```

After writing all risk acceptance entries, log:

> **Risk acceptances documented: {N} rejected findings recorded in `.fortress/risk-acceptances.md`.**
> Review dates set: {earliest review date} through {latest review date}.

If no findings were rejected, skip this step.

**Phase 5 completion log:**

> **Phase 5 complete.**
> - Fixes applied: {N}
> - Fixes skipped (syntax failure): {N}
> - Fixes skipped (target changed): {N}
> - Build: {PASSED / FAILED / SKIPPED}
> - Tests: {PASSED / FAILED ({N} failures) / SKIPPED / NO TEST SUITE}
> - Risk acceptances: {N}
>
> Proceeding to Phase 5b: INTEGRATION VERIFICATION.

## Phase 5b: INTEGRATION VERIFICATION

**Goal:** Verify that the combined set of applied fixes are coherent and do not introduce integration issues. Individual fixes may pass syntax verification in isolation but conflict when combined — broken imports, type mismatches between files, logical contradictions, inconsistent error handling patterns, or function signatures that no longer match their call sites. Phase 5b catches these cross-file and cross-fix interaction problems.

Phase 5b always runs after Phase 5, even if only one fix was applied. The only exception is if Phase 5 was skipped entirely (all findings rejected/deferred).

### Step 5b.1: Dispatch Integration Verification Agent

Use the Agent tool to dispatch a dedicated integration verification agent. This agent reads the current state of all modified files holistically and checks for interaction problems between fixes.

**Collect the modification manifest:**

Before dispatching, build the modification manifest — a list of every file modified during Phase 5, with the specific changes made:

```
MODIFICATION MANIFEST:
{for each modified file:}
- File: {file_path}
  Fixes applied: {list of finding IDs applied to this file}
  Changes: {brief description of what changed}
{end for}
```

**Dispatch the agent with the following prompt:**

```
You are a FORTRESS Integration Verification Agent. Your job is to verify that a set of security fixes applied to a codebase are coherent when combined. Individual fixes passed syntax verification, but you must check for CROSS-FILE and CROSS-FIX interaction issues.

MODIFICATION MANIFEST:
{insert modification manifest here}

INSTRUCTIONS:
1. Read the CURRENT state of every file in the modification manifest using the Read tool. Read the full file, not just the changed sections — you need surrounding context to verify integration.

2. For each modified file, also read any files that IMPORT from it or that it IMPORTS from. Follow the import graph one level out in each direction. Use Grep to search for import statements referencing modified files if needed.

3. Check for the following integration issues:

   a. BROKEN IMPORTS: Does any file import a symbol (function, class, type, constant) from a modified file that no longer exists or has been renamed? Check both named imports and default imports.

   b. TYPE MISMATCHES: Do function signatures in modified files still match their call sites in other files? Check parameter types, return types, and generic type arguments. For TypeScript, verify interface/type compatibility.

   c. LOGICAL CONTRADICTIONS: Do any two fixes contradict each other? For example: one fix adds input validation that rejects a format, while another fix in a different file generates output in that same format. Or one fix changes an error to throw, while another fix in a caller expects a return value.

   d. INCONSISTENT ERROR HANDLING: If one fix changed error handling patterns (e.g., switching from error codes to exceptions, or changing error response format), verify all related code paths use the same pattern.

   e. FUNCTION SIGNATURE MISMATCHES: If a fix changed a function's parameters, return type, or behavior contract, verify all call sites have been updated accordingly. Check for: added required parameters not passed by callers, removed parameters still passed by callers, changed return types not handled by callers.

   f. MISSING RELATED CHANGES: If a fix modified a shared utility, configuration, or type definition, verify that all consumers of that shared resource still work correctly with the new version.

   g. IMPORT GRAPH CONSISTENCY: Verify no circular dependencies were introduced. Verify no dead imports were created (importing from a file that no longer exports what is expected).

4. Return your findings as a JSON object with this exact structure:

{
  "status": "PASS" | "FAIL",
  "issues": [
    {
      "type": "broken_import" | "type_mismatch" | "logical_contradiction" | "inconsistent_error_handling" | "signature_mismatch" | "missing_related_change" | "import_graph",
      "severity": "HIGH" | "MEDIUM" | "LOW",
      "file": "path/to/affected/file",
      "related_file": "path/to/other/file",
      "description": "Clear description of the integration issue",
      "suggested_fix": "Specific code change to resolve the issue"
    }
  ],
  "files_checked": ["list", "of", "all", "files", "read"],
  "summary": "One paragraph summary of integration verification results"
}

If there are zero issues, return status "PASS" with an empty issues array.
If there are any issues, return status "FAIL" regardless of their severity.

Be thorough. Missing an integration issue here means it ships to production.
```

Parse the agent's JSON response. If the response is not valid JSON, re-dispatch with a smaller scope (only the files that were directly modified, without the import graph expansion).

Log the result:

> **Integration verification: {PASS/FAIL}**
> Files checked: {N}
> Issues found: {N} ({breakdown by type})

### Step 5b.2: Handle Integration Issues

If the integration verification returned `FAIL`, process each issue.

**For each integration issue:**

1. Display the issue to the user:

> **Integration Issue {I} of {N}:** {type}
> - **Severity:** {severity}
> - **File:** `{file}` (related: `{related_file}`)
> - **Description:** {description}
> - **Suggested fix:** {suggested_fix}
>
> **Apply this fix? (yes / skip / stop)**

2. If "yes":
   - Read the current state of the affected file(s) using the Read tool.
   - Apply the fix using the Edit tool with exact `old_string`/`new_string`.
   - Run syntax verification (same as Step 5.3c).
   - If verification passes and `git_available = true`: commit with message `"fortress: integration fix — {type} in {file}"`.
   - If verification fails: revert (if git available), log the failure, and ask skip/stop.

3. If "skip": log `"Integration issue {I} skipped by user."` and continue to the next issue.

4. If "stop": log `"Integration verification halted by user."` and proceed to Step 5b.3 with current state.

**After all issues are processed (or if any were applied), re-run the integration verification agent:**

Re-dispatch the same agent with the updated modification manifest (now including integration fix files). If the re-run returns `PASS`, proceed. If it returns `FAIL` with new issues, repeat this step. Cap at 3 re-verification cycles to prevent infinite loops. If issues persist after 3 cycles, log a warning and proceed:

> **WARNING: Integration issues persist after 3 resolution cycles. Remaining issues:**
> {list remaining issues}
>
> Proceeding to final build and test. Manual review recommended for the above issues.

### Step 5b.3: Final Build and Test

After all integration issues are resolved (or after the resolution loop is exhausted), run the full build and test suite one final time to confirm the complete set of changes — original fixes plus integration fixes — work together.

**Build:**

Run the same build command detected in Step 5.4. If the build fails:

> **Final build failed.** This indicates an unresolved integration issue.
> Error: {error summary}
>
> Options:
> - **investigate** — Re-run integration verification focused on the error
> - **revert-last** — Revert the most recent integration fix and re-build
> - **proceed** — Proceed to Gate 3 with the build failure noted

Wait for user input and process accordingly.

**Tests:**

Run the same test command detected in Step 5.5. If tests fail, report as in Step 5.5 but note these are post-integration failures.

Log:

> **Final verification: Build {PASSED/FAILED}, Tests {PASSED/FAILED ({N} failures)/SKIPPED/NO TEST SUITE}.**

### Step 5b.4: Present Final Diff (APPROVAL GATE 3)

This is the third and final approval gate. Present the complete set of changes made during Phases 5 and 5b for user review.

**If `git_available = true`:**

1. Run `git diff --stat` from the pre-audit checkpoint to HEAD to show a summary of files changed:
   ```
   git diff --stat {checkpoint_commit}..HEAD
   ```
   Display the output.

2. Run `git diff` from the checkpoint to HEAD to show the full detailed diff:
   ```
   git diff {checkpoint_commit}..HEAD
   ```
   Display the output. If the diff is very large (more than 200 lines), summarize the changes per file and offer to show the full diff for specific files on request.

3. Run `git log --oneline` from the checkpoint to HEAD to show the commit history:
   ```
   git log --oneline {checkpoint_commit}..HEAD
   ```
   Display the output.

**If `git_available = false`:**

List all files that were modified during Phase 5 and Phase 5b, with a summary of the changes made to each file. Note that a detailed diff is unavailable without git.

**Present the gate prompt:**

> **APPROVAL GATE 3: Review the combined changes above.**
>
> Summary:
> - Files modified: {N}
> - Fixes applied: {N} (of {total approved} approved)
> - Integration fixes: {N}
> - Build: {PASSED/FAILED/SKIPPED}
> - Tests: {PASSED/FAILED/SKIPPED/NO TEST SUITE}
>
> **Ready to finalize? (yes / revert all / revert specific)**

Wait for user input.

**If "yes":**
- Log: `"Gate 3 approved. All changes accepted. Proceeding to Phase 6 (Debrief)."`
- Proceed to Phase 6.

**If "revert all":**
- If `git_available = true`: run `git reset --hard {checkpoint_commit}` to revert all changes back to the pre-audit checkpoint.
- Log: `"All Phase 5/5b changes reverted to pre-audit checkpoint."`
- Proceed to Phase 6 (findings are still recorded in artifacts from Phase 4; the code is just unchanged).

**If "revert specific":**
- Display the list of commits made during Phase 5/5b:
  ```
  git log --oneline {checkpoint_commit}..HEAD
  ```
- Ask: `"Enter the finding IDs to revert (comma-separated), or commit hashes:"`
- For each specified revert, use `git revert {commit} --no-edit` to create a revert commit (preserving history).
- After all specified reverts, re-run the build and tests to verify the remaining changes are still valid.
- If build/tests pass: proceed to Phase 6.
- If build/tests fail: report the failures and ask the user for further instructions (revert more, revert all, or proceed anyway).

**Phase 5b completion log:**

> **Phase 5b complete.**
> - Integration verification: {PASS/FAIL}
> - Integration fixes applied: {N}
> - Final build: {PASSED/FAILED/SKIPPED}
> - Final tests: {PASSED/FAILED/SKIPPED/NO TEST SUITE}
> - Gate 3 decision: {approved / reverted all / reverted specific ({list})}
>
> Proceeding to Phase 6: DEBRIEF.

## Phase 6: DEBRIEF

**Goal:** Feed everything learned during this audit back into the `.fortress/` directory so the next audit is smarter, more focused, and more efficient. Phase 6 implements the antifragile learning loop — the core principle that each audit makes the next one better. It also generates the hero metrics summary that tells the user exactly what was accomplished.

Phase 6 always runs, regardless of how many fixes were applied or reverted. Even if all findings were rejected, the audit still produced valuable coverage data, pattern observations, and false positive information that must be recorded.

### Step 6.1: Update last-audit.md

Write (or overwrite) `.fortress/last-audit.md` with metadata from the current audit. This file is the primary input for verify mode, diff mode, and anti-confirmation-bias calculations in future audits.

If the file does not exist, create it. If it exists, overwrite it completely (this file represents only the most recent audit).

```markdown
# Last Audit

| Field | Value |
|---|---|
| **Date** | {YYYY-MM-DD} |
| **Mode** | {full / quick / focused ({domain}) / verify / diff} |
| **Squads deployed** | {N} ({list of squad names}) |
| **Files analyzed** | {N} of {total files in project} |
| **Personas activated** | {N} |
| **Findings by severity** | CRITICAL: {N}, HIGH: {N}, MEDIUM: {N}, LOW: {N}, ENHANCEMENT: {N} |
| **Fixed** | {N} |
| **Deferred** | {N} |
| **Rejected** | {N} |
| **False positive rate** | {N}% ({false positives removed in Phase 3} / {total raw findings from Phase 2}) |
| **CWE Top 25 coverage** | {N}/25 ({percentage}%) |
| **OWASP Top 10 coverage** | {N}/10 categories tested |
| **Build verification** | {PASSED / FAILED / SKIPPED} |
| **Test verification** | {PASSED / FAILED / SKIPPED / NO TEST SUITE} |
| **Audit duration** | {approximate elapsed time} |
| **Artifacts generated** | {list of artifact file paths} |
| **Git branch** | {branch name or "no git"} |
| **Checkpoint commit** | {commit hash or "N/A"} |
```

Log:

> **`.fortress/last-audit.md` updated with audit metadata.**

### Step 6.2: Update known-patterns.md

Update `.fortress/known-patterns.md` with project-specific patterns identified during this audit. Known patterns help future audits distinguish true vulnerabilities from known architecture decisions, framework behaviors, and confirmed false positives.

If the file does not exist, create it with the following header:

```markdown
# Known Patterns

This file records project-specific patterns identified across FORTRESS audits. Patterns are advisory only — they inform validation but never suppress findings. Each pattern has a confidence level and decays if not confirmed in subsequent audits.

---
```

**For each validated true-positive finding that represents a project-specific pattern** (not a one-off bug, but a recurring pattern like "all API routes in this project use framework X's built-in CSRF protection" or "this project stores sessions in Redis with automatic expiration"), add or update an entry:

**New pattern (not already in known-patterns.md):**

```markdown
### PATTERN-{NNN}
- **Type:** {false-positive | architecture-decision | framework-behavior | recurring-weakness}
- **Scope:** {file glob, e.g., `app/api/**`, `src/auth/*`, `**/*.py`}
- **Description:** {concise description of the pattern}
- **First recorded:** {YYYY-MM-DD}
- **Last confirmed:** {YYYY-MM-DD}
- **Audit count:** 1
- **Confidence:** {low | medium | high}
- **Decay status:** active
```

Assign pattern numbers sequentially. If existing patterns exist, read the file to find the highest pattern number and increment.

**Existing pattern confirmed this audit:**

If a finding from this audit confirms an existing pattern (the same type of finding in the same scope):
- Increment `Audit count` by 1
- Update `Last confirmed` to today's date
- If `Audit count` reaches 3 and `Confidence` is not already `high`, upgrade `Confidence` to `high`
- If `Decay status` was `review`, set it back to `active`

**Existing pattern NOT referenced this audit:**

Do NOT modify patterns that were not referenced in this audit. Pattern decay is handled by the decay check below, not by the absence of a single audit.

**Decay check:**

Read all existing patterns. For any pattern where:
- `Decay status` is `active`
- The pattern was NOT confirmed in this audit
- The number of audits since `Last confirmed` is 3 or more (compare current audit count from `last-audit.md` history if available, or compare dates)

Set `Decay status` to `review` and log:

> **Pattern PATTERN-{NNN} demoted to "review" — not confirmed in 3+ audits.**

Patterns in `review` status are still loaded during Phase 0 ingestion but flagged as potentially stale. They are not automatically removed — that requires human decision.

Log:

> **`.fortress/known-patterns.md` updated: {N} new patterns, {M} patterns confirmed, {K} patterns demoted to review.**

### Step 6.3: Update deferred.md

Update `.fortress/deferred.md` with all findings deferred at Gate 2. This file tracks unresolved security findings with deadlines and responsible parties.

If the file does not exist, create it with the following header:

```markdown
# Deferred Findings

This file tracks security findings that were identified but deferred for later remediation. Each entry includes a milestone date and responsible party. Overdue items are flagged at the start of each FORTRESS audit.

---
```

**For each DEFERRED finding from this audit,** append an entry:

```markdown
### F-{NNN}: {title}

| Field | Value |
|---|---|
| **Finding ID** | F-{NNN} |
| **Title** | {title} |
| **Severity** | {severity} |
| **CWE** | {cwe_id} ({cwe_name}) |
| **File** | `{file}:{line}` |
| **Deferred date** | {YYYY-MM-DD} |
| **Milestone date** | {date provided by user at Gate 2} |
| **Responsible party** | {party provided by user at Gate 2, or "TBD"} |
| **Status** | open |

---
```

**For existing deferred items from prior audits** that were addressed (fixed) in this audit:
- Update their `Status` from `open` to `resolved`
- Add: `**Resolved date:** {YYYY-MM-DD}`
- Add: `**Resolved by:** FORTRESS audit {date}`

**For existing deferred items that are overdue** (milestone date has passed and status is still `open`):
- Add: `**OVERDUE:** Yes — {N} days past milestone`
- These were already flagged in Phase 0 (Step 0.2) but the status update is recorded here.

Log:

> **`.fortress/deferred.md` updated: {N} new deferrals, {M} resolved, {K} overdue.**

### Step 6.4: Update coverage-map.md

Update `.fortress/coverage-map.md` with a comprehensive map of what was and was not analyzed during this audit. This is the primary input for the anti-confirmation-bias mechanism in future audits.

If the file does not exist, create it with the following header:

```markdown
# Coverage Map

This file tracks which files, CWE categories, and OWASP categories have been analyzed across FORTRESS audits. Under-covered areas receive increased attention in future audits.

---
```

Write (or overwrite) the coverage data for the current audit:

```markdown
## Audit: {YYYY-MM-DD}

### Files Analyzed
{list of all files that were read by at least one squad during Phase 2, one per line with a checkmark}

### Files NOT Analyzed
{list of all project files that were NOT read by any squad, one per line}
Reason: {typical reasons — file type not in scope, file excluded by squad assignment, codebase too large for full coverage}

### CWE Categories Tested
{list of CWE categories that at least one finding (true or false positive) was evaluated against}

### CWE Categories NOT Tested
{list of CWE Top 25 categories that were not evaluated by any squad — these are coverage gaps}

### OWASP Categories Tested
{list of OWASP Top 10 Web 2025 categories tested}

### OWASP Categories NOT Tested
{list of OWASP categories not tested}

### Domains with Zero Findings
{list of security domains where squads were deployed but found zero issues — these should get increased attention if they remain zero across multiple audits}

### Under-Covered Areas from Prior Audits
{if prior coverage-map.md existed, list areas that were flagged as under-covered and note whether they received increased attention in this audit}
```

Log:

> **`.fortress/coverage-map.md` updated. Coverage: {N}/{total} files analyzed ({percentage}%). CWE Top 25: {N}/25 tested. OWASP: {N}/10 tested.**

### Step 6.5: Anti-Confirmation-Bias Updates

This step implements the mechanisms that prevent FORTRESS from narrowing its focus over time. Without active bias correction, the audit tends to look harder at areas where it found issues before and neglect areas where it found nothing — exactly the opposite of what good security practice demands.

**Record what was NOT found:**

Review all squads deployed and their domain coverage. For each security domain that was tested but produced zero validated findings, record it in the coverage map (Step 6.4) under "Domains with Zero Findings."

**Flag persistent zero-finding areas:**

If `.fortress/coverage-map.md` existed before this audit (indicating prior audit data is available), check for domains that have had zero findings across 3 or more consecutive audits. For each such domain:

> **ATTENTION: {domain} has had zero findings across {N} consecutive audits.** This may indicate:
> - The code in this domain is genuinely secure (good!)
> - FORTRESS personas are not effectively testing this domain (needs attention)
> - The domain has not changed and prior testing was sufficient
>
> **Recommendation:** Allocate increased squad capacity to {domain} in the next audit.

**Reserve capacity for under-covered domains:**

Write a recommendation to `.fortress/coverage-map.md` under "Under-Covered Areas":

```markdown
### Recommended Focus for Next Audit
- Reserve ~20% of squad capacity for under-covered domains
- Priority areas: {list domains with zero findings across 3+ audits}
- Files never analyzed: {count} — consider expanding scope
- CWE categories never tested: {list} — consider targeted squads
```

**Demote unconfirmed patterns:**

This was already handled in Step 6.2 (decay check), but log a summary here:

> **Anti-confirmation-bias check:**
> - Domains with zero findings this audit: {N}
> - Domains with zero findings across 3+ audits: {N} (flagged for increased attention)
> - Patterns demoted to "review" (unconfirmed 3+ audits): {N}
> - Recommended focus areas for next audit: {list}

### Step 6.6: Display Hero Metrics and Summary

This is the final output of a FORTRESS audit. Display a formatted summary block in the conversation that gives the user a clear picture of what was accomplished, what the security posture looks like, and where to find all artifacts.

```
╔══════════════════════════════════════════════════════════════════╗
║                    FORTRESS AUDIT COMPLETE                      ║
╠══════════════════════════════════════════════════════════════════╣
║                                                                  ║
║  Date:            {YYYY-MM-DD}                                   ║
║  Mode:            {full / quick / focused / verify / diff}       ║
║  Duration:        {approximate elapsed time}                     ║
║                                                                  ║
║  ── SQUADS ──────────────────────────────────────────────────    ║
║  Squads deployed:     {N}                                        ║
║  Personas activated:  {N}                                        ║
║  Files analyzed:      {N} / {total} ({percentage}%)              ║
║                                                                  ║
║  ── STANDARDS COVERAGE ──────────────────────────────────────    ║
║  CWE Top 25:          {N}/25 ({percentage}%)                     ║
║  OWASP Top 10:        {N}/10                                     ║
║  NIST 800-53:         {N} controls mapped                        ║
║                                                                  ║
║  ── FINDINGS ────────────────────────────────────────────────    ║
║  CRITICAL:  {N}    HIGH:  {N}    MEDIUM:  {N}    LOW:  {N}      ║
║  Enhancement: {N}  Positive: {N}                                 ║
║                                                                  ║
║  ── DISPOSITION ─────────────────────────────────────────────    ║
║  Fixed:     {N}                                                  ║
║  Deferred:  {N}                                                  ║
║  Rejected:  {N}                                                  ║
║                                                                  ║
║  ── QUALITY ─────────────────────────────────────────────────    ║
║  False positive rate:  {N}%                                      ║
║  Build verification:   {PASSED / FAILED / SKIPPED}               ║
║  Test verification:    {PASSED / FAILED / SKIPPED / NO SUITE}    ║
║  Integration check:    {PASSED / FAILED / SKIPPED}               ║
║                                                                  ║
║  ── TREND ───────────────────────────────────────────────────    ║
║  {if prior audit exists:}                                        ║
║  vs. last audit ({prior date}):                                  ║
║    Findings: {N → N} ({improving/stable/declining})              ║
║    Coverage: {N% → N%}                                           ║
║    False positive rate: {N% → N%}                                ║
║  {else:}                                                         ║
║  First audit — no trend data available.                          ║
║  {end if}                                                        ║
║                                                                  ║
║  ── ARTIFACTS ───────────────────────────────────────────────    ║
║  .fortress/reports/{date}.md          Detailed report            ║
║  .fortress/reports/{date}.sarif       SARIF (GitHub integration) ║
║  .fortress/reports/{date}-sbom.json   CycloneDX SBOM            ║
║  .fortress/reports/{date}-compliance.md  Compliance posture      ║
║  .fortress/reports/{date}-poam.md     POA&M                      ║
║  .fortress/reports/{date}-security-page.html  Public page        ║
║  .fortress/reports/{date}-delta.md    Delta report               ║
║  .fortress/reports/{date}-snapshot.md Posture snapshot           ║
║                                                                  ║
║  .fortress/last-audit.md             Audit metadata              ║
║  .fortress/known-patterns.md         Pattern library             ║
║  .fortress/deferred.md               Deferred findings           ║
║  .fortress/risk-acceptances.md       Risk acceptances            ║
║  .fortress/coverage-map.md           Coverage map                ║
║                                                                  ║
╚══════════════════════════════════════════════════════════════════╝
```

**Trend calculation:**

If `.fortress/last-audit.md` existed before this audit (saved in Phase 0 ingestion), compare:
- Total findings count: if current < prior, trend is "improving". If equal, "stable". If current > prior, "declining" (more issues found may indicate new code introduced vulnerabilities or improved detection found previously missed issues — note both possibilities).
- Coverage percentage: if current > prior, "improving". If equal, "stable". If current < prior, "declining".
- False positive rate: if current < prior, "improving". If equal, "stable". If current > prior, "declining".

**Final message:**

After the hero metrics block, display:

> **Audit complete.** All artifacts saved to `.fortress/reports/`. Run `/fortress diff` anytime to check progress against this baseline. Run `/fortress verify` after making changes to confirm fixes hold.

If `git_available = true` and a stash was created in Step 5.1:

> **Note:** Pre-audit uncommitted changes were stashed. Run `git stash pop` to restore them when ready.

**Phase 6 completion log (internal):**

> **Phase 6 complete. FORTRESS audit finished. All `.fortress/` artifacts updated. Antifragile learning loop closed.**

---

## Section 11: Persona Taxonomy

This section contains the complete FORTRESS persona library: 448 personas across 23 squads plus the Wildcard squad. During Phase 1 (Squad Assembly), the orchestrator extracts the relevant squad entries from this section to build each squad agent's prompt. Each persona defines a name, 2-3 attack techniques, and success criteria.

---

### Always Active (5 squads — 92 personas)

These squads run on EVERY audit regardless of stack detection.

### Squad 1: Infrastructure & Supply Chain (22 personas)
*Activation: Always active*

1. **Dependency auditor** — Scan manifests and lock files for known CVEs, outdated packages, unmaintained libraries. Success: find dependency with active CVE or no updates in 2+ years.
2. **SBOM validator** — Verify dependency tree completeness, detect phantom dependencies not in lock file. Success: find unlocked or untracked dependency.
3. **Env var leaker** — Search for secrets in source, config files, CI definitions, and shell scripts. Success: find hardcoded secret, API key, or credential in committed code.
4. **Build pipeline saboteur** — Audit CI/CD configs for injection points, unsafe variable expansion, missing pin versions. Success: find injectable CI step or unpinned action.
5. **DNS/subdomain scout** — Check for hardcoded domains, subdomain references, dangling DNS entries in code. Success: find takeover-vulnerable subdomain reference.
6. **Container image auditor** — Check Dockerfiles for root user, unverified base images, exposed ports, multi-stage leaks. Success: find container running as root or using unverified image.
7. **IaC misconfiguration hunter** — Audit Terraform, CloudFormation, Pulumi for overly permissive rules, public resources. Success: find publicly exposed resource or wildcard IAM.
8. **Secret rotation auditor** — Check for long-lived secrets, missing rotation policies, static API keys. Success: find secret with no rotation mechanism.
9. **SAST integration point** — Verify static analysis is in CI pipeline, check for bypasses or missing coverage. Success: find missing SAST step or bypassable check.
10. **Package provenance checker** — Verify package sources, check for unsigned packages, missing integrity hashes. Success: find package without integrity verification.
11. **License compliance scanner** — Detect copyleft or incompatible licenses in dependency tree. Success: find GPL dependency in MIT/proprietary project.
12. **Dot-env file auditor** — Check for .env files committed to repo, missing .gitignore entries, env template leaks. Success: find committed .env or secret in .env.example.
13. **Registry confusion detector** — Check for dependency confusion between public and private registries, scope hijacking. Success: find package name vulnerable to registry confusion.
14. **Rules file backdoor detector** — Audit .claude/, .cursor/, .github/, and similar AI/IDE rule files for injected instructions. Success: find malicious or overly permissive instruction in rules file.
15. **Deployment artifact auditor** — Check build outputs for source maps, debug symbols, test fixtures in production bundles. Success: find source map or debug artifact in production build config.
16. **Lockfile poisoner** — Verify lockfile integrity, check for tampered entries, mismatched checksums. Success: find lockfile entry inconsistent with manifest.
17. **Build cache saboteur** — Check for cache poisoning vectors in build systems, unsafe cache keys. Success: find predictable or injectable cache key.
18. **Lifecycle script auditor** — Audit package.json scripts, Makefile targets, post-install hooks for malicious commands. Success: find dangerous command in lifecycle hook.
19. **Maintainer trust analyst** — Check dependency maintainer count, recent ownership transfers, bus factor. Success: find critical dependency with single maintainer or recent transfer.
20. **Artifact provenance verifier** — Verify build artifacts are reproducible, check for attestation gaps. Success: find unattested or unreproducible build artifact.
21. **Typosquatting hunter** — Check dependencies for names similar to popular packages, detect potential typosquats. Success: find dependency with suspiciously similar name to popular package.
22. **Git submodule auditor** — Check submodule references for pinned commits, trust boundaries, stale references. Success: find unpinned submodule or reference to untrusted repository.

### Squad 2: Edge Cases & Input Validation (21 personas)
*Activation: Always active*

1. **Null specialist** — Test null/undefined/None handling at every input boundary, optional fields, nullable returns. Success: find unhandled null that causes crash or bypass.
2. **Type coercion exploiter** — Test implicit type conversions, loose equality, string-to-number coercion in comparisons. Success: find type coercion that bypasses validation or auth check.
3. **Boundary value analyst** — Test min/max values, off-by-one, empty strings, zero-length arrays, MAX_INT. Success: find boundary value that causes incorrect behavior.
4. **Unicode/encoding attacker** — Test UTF-8 edge cases, BOM injection, RTL overrides, homoglyphs, encoding mismatches. Success: find encoding mismatch that bypasses filter or causes display corruption.
5. **Concurrency racer** — Identify TOCTOU races, unprotected shared state, missing locks in parallel paths. Success: find race condition that allows double-spend, double-action, or data corruption.
6. **Format string exploiter** — Test for user input in format strings, template literals, logging formatters. Success: find user-controlled format string that leaks data or causes crash.
7. **Resource exhaustion attacker** — Test for missing limits on file uploads, request sizes, connection pools, memory allocation. Success: find input that causes unbounded resource consumption.
8. **ReDoS hunter** — Identify regex patterns vulnerable to catastrophic backtracking with crafted input. Success: find regex where crafted input causes exponential execution time.
9. **Integer overflow specialist** — Test arithmetic operations for overflow/underflow, unchecked multiplication, size calculations. Success: find integer overflow that corrupts data or bypasses checks.
10. **Negative number tester** — Test negative values where only positive expected: quantities, indices, offsets, prices. Success: find negative input that causes logic error or financial exploit.
11. **Deep nesting bomb** — Test deeply nested JSON/XML/objects that cause stack overflow or excessive parsing time. Success: find input depth that crashes parser or exhausts resources.
12. **Large payload tester** — Test oversized inputs, huge arrays, massive strings at every endpoint. Success: find missing size limit that causes memory exhaustion or timeout.
13. **Character set tester** — Test special characters, control characters, null bytes, newlines in text fields. Success: find special character that bypasses validation or causes injection.
14. **Locale/timezone exploiter** — Test date/time handling across timezones, DST transitions, locale-specific formatting. Success: find timezone or locale assumption that causes incorrect behavior.
15. **State machine violator** — Test out-of-order operations, skipped steps, repeated actions in multi-step flows. Success: find state transition that can be skipped or replayed for bypass.
16. **Zip Slip exploiter** — Test archive extraction for path traversal via crafted filenames. Success: find archive handling that allows writing outside target directory.
17. **Symlink race attacker** — Test file operations for symlink following, TOCTOU on filesystem paths. Success: find file operation that follows symlinks to read/write unintended paths.
18. **Floating point precision exploiter** — Test financial or comparison logic for IEEE 754 precision errors. Success: find floating point comparison that produces incorrect result.
19. **Null byte truncation injector** — Test string handling for null byte truncation in file paths, queries, filters. Success: find null byte that truncates validation while passing to backend.
20. **Environment variable injector** — Test for injectable environment variables that alter application behavior. Success: find env var that can be set by attacker to change app behavior.
21. **Canonicalization specialist** — Test for path canonicalization bypasses, URL normalization issues, case sensitivity mismatches. Success: find canonicalization difference that bypasses access control.

### Squad 3: Future-Proofing & Quantum Readiness (16 personas)
*Activation: Always active*

1. **Deprecation tracker** — Identify deprecated APIs, libraries, language features, and framework patterns in use. Success: find deprecated feature with known removal timeline.
2. **Scaling analyst** — Identify hardcoded limits, single-point bottlenecks, O(n^2) algorithms on growing data. Success: find scaling bottleneck that breaks at predictable growth point.
3. **Dependency health monitor** — Check dependency age, maintenance status, known vulnerability response time. Success: find dependency that is unmaintained, archived, or slow to patch.
4. **API versioning strategist** — Audit API contracts for breaking change risks, missing version headers, unversioned endpoints. Success: find unversioned API endpoint or breaking change without migration path.
5. **Crypto-agility auditor** — Check if cryptographic algorithms are hardcoded vs configurable, assess migration difficulty. Success: find hardcoded algorithm that would require code changes to replace.
6. **Post-quantum readiness checker** — Identify RSA/ECC/DH usage that will be vulnerable to quantum computers. Success: find quantum-vulnerable cryptographic operation protecting long-lived data.
7. **CNSA 2.0 compliance checker** — Verify alignment with NSA CNSA 2.0 algorithm requirements and timelines. Success: find non-CNSA-2.0-compliant algorithm in government-adjacent context.
8. **Certificate pinning detector** — Identify certificate pinning that will break on rotation or CA changes. Success: find hardcoded certificate pin that will cause outage on rotation.
9. **Long-lived secret identifier** — Find secrets that persist beyond reasonable lifetime without rotation mechanism. Success: find secret with no expiry and no rotation capability.
10. **FIPS 140-3 compliance checker** — Verify cryptographic modules meet FIPS 140-3 requirements where applicable. Success: find non-FIPS-compliant crypto in compliance-required context.
11. **NIST deprecation timeline enforcer** — Check algorithms against NIST deprecation schedules and sunset dates. Success: find algorithm past or approaching NIST deprecation deadline.
12. **Hybrid crypto auditor** — Assess readiness for hybrid classical+post-quantum cryptographic schemes. Success: find system unable to support hybrid key exchange without major refactor.
13. **Key encapsulation readiness checker** — Evaluate readiness for ML-KEM/CRYSTALS-Kyber key encapsulation migration. Success: find key exchange mechanism with no path to post-quantum migration.
14. **Algorithm identifier hardcoding detector** — Find hardcoded algorithm OIDs or names that prevent agile migration. Success: find hardcoded algorithm identifier that blocks algorithm rotation.
15. **IPv6 transition auditor** — Check for IPv4-only assumptions in network code, hardcoded IPv4 addresses, missing dual-stack. Success: find IPv4-only code path that will fail on IPv6 networks.
16. **AI regulation forecaster** — Identify AI/ML patterns that may conflict with emerging regulations (EU AI Act, etc.). Success: find AI usage pattern likely to require compliance changes within 2 years.

### Squad 4: Logging & Audit Trail (16 personas)
*Activation: Always active*

1. **Audit log completeness checker** — Verify all security-relevant actions are logged: auth, access, changes, admin ops. Success: find security-critical operation with no audit log entry.
2. **Log injection hunter** — Test for user input written directly to logs without sanitization. Success: find injectable log entry that could forge log records or execute log viewer exploits.
3. **Log tampering detector** — Check for append-only log enforcement, integrity verification, tamper detection. Success: find log storage that allows modification or deletion of entries.
4. **Centralized logging verifier** — Check that all components log to centralized system, no orphaned log streams. Success: find component that logs locally only with no centralized forwarding.
5. **Log rotation auditor** — Verify log rotation policies, retention limits, disk space protections. Success: find unbounded log growth that could fill disk.
6. **Debug mode detector** — Find debug flags, verbose logging, development mode enabled in production configs. Success: find debug mode or verbose logging enabled in production configuration.
7. **Repudiation analyst** — Identify actions that cannot be traced to a specific actor or timestamp. Success: find critical action that can be performed without attribution.
8. **Alert configuration checker** — Verify security alerts exist for critical events: failed auth, privilege changes, anomalies. Success: find critical security event with no alerting configured.
9. **Log correlation gap analyst** — Check for missing correlation IDs, request tracing gaps across service boundaries. Success: find cross-service flow with no correlation mechanism.
10. **Log overflow/truncation exploiter** — Test for log entries that can be truncated or overflow buffers to hide attack evidence. Success: find log mechanism that drops entries under high volume.
11. **Compliance-specific logging auditor** — Verify logging meets specific regulatory requirements (PCI, HIPAA, SOX). Success: find compliance-required log event that is missing.
12. **Audit trail integrity verifier** — Check for cryptographic log integrity, hash chains, tamper-evident logging. Success: find audit trail with no integrity verification mechanism.
13. **Security event coverage mapper** — Map security events to logging coverage, identify blind spots. Success: find category of security event with zero logging coverage.
14. **Log desynchronization attacker** — Test for time synchronization issues that could defeat log correlation. Success: find log sources using different time references.
15. **Structured logging escape artist** — Test structured log formats for injection via field values that break parsing. Success: find structured log field that breaks JSON/format parsing when injected.
16. **Privileged operation completeness checker** — Verify all privileged operations (admin, config change, data export) are logged. Success: find privileged operation with no audit trail.

### Squad 5: Code Quality & Configuration (17 personas)
*Activation: Always active*

1. **Resource leak hunter** — Find unclosed file handles, database connections, network sockets, event listeners. Success: find resource that is opened but never closed on all code paths.
2. **Dead code detector** — Identify unreachable code, unused exports, commented-out logic that may indicate removed security controls. Success: find dead code that was a security control or contains secrets.
3. **Configuration drift auditor** — Compare dev/staging/prod configs for security-relevant differences, missing hardening. Success: find security setting present in one environment but missing in production.
4. **Default credential checker** — Search for default passwords, admin/admin, test credentials, sample API keys in code. Success: find default or test credential that works in production config.
5. **Memory management auditor** — Check for memory leaks, unbounded caches, missing cleanup in long-running processes. Success: find unbounded memory growth pattern in server process.
6. **Exception handling reviewer** — Check for swallowed exceptions, overly broad catches, missing error propagation. Success: find swallowed exception that hides security-relevant failure.
7. **Null safety auditor** — Check for missing null checks on external data, optional chaining gaps, unsafe assertions. Success: find null dereference on data from external source.
8. **Hardcoded value finder** — Find hardcoded IPs, URLs, ports, paths, magic numbers that should be configurable. Success: find hardcoded value that creates deployment or security fragility.
9. **Unused permission detector** — Identify requested permissions (OAuth scopes, IAM roles, file system) that are never used. Success: find granted permission or scope that is never exercised in code.
10. **Production readiness checker** — Verify production configuration: debug off, error pages safe, test routes removed. Success: find development-only feature accessible in production config.
11. **Fail-open detector** — Find error handling that defaults to allowing access, skipping validation, or granting permissions. Success: find error path that grants access instead of denying.
12. **Thread safety auditor** — Identify shared mutable state without synchronization in concurrent code. Success: find unprotected shared state that causes data race.
13. **Dependency injection escape analyst** — Check for DI container misconfigurations, scope escapes, singleton mutations. Success: find DI scope that leaks state between requests or users.
14. **Signal handler safety auditor** — Check signal/interrupt handlers for unsafe operations, non-reentrant calls. Success: find signal handler performing unsafe operations.
15. **Error message disclosure specialist** — Check all error paths for information leakage in messages returned to users. Success: find error message revealing internal paths, versions, or stack traces.
16. **Environment parity auditor** — Verify dev/prod environment parity for security-relevant configurations. Success: find security mechanism that behaves differently in dev vs prod.
17. **Dynamic code execution auditor** — Find eval, Function(), vm.runInNewContext, exec, dynamic imports, reflection, or equivalent dynamic code execution controlled by user input. Success: find dynamic code execution or reflection with any path to user-controlled input.

---

### Conditionally Active (15 squads — 290 personas)

These squads activate only when their trigger conditions are detected during Phase 0/Phase 1.

### Squad 6: Web Injection & XSS (16 personas)
*Activation: Trigger: .html/.jsx/.tsx/.vue/.svelte files OR web framework in package manifest*

1. **XSS hunter (reflected/stored/DOM)** — Test all output contexts for unescaped user input, dangerouslySetInnerHTML, v-html usage. Success: find user input rendered without escaping in HTML context.
2. **SSTI specialist** — Test server-side template engines for user input in template strings, eval contexts. Success: find user input passed into template rendering engine.
3. **HTML injection specialist** — Test for HTML injection in contexts where full XSS is prevented but markup is injectable. Success: find injectable HTML that modifies page content or phishes users.
4. **CSS injection specialist** — Test for user-controlled CSS that enables data exfiltration or UI redress. Success: find user-controlled style injection that leaks data or modifies UI.
5. **LDAP injection hunter** — Test LDAP query construction for unescaped special characters in user input. Success: find LDAP query with unsanitized user input.
6. **XPath injection hunter** — Test XML/XPath queries for injectable user input in path expressions. Success: find XPath query constructed with unsanitized user input.
7. **Header injection specialist** — Test HTTP response header construction for CRLF injection, header splitting. Success: find user input reflected in HTTP response headers without sanitization.
8. **Open redirect finder** — Test redirect endpoints for unvalidated redirect URLs, path-based bypasses. Success: find redirect accepting arbitrary external URLs.
9. **Prototype pollution hunter** — Test object merging, deep copy, query parsing for __proto__ or constructor.prototype injection. Success: find prototype pollution via object merge with user-controlled keys.
10. **DOM clobbering specialist** — Test for named DOM elements that shadow JavaScript globals or API objects. Success: find DOM element name collision that overrides security-critical code.
11. **Mutation XSS specialist** — Test for HTML that mutates through browser parsing to become executable. Success: find HTML input that is safe pre-parse but executes post-mutation.
12. **SVG/XML injection specialist** — Test SVG upload/embed for script execution, XML entity injection, namespace confusion. Success: find SVG or XML input path that achieves script execution.
13. **Dangling markup injector** — Test for injectable markup that captures subsequent page content via unclosed tags. Success: find injection point where unclosed tag captures sensitive page content.
14. **Polyglot payload crafter** — Test inputs that are valid across multiple contexts (JS, HTML, SQL) simultaneously. Success: find input that passes validation in one context but executes in another.
15. **PDF/document injection specialist** — Test document generation for injectable content, formula injection in CSV/Excel, PDF JavaScript. Success: find user input rendered in generated document without sanitization.
16. **WebAssembly tampering specialist** — Test WASM module loading for integrity verification, tampering detection, memory boundary checks. Success: find WASM module loaded without integrity verification.

### Squad 7: Headers, CORS & Transport (17 personas)
*Activation: Trigger: .html/.jsx/.tsx/.vue/.svelte files OR web framework in package manifest*

1. **CSP auditor** — Check Content-Security-Policy header for unsafe-inline, unsafe-eval, wildcard sources, missing directives. Success: find CSP that allows inline scripts or overly broad sources.
2. **CORS misconfiguration hunter** — Test CORS headers for wildcard origins, reflected origins, credentialed wildcard. Success: find CORS policy that reflects arbitrary origin with credentials.
3. **HSTS verifier** — Check Strict-Transport-Security header presence, max-age, includeSubDomains, preload. Success: find missing HSTS or max-age too short.
4. **Cookie security auditor** — Check cookies for missing Secure, HttpOnly, SameSite, proper Path/Domain scoping. Success: find session cookie missing security flags.
5. **Clickjacking specialist** — Test for missing X-Frame-Options and frame-ancestors CSP directive. Success: find frameable page with sensitive actions.
6. **CSRF hunter** — Test state-changing operations for missing CSRF tokens, SameSite inadequacy, origin checking. Success: find state-changing endpoint with no CSRF protection.
7. **Cache poisoning specialist** — Test for cache key manipulation, unkeyed headers affecting response content. Success: find response that varies by unkeyed input and is cached.
8. **HTTP request smuggling detector** — Test for CL/TE or TE/TE desynchronization between reverse proxy and application. Success: find request parsing difference between proxy layers.
9. **Referrer leak finder** — Check Referrer-Policy, find sensitive URLs leaked via Referer header to third parties. Success: find sensitive data in URL leaked via Referer to external domain.
10. **Mixed content detector** — Find HTTP resources loaded on HTTPS pages, insecure WebSocket, insecure form targets. Success: find insecure resource loaded in secure context.
11. **Service worker persistence analyst** — Test service worker registration for scope hijacking, update bypass, cache persistence. Success: find service worker that persists malicious content or intercepts requests.
12. **SRI auditor** — Check for missing Subresource Integrity hashes on third-party scripts and stylesheets. Success: find external script loaded without SRI hash.
13. **Permissions policy auditor** — Check Permissions-Policy header for overly permissive feature access (camera, microphone, geolocation). Success: find unrestricted browser feature access that should be limited.
14. **Cross-origin isolation auditor** — Test COOP/COEP headers for SharedArrayBuffer safety, Spectre mitigations. Success: find missing cross-origin isolation where high-resolution timers are available.
15. **Cookie prefix enforcer** — Check for __Host- and __Secure- cookie prefix usage on sensitive cookies. Success: find sensitive cookie without appropriate security prefix.
16. **HTTP/2 and HTTP/3 protocol specialist** — Test for HTTP/2 specific attacks: HPACK bomb, stream reset, priority manipulation. Success: find HTTP/2 specific vulnerability or misconfiguration.
17. **Browser storage security auditor** — Check localStorage, sessionStorage, IndexedDB for sensitive data, missing encryption. Success: find sensitive data stored in browser storage without protection.

### Squad 8: WebSocket/GraphQL/gRPC (18 personas)
*Activation: Trigger: ws:/wss: usage, graphql imports, grpc/protobuf imports*

1. **Cross-Site WebSocket Hijacker** — Test WebSocket handshake for missing origin validation, credential leakage. Success: find WebSocket endpoint accepting connections from arbitrary origins.
2. **WebSocket auth bypass tester** — Test for authentication only at handshake, missing per-message authorization. Success: find WebSocket that authenticates on connect but not per message.
3. **WebSocket smuggling specialist** — Test for message framing issues, protocol confusion between WS and HTTP. Success: find WebSocket message that smuggles data past security controls.
4. **GraphQL introspection leaker** — Test for enabled introspection in production exposing full schema. Success: find GraphQL endpoint with introspection enabled in production.
5. **GraphQL batching abuser** — Test for unbounded query batching that bypasses rate limits or amplifies attacks. Success: find GraphQL endpoint accepting unlimited batched queries.
6. **GraphQL complexity bomb** — Test for missing query depth/complexity limits, deeply nested or aliased queries. Success: find GraphQL query that causes excessive computation or timeout.
7. **GraphQL authorization tester** — Test field-level and type-level authorization in GraphQL resolvers. Success: find GraphQL field accessible without proper authorization check.
8. **gRPC reflection exploiter** — Test for gRPC reflection service enabled in production, exposing service definitions. Success: find gRPC reflection endpoint accessible in production.
9. **gRPC plaintext detector** — Test for gRPC services running without TLS encryption. Success: find gRPC service accepting plaintext connections.
10. **gRPC auth bypass tester** — Test gRPC interceptors for missing authentication, metadata-based auth bypass. Success: find gRPC method callable without proper authentication.
11. **Protobuf injection specialist** — Test protobuf message handling for unknown field preservation, type confusion. Success: find protobuf handling that allows type confusion or data injection.
12. **GraphQL federation subgraph injector** — Test federated GraphQL for subgraph injection, entity type hijacking. Success: find federation configuration allowing unauthorized subgraph registration.
13. **GraphQL field suggestion enumerator** — Test for field suggestion leaking schema information in error messages. Success: find GraphQL error message revealing valid field names.
14. **WebSocket subscription exhaustion specialist** — Test for unbounded subscription creation causing resource exhaustion. Success: find WebSocket endpoint allowing unlimited active subscriptions.
15. **gRPC metadata injection specialist** — Test gRPC metadata (headers) for injection, forwarding of untrusted values. Success: find gRPC metadata passed to backend without validation.
16. **API schema drift detector** — Test for runtime API behavior diverging from OpenAPI/protobuf schema definitions. Success: find endpoint behavior inconsistent with schema definition.
17. **GraphQL persisted query abuse specialist** — Test persisted query mechanisms for injection, cache confusion, bypass of query allowlists. Success: find persisted query mechanism that accepts arbitrary queries.
18. **SSE security analyst** — Test Server-Sent Events for auth bypass, data leakage, missing access control on event streams. Success: find SSE endpoint leaking data without proper authorization.

### Squad 9: API REST & Endpoints (19 personas)
*Activation: Trigger: api/ or routes/ directory OR route definition patterns*

1. **BOLA/IDOR hunter** — Test object references for horizontal access control bypass, predictable IDs, missing ownership checks. Success: find API endpoint returning another user's data via ID manipulation.
2. **Broken function level authorization** — Test admin/privileged endpoints for access without proper role checks. Success: find admin endpoint accessible by regular user.
3. **Mass assignment exploiter** — Test for unprotected fields in create/update operations, role/isAdmin injection. Success: find endpoint accepting fields that should be server-controlled.
4. **Rate limit bypasser** — Test rate limiting for bypass via headers, IP rotation, parameter variation, endpoint aliases. Success: find rate limit that can be bypassed or reset.
5. **API versioning exploiter** — Test old API versions for removed security controls, deprecated but active endpoints. Success: find older API version with weaker security than current.
6. **Verb tampering specialist** — Test unexpected HTTP methods on endpoints, method override headers. Success: find endpoint behaving differently with unexpected HTTP method.
7. **Parameter pollution tester** — Test duplicate parameters, array injection, nested object exploitation. Success: find parameter handling inconsistency that bypasses validation.
8. **Content-type confusion** — Test endpoints with unexpected Content-Type headers, type negotiation manipulation. Success: find endpoint that processes body differently based on Content-Type.
9. **Pagination/filtering exploiter** — Test pagination for data leakage, filter injection, unlimited page sizes, cursor manipulation. Success: find pagination that leaks data or allows unbounded queries.
10. **Bulk endpoint abuse** — Test bulk operation endpoints for missing per-item authorization, amplification attacks. Success: find bulk endpoint that skips authorization on individual items.
11. **API key scope tester** — Test API key permissions for overly broad scope, missing key rotation, key leakage in URLs. Success: find API key with broader permissions than needed.
12. **Response data leakage** — Test API responses for excessive data exposure, debug fields, internal IDs, metadata. Success: find API response containing data not needed by the client.
13. **Shadow API cartographer** — Map undocumented endpoints, test routes, debug endpoints, health checks with sensitive data. Success: find undocumented API endpoint accessible in production.
14. **Zombie API necromancer** — Find deprecated but still active API endpoints that have been abandoned but not removed. Success: find deprecated endpoint still accepting requests without current security controls.
15. **SSRF via API callback specialist** — Test webhook URLs, callback parameters, URL inputs for server-side request forgery. Success: find API parameter that triggers server-side requests to attacker-controlled URLs.
16. **OpenAPI/Swagger exposure exploiter** — Test for exposed API documentation revealing internal endpoints, schemas, auth mechanisms. Success: find exposed Swagger/OpenAPI endpoint in production with sensitive details.
17. **Unsafe API consumption auditor** — Test outgoing API calls for missing TLS verification, SSRF, response injection. Success: find outgoing API call that doesn't validate response or verify TLS.
18. **API response header manipulation specialist** — Test for missing security headers on API responses, header injection in API context. Success: find API endpoint missing security headers present on web endpoints.
19. **GraphQL-REST boundary confusion specialist** — Test for inconsistent auth/validation between GraphQL and REST endpoints serving same data. Success: find data accessible via GraphQL that is restricted on REST equivalent.

### Squad 10: OAuth/JWT/Sessions (20 personas)
*Activation: Trigger: oauth, jwt, jsonwebtoken, passport, next-auth, clerk, auth0 imports*

1. **JWT alg:none attacker** — Test JWT validation for algorithm none acceptance, missing algorithm enforcement. Success: find JWT validation that accepts alg:none or unsigned tokens.
2. **JWT key confusion attacker** — Test for RS256/HS256 confusion, using public key as HMAC secret. Success: find JWT validation vulnerable to algorithm switching attack.
3. **JWT kid injector** — Test kid header for path traversal, SQL injection, key file manipulation. Success: find JWT kid parameter that is injectable.
4. **JWT jwk/jku spoofer** — Test for JWT accepting attacker-supplied keys via jwk/jku header parameters. Success: find JWT validation fetching keys from attacker-controlled URL.
5. **JWT weak secret brute-forcer** — Identify common or weak JWT signing secrets, dictionary-attackable keys. Success: find JWT signed with weak, guessable, or common secret.
6. **JWT expiration bypass** — Test for missing or ignored expiration claims, overly long token lifetimes. Success: find JWT that works after expiration or has excessive lifetime.
7. **OAuth redirect_uri bypasser** — Test redirect URI validation for subdomain bypass, path traversal, fragment tricks, open redirects. Success: find OAuth redirect_uri that accepts attacker-controlled destination.
8. **OAuth PKCE enforcer** — Test OAuth flows for missing PKCE, optional PKCE enforcement, S256 downgrade. Success: find OAuth flow that works without PKCE or with plain code verifier.
9. **OAuth state/CSRF tester** — Test for missing state parameter validation, predictable state, reusable state values. Success: find OAuth flow vulnerable to CSRF via missing or weak state.
10. **OAuth device flow abuser** — Test device code flow for polling abuse, code reuse, social engineering vectors. Success: find device flow without rate limiting or with reusable codes.
11. **Session fixation hunter** — Test for session ID reuse across authentication boundary, pre-auth session adoption. Success: find session that persists through login without regeneration.
12. **Session timeout tester** — Test session duration, idle timeout, absolute timeout enforcement. Success: find session with no timeout or timeout not enforced server-side.
13. **Token storage auditor** — Check where tokens are stored: localStorage (XSS risk), cookies (flag check), memory. Success: find auth token stored in XSS-accessible location.
14. **Cross-tenant impersonation** — Test multi-tenant token/session isolation, tenant ID in JWT without validation. Success: find token from one tenant accepted by another tenant's context.
15. **Token sidejacking analyst** — Test for token transmission over insecure channels, missing binding to connection. Success: find token sent without TLS or bindable to attacker's session.
16. **Refresh token rotation auditor** — Test refresh token reuse detection, rotation enforcement, revocation propagation. Success: find refresh token that is reusable after rotation.
17. **OIDC discovery manipulator** — Test for OIDC well-known endpoint manipulation, issuer validation bypass. Success: find OIDC configuration accepting manipulated discovery document.
18. **JWE wrapping attack specialist** — Test JWE for algorithm confusion, key wrapping bypasses, invalid curve attacks. Success: find JWE decryption vulnerable to key recovery or algorithm confusion.
19. **Concurrent session abuser** — Test for missing concurrent session limits, session stealing without invalidation. Success: find system allowing unlimited concurrent sessions per user.
20. **Token binding/DPoP absence analyst** — Test for missing proof-of-possession, bearer tokens usable from any client. Success: find bearer token with no binding to originating client.

### Squad 11: Auth & Identity (19 personas)
*Activation: Trigger: login/signup/register/auth patterns, user models, password fields*

1. **Privilege escalation hunter** — Test for vertical privilege escalation via role manipulation, parameter tampering, forced browsing. Success: find path from low-privilege to high-privilege access.
2. **Missing authorization checker** — Map all endpoints and verify each has authorization check, find unprotected routes. Success: find endpoint serving sensitive data with no authorization check.
3. **Incorrect authorization checker** — Test authorization logic for flaws: OR vs AND, negation errors, scope mismatches. Success: find authorization check that passes when it should deny.
4. **Password policy tester** — Test password requirements enforcement, minimum length, complexity, breach database checking. Success: find weak password accepted by registration/change flow.
5. **Password storage auditor** — Check password hashing algorithm, salt usage, work factor, plaintext storage. Success: find passwords stored with weak hashing or insufficient work factor.
6. **MFA bypass specialist** — Test MFA for bypass via backup codes, session manipulation, race conditions, API routes. Success: find path to authenticate without completing MFA challenge.
7. **Account enumeration tester** — Test login, registration, reset flows for user existence disclosure via timing or response differences. Success: find endpoint that reveals whether account exists.
8. **Account takeover chain builder** — Combine multiple lower-severity issues into account takeover chain. Success: build multi-step attack chain leading to full account takeover.
9. **Registration abuse tester** — Test registration for duplicate accounts, domain spoofing, disposable email acceptance. Success: find registration bypass that creates unauthorized accounts.
10. **Role/permission model auditor** — Audit RBAC/ABAC implementation for gaps, inconsistencies, missing checks. Success: find permission model inconsistency that grants unintended access.
11. **SAML assertion manipulator** — Test SAML for XML signature wrapping, assertion manipulation, replay attacks. Success: find SAML flow vulnerable to assertion manipulation.
12. **SCIM provisioning tester** — Test SCIM endpoints for unauthorized user creation, attribute manipulation. Success: find SCIM endpoint allowing unauthorized user provisioning.
13. **Password reset poisoner** — Test password reset for host header injection, token in referrer, token reuse. Success: find password reset token that can be intercepted or reused.
14. **2FA/MFA downgrade specialist** — Test for ability to disable MFA, downgrade from hardware key to SMS, bypass enrollment. Success: find path to downgrade or disable MFA without proper authorization.
15. **Magic link token analyst** — Test magic link tokens for predictability, reuse, expiration, leakage in referrer. Success: find magic link token that is reusable or predictable.
16. **Passkey/WebAuthn attacker** — Test WebAuthn for credential ID manipulation, challenge replay, origin validation bypass. Success: find WebAuthn implementation with exploitable validation gap.
17. **SSO configuration bypass specialist** — Test for ability to bypass SSO requirement via direct login, password fallback. Success: find authentication path that bypasses mandatory SSO.
18. **Credential stuffing analyst** — Test for missing credential stuffing protections, rate limiting, account lockout. Success: find login endpoint vulnerable to automated credential testing.
19. **Authorization context confusion specialist** — Test for context-dependent authorization that can be confused across features or tenants. Success: find authorization check using wrong context for access decision.

### Squad 12: Payments & Financial (19 personas)
*Activation: Trigger: stripe, solana/web3.js, ethers, web3, payment, checkout patterns*

1. **Fee manipulation specialist** — Test fee calculations for manipulation, negative fees, fee bypass, rounding exploitation. Success: find fee calculation that can be manipulated to reduce or eliminate fees.
2. **Double-spend racer** — Test for race conditions in payment processing, duplicate transaction submission. Success: find race condition that allows same funds to be spent twice.
3. **Replay attacker** — Test for transaction replay, missing nonce/idempotency, reusable payment tokens. Success: find transaction that can be replayed for duplicate effect.
4. **Partial transaction exploiter** — Test multi-step payment flows for incomplete transaction states, partial payment acceptance. Success: find payment flow that can be left in partial state with benefit to attacker.
5. **Refund abuse tester** — Test refund logic for double refund, refund without purchase, refund to different method. Success: find refund mechanism that can be exploited for financial gain.
6. **Price manipulation** — Test for client-side price setting, price mismatch between display and charge, unsigned prices. Success: find price that can be modified by client before server processes payment.
7. **Treasury redirect specialist** — Test for ability to change payment destination addresses, withdrawal targets. Success: find payment routing that can be redirected to attacker-controlled account.
8. **Webhook forgery tester** — Test payment webhooks for missing signature verification, replay, body manipulation. Success: find webhook endpoint that accepts unverified payment notifications.
9. **Coupon/promo abuse** — Test coupon/promo code logic for stacking, reuse, negative discounts, unlimited usage. Success: find coupon mechanism that can be exploited beyond intended use.
10. **Subscription manipulation** — Test subscription logic for plan downgrade retaining premium features, trial abuse. Success: find subscription state manipulation that grants unpaid access.
11. **PCI DSS surface mapper** — Map all points where card data is handled, stored, or transmitted, assess PCI scope. Success: find card data stored, logged, or transmitted insecurely.
12. **Transaction integrity verifier** — Verify transaction atomicity, consistent state, rollback on failure. Success: find transaction that can result in inconsistent financial state.
13. **Decimal precision exploiter** — Test financial calculations for rounding errors, precision loss, truncation exploitation. Success: find decimal precision error that causes financial discrepancy.
14. **Idempotency key manipulator** — Test idempotency key handling for collision, reuse, bypass of duplicate detection. Success: find idempotency mechanism that can be bypassed or confused.
15. **Settlement timing arbitrageur** — Test for timing gaps between authorization and settlement that enable exploitation. Success: find timing window in payment flow that allows state manipulation.
16. **Currency conversion exploiter** — Test currency conversion for rounding exploitation, rate manipulation, conversion bypass. Success: find currency conversion that can be exploited for financial gain.
17. **Webhook race condition exploiter** — Test for race conditions between webhook processing and user-facing status updates. Success: find race between webhook and UI that allows premature access.
18. **Gift card/store credit abuse specialist** — Test gift card and credit systems for generation, duplication, balance manipulation. Success: find store credit or gift card mechanism that can be exploited.
19. **Payment processor downgrade specialist** — Test for ability to force fallback to less secure payment processor or method. Success: find payment flow that can be forced to use weaker processing path.

### Squad 13: Database & Data (18 personas)
*Activation: Trigger: prisma, sequelize, mongoose, pg, mysql, sqlite, redis, drizzle imports*

1. **SQL injection deep diver** — Test beyond basic injection: second-order, blind, time-based, out-of-band SQL injection. Success: find SQL injection vector including blind or second-order variants.
2. **NoSQL operator injection** — Test for MongoDB operator injection, Redis command injection, DynamoDB condition injection. Success: find NoSQL query accepting operator objects from user input.
3. **ORM bypass specialist** — Test ORM queries for raw query escape hatches, filter bypass, relation traversal exploitation. Success: find ORM usage that can be bypassed to execute unintended queries.
4. **Race condition racer** — Test database operations for TOCTOU, missing transactions, non-atomic check-then-act patterns. Success: find database race condition that corrupts data or bypasses checks.
5. **Connection exhaustion attacker** — Test for missing connection pool limits, connection leak, pool poisoning. Success: find path to exhaust database connection pool.
6. **Migration safety auditor** — Check database migrations for data loss risk, irreversible changes, missing rollback plans. Success: find migration that could cause data loss or has no rollback.
7. **Data-at-rest encryption verifier** — Verify sensitive data encryption at rest, check for plaintext PII in database. Success: find sensitive data stored in plaintext in database.
8. **Backup security auditor** — Check for backup access controls, encryption, tested restore procedures. Success: find database backup stored without encryption or access control.
9. **PII detector** — Scan database schemas and queries for unprotected personally identifiable information. Success: find PII stored without encryption or access controls.
10. **Cache poisoning specialist** — Test Redis/Memcached for cache poisoning, key injection, deserialization attacks. Success: find cache key that can be manipulated to serve poisoned content.
11. **Query performance exploiter** — Test for queries that can be made deliberately slow via crafted input, missing indexes. Success: find query input that causes table scan or excessive computation.
12. **Second-order SQL injection specialist** — Test for stored data that becomes injectable when used in subsequent queries. Success: find stored value that triggers SQL injection when used in later query.
13. **Stored procedure privilege escalator** — Test stored procedures for privilege escalation, unsafe dynamic SQL, definer context abuse. Success: find stored procedure that runs with elevated privileges exploitably.
14. **ORM parameter injection specialist** — Test ORM method parameters for injection of query operators or relation traversal. Success: find ORM method that accepts injectable parameters from user input.
15. **Database link/federation abuser** — Test database links, foreign data wrappers for unauthorized cross-database access. Success: find database federation that exposes data across trust boundaries.
16. **PostgreSQL extension exploiter** — Test for dangerous PostgreSQL extensions, extension-based code execution. Success: find PostgreSQL extension that enables unauthorized code execution.
17. **Multi-tenancy data isolation analyst** — Test tenant data isolation at query, schema, and connection levels. Success: find path to access another tenant's data.
18. **Database feature escalation specialist** — Test for database features (jobs, mail, filesystem access) that enable privilege escalation. Success: find database feature that extends access beyond intended scope.

### Squad 14: AI/LLM Security (22 personas)
*Activation: Trigger: openai, anthropic, langchain, llamaindex, ai-sdk, model imports*

1. **Prompt injector (direct)** — Test for direct prompt injection in user inputs sent to LLM, system prompt override. Success: find user input that overrides system prompt or changes LLM behavior.
2. **Prompt injector (indirect)** — Test for prompt injection via data sources: documents, emails, web content fed to LLM. Success: find data source that injects instructions when processed by LLM.
3. **System prompt extractor** — Test for ability to extract system prompt via crafted user messages. Success: find technique that causes LLM to reveal its system prompt.
4. **Data exfiltrator via LLM** — Test for LLM-mediated data exfiltration via crafted outputs, tool calls, markdown rendering. Success: find path where LLM can be tricked into exfiltrating data.
5. **Excessive agency tester** — Test for LLM actions without proper human approval, overly broad tool access. Success: find LLM with ability to perform sensitive actions without approval gate.
6. **Cost DOS attacker** — Test for ability to trigger expensive LLM calls, large context windows, repeated invocations. Success: find input that causes disproportionate LLM cost.
7. **Output handler auditor** — Test LLM output handling for injection into SQL, HTML, commands, file paths. Success: find LLM output used in unsafe context without sanitization.
8. **RAG poisoning specialist** — Test retrieval-augmented generation for document injection, ranking manipulation. Success: find RAG pipeline that can be poisoned via injected documents.
9. **Embedding inversion tester** — Test embedding endpoints for information leakage, training data recovery from embeddings. Success: find embedding that leaks sensitive information about source data.
10. **Model provenance verifier** — Check model sources, verify model integrity, detect model replacement or tampering. Success: find model loaded without integrity verification.
11. **Jailbreak resistance tester** — Test LLM guardrails for bypass techniques, multi-turn jailbreaks, encoding tricks. Success: find jailbreak technique that bypasses content safety controls.
12. **Hallucination impact assessor** — Identify where hallucinated outputs could cause security decisions, code execution, or data corruption. Success: find hallucination-vulnerable path that affects security-critical decisions.
13. **Token/context window abuse** — Test for context window overflow attacks, token counting bypass, context manipulation. Success: find input that exhausts context window causing instruction loss.
14. **Multi-turn manipulation** — Test for gradual manipulation across conversation turns to bypass safety measures. Success: find multi-turn sequence that achieves restricted outcome.
15. **Training data extraction specialist** — Test for memorized training data leakage via crafted prompts. Success: find prompt that extracts memorized sensitive data from model.
16. **Membership inference attacker** — Test whether specific data was in training set via model behavior analysis. Success: find technique to determine training data membership.
17. **Fine-tuning poisoning specialist** — Test fine-tuning pipelines for data poisoning, safety alignment degradation. Success: find fine-tuning input that degrades model safety behavior.
18. **Model fingerprinting analyst** — Test for model version/type disclosure via output characteristics. Success: find technique to identify specific model version being used.
19. **Differential privacy failure auditor** — Test for missing or inadequate differential privacy in data pipelines feeding LLMs. Success: find data pipeline lacking privacy guarantees for sensitive data.
20. **Multimodal injection specialist** — Test image/audio/video inputs for embedded prompt injection, steganographic instructions. Success: find multimodal input that contains hidden prompt injection.
21. **AI capability weaponization tester** — Test for AI capabilities that could be weaponized: code generation, social engineering, misinformation. Success: find AI feature that can be leveraged for harmful output generation.
22. **Vector store access control tester** — Test vector databases and embedding stores for unauthorized queries, cross-tenant embedding leakage, similarity search manipulation via adversarial embeddings, and missing access controls on semantic search. Success: find vector store query that returns embeddings from unauthorized context or tenant.

### Squad 15: Single-Agent & MCP Exploitation (14 personas)
*Activation: Trigger: mcp, tool_use, function_calling patterns, .claude/ directory*

1. **MCP server trust auditor** — Test MCP server connections for trust verification, TLS, authentication. Success: find MCP server connection without proper trust verification.
2. **Agent goal hijacking tester** — Test for ability to redirect agent goals via crafted inputs or tool results. Success: find input that causes agent to pursue attacker-specified goal.
3. **Memory poisoning specialist** — Test agent memory/context for persistent injection that affects future interactions. Success: find injected memory that alters agent behavior in subsequent sessions.
4. **Agent permission scope tester** — Test agent tool permissions for overly broad access, missing least-privilege. Success: find agent with tool access beyond what its task requires.
5. **Rogue agent detector** — Test for agents that deviate from intended behavior, hidden agency, goal misalignment. Success: find agent behavior that diverges from specified instructions.
6. **Sandbox escape tester** — Test agent sandboxing for escape vectors, filesystem access, network access beyond intended scope. Success: find agent action that accesses resources outside its sandbox.
7. **Tool result injection** — Test for ability to inject malicious content in tool results that agents process unsafely. Success: find tool result that injects instructions or code into agent context.
8. **Rug pull detector** — Test for tool servers that change behavior post-approval, delayed malicious activation. Success: find tool that could alter its behavior after initial trust establishment.
9. **Tool description poisoning specialist** — Test tool descriptions for hidden instructions that manipulate agent behavior, and test for malicious tool descriptions that inject instructions into agent context. Success: find tool description containing hidden instructions or behavioral manipulation for the agent.
10. **Human-in-the-loop bypass specialist** — Test for paths that skip required human approval in agent workflows. Success: find agent action path that bypasses required human confirmation.
11. **Excessive autonomy auditor** — Test for agent actions that should require approval but execute automatically. Success: find destructive or sensitive agent action with no approval gate.
12. **Agent context window manipulation specialist** — Test for attacks that fill agent context to push out safety instructions. Success: find input that displaces agent instructions from context window.
13. **Tool composition loop detector** — Test for unsafe recursive tool invocation, unintended tool chain loops, and tool composition that causes amplification or side effects. Success: find tool chain that recurses or composes in unintended way causing harmful side effects.
14. **Deceptive agent explanation tester** — Test for agents presenting misleading confidence, fabricating justifications for actions, or framing harmful actions as beneficial to manipulate human approval. Success: find agent output that frames a risky action with misleading confidence or false justification.

### Squad 16: Blockchain/Web3 (19 personas)
*Activation: Trigger: solana, ethereum, ethers, web3, anchor, hardhat patterns*

1. **Transaction warfare specialist** — Test transaction ordering attacks, frontrunning, sandwich attacks, MEV exploitation. Success: find transaction flow vulnerable to ordering manipulation.
2. **Program authority checker** — Test for missing authority/signer checks on privileged program instructions. Success: find program instruction callable without required authority.
3. **Signature forger** — Test signature validation for missing checks, partial verification, signature malleability. Success: find signature validation that accepts forged or malleable signatures.
4. **Token standard specialist** — Test token implementations for standard deviations, reentrancy via callbacks, approval race conditions. Success: find token implementation that deviates from standard in exploitable way.
5. **Bridge attack specialist** — Test cross-chain bridge logic for message forgery, replay attacks, validator collusion. Success: find bridge message that can be forged or replayed.
6. **Flash loan exploiter** — Test for flash loan vulnerable patterns: oracle manipulation, governance attacks, liquidation exploits. Success: find protocol logic exploitable via flash loan.
7. **Reentrancy hunter** — Test for reentrancy vulnerabilities in external calls, cross-function reentrancy, read-only reentrancy. Success: find external call that allows reentrant state manipulation.
8. **Integer overflow in smart contracts** — Test arithmetic operations in contracts for overflow/underflow without SafeMath. Success: find unchecked arithmetic that overflows in contract.
9. **Access control on-chain** — Test on-chain access control for missing checks, role manipulation, admin key exposure. Success: find on-chain function missing required access control.
10. **Instruction data validation** — Test program instruction inputs for missing validation, type confusion, buffer overflows. Success: find program instruction that processes unvalidated input.
11. **PDA derivation auditor** — Test PDA derivation for seed manipulation, collision attacks, missing bump verification. Success: find PDA derivation with manipulable seeds or missing bump check.
12. **Treasury/vault security** — Test treasury and vault contracts for unauthorized withdrawal, drain attacks, access control. Success: find path to unauthorized treasury withdrawal.
13. **Oracle manipulation specialist** — Test price oracles for manipulation, stale data, single-source dependency, flash loan attacks. Success: find oracle that can be manipulated to return incorrect data.
14. **Governance attack analyst** — Test governance for flash loan voting, proposal manipulation, timelock bypass. Success: find governance mechanism exploitable via token borrowing or timing.
15. **Permit/approval abuse hunter** — Test token approvals for unlimited allowance, approval race, permit replay. Success: find token approval mechanism that can be exploited.
16. **Upgradeable proxy auditor** — Test proxy patterns for storage collision, initialization frontrunning, unauthorized upgrade. Success: find proxy that can be upgraded without proper authorization.
17. **Clock/slot manipulation analyst** — Test for reliance on block timestamps or slot numbers that can be manipulated by validators. Success: find logic dependent on manipulable block time or slot.
18. **Cross-chain message verification specialist** — Test cross-chain message verification for spoofing, replay, missing source validation. Success: find cross-chain message accepted without proper source verification.
19. **NFT/metadata manipulation analyst** — Test NFT metadata for injection, URI manipulation, off-chain metadata tampering. Success: find NFT metadata that can be manipulated to mislead users.

### Squad 17: Cloud/Container/Serverless (19 personas)
*Activation: Trigger: Dockerfile, docker-compose, k8s manifests, terraform, serverless.yml, AWS/GCP/Azure SDK*

1. **SSRF to cloud metadata** — Test for SSRF vectors reaching cloud metadata endpoints (169.254.169.254, metadata.google.internal). Success: find request path that reaches cloud metadata service.
2. **IAM over-permission hunter** — Test IAM policies for wildcard permissions, overly broad resource access, unused permissions. Success: find IAM policy granting more permissions than needed.
3. **K8s RBAC auditor** — Test Kubernetes RBAC for overly permissive roles, default service account abuse, privilege escalation. Success: find K8s role with excessive permissions.
4. **Container escape tester** — Test for container escape vectors: privileged mode, mounted sockets, kernel exploits, capabilities. Success: find container configuration that allows escape to host.
5. **Serverless event injection** — Test serverless function triggers for event injection, untrusted event data processing. Success: find serverless function that processes unvalidated event data.
6. **Secret management auditor** — Check for secrets in environment variables, config files, container images vs proper secret stores. Success: find secret stored outside of dedicated secret management.
7. **Cold start info leakage** — Test for information leakage during serverless cold starts, initialization logs, timing. Success: find sensitive information leaked during function initialization.
8. **Service account abuse** — Test for default service account usage, shared credentials, over-permissioned service identities. Success: find service account with permissions beyond its function's needs.
9. **Network segmentation verifier** — Test network policies for missing segmentation, lateral movement paths, exposed internal services. Success: find internal service accessible from untrusted network segment.
10. **Cloud storage exposure** — Test for public buckets, overly permissive ACLs, misconfigured storage policies. Success: find cloud storage object accessible without authentication.
11. **Infrastructure drift detector** — Test for differences between IaC definitions and actual deployed infrastructure. Success: find deployed resource that doesn't match IaC definition.
12. **Cost exhaustion attacker** — Test for inputs that trigger expensive cloud operations, auto-scaling abuse, resource amplification. Success: find input that causes disproportionate cloud cost.
13. **Assume role chain exploiter** — Test for role assumption chains that escalate privileges across accounts or services. Success: find role chain that grants broader access than any individual role.
14. **Resource policy backdoor analyst** — Test resource policies for backdoor access, cross-account trust, wildcard principals. Success: find resource policy granting unintended cross-account access.
15. **Lambda layer poisoning specialist** — Test for shared Lambda layers with code injection, version manipulation. Success: find Lambda layer that can be poisoned to affect multiple functions.
16. **Container registry poisoner** — Test container registry access controls, image signing, tag mutability. Success: find container registry allowing unauthorized image push or tag overwrite.
17. **CloudTrail evasion detector** — Test for actions that bypass cloud audit logging, log gaps, service-specific logging omissions. Success: find cloud action that doesn't generate audit log entry.
18. **IMDSv1 vs v2 enforcement specialist** — Test for instances using IMDSv1 instead of v2, missing hop limit enforcement. Success: find instance metadata accessible via IMDSv1 without token.
19. **Cloud storage object-level ACL analyst** — Test for object-level ACLs that override bucket policy, individual object exposure. Success: find cloud storage object with ACL more permissive than bucket policy.

### Squad 18: Cryptography & Encryption (18 personas)
*Activation: Trigger: crypto, bcrypt, argon2, openssl, tls, certificate, encrypt/decrypt patterns*

1. **Weak algorithm detector** — Identify MD5, SHA-1, DES, RC4, ECB mode, and other deprecated algorithms in security contexts. Success: find deprecated cryptographic algorithm used for security purpose.
2. **Key management auditor** — Check for hardcoded keys, key material in source, missing key rotation, insecure key storage. Success: find cryptographic key stored insecurely or hardcoded.
3. **Random number auditor** — Check for Math.random(), predictable seeds, insufficient entropy in security contexts. Success: find non-cryptographic random number used for security-sensitive value.
4. **TLS configuration tester** — Test for TLS version support, cipher suite configuration, certificate validation. Success: find weak TLS configuration or missing certificate validation.
5. **Certificate validation checker** — Test for disabled certificate verification, pinning bypass, expired certificate acceptance. Success: find code that disables or weakens certificate validation.
6. **Padding oracle tester** — Test for padding oracle vulnerabilities in CBC mode encryption, error message differences. Success: find encryption that leaks padding validity information.
7. **Timing-safe comparison checker** — Test for non-constant-time comparison of secrets, MACs, tokens, passwords. Success: find secret comparison vulnerable to timing side-channel.
8. **Hash collision exploiter** — Test for hash collision vulnerabilities in identity, integrity, or deduplication contexts. Success: find hash usage where collisions could cause security impact.
9. **Key derivation auditor** — Check for proper KDF usage, salt uniqueness, iteration count adequacy. Success: find key derivation with missing salt or insufficient iterations.
10. **Constant-time operation verifier** — Verify security-critical operations are constant-time, check for early returns on secret data. Success: find security operation with data-dependent timing.
11. **Nonce/IV reuse detector** — Check for nonce or initialization vector reuse in encryption operations. Success: find encryption reusing nonce or IV across operations.
12. **Protocol downgrade attack analyst** — Test for ability to force downgrade to weaker cryptographic protocols or cipher suites. Success: find protocol negotiation vulnerable to downgrade attack.
13. **Key reuse across contexts auditor** — Check for same key used for different purposes (encryption vs signing, different tenants). Success: find cryptographic key reused across different security contexts.
14. **Certificate transparency monitor** — Check for certificate transparency enforcement, CT log monitoring, rogue certificate detection. Success: find certificate used without CT log presence.
15. **KDF misuse specialist** — Test for improper KDF usage: wrong algorithm, insufficient output length, missing purpose separation. Success: find KDF usage that weakens derived key security.
16. **Cryptographic side-channel analyst** — Test for side-channel leakage via timing, power, cache, or error messages in crypto operations. Success: find cryptographic operation leaking information via side channel.
17. **Key ceremony/HSM boundary auditor** — Check for key operations outside HSM boundary, missing ceremony procedures. Success: find key operation that should be in HSM but occurs in software.
18. **Zero-knowledge proof verification auditor** — Test ZK proof verification for soundness gaps, missing verification steps, proof malleability. Success: find ZK proof verification with exploitable gap.

### Squad 19: Privacy & Compliance (18 personas)
*Activation: Trigger: user data storage, email/phone/address fields, GDPR/CCPA/HIPAA references*

1. **GDPR right-to-deletion auditor** — Test data deletion completeness: backups, caches, logs, third-party copies, derived data. Success: find user data that persists after deletion request.
2. **PII in logs/errors/URLs** — Search for personal data appearing in log statements, error messages, query strings, URLs. Success: find PII written to logs, error output, or URL parameters.
3. **Data minimization checker** — Verify only necessary data is collected, stored, and processed for each feature. Success: find data collection beyond what is needed for stated purpose.
4. **Consent mechanism auditor** — Test consent flows for dark patterns, pre-checked boxes, insufficient granularity. Success: find consent mechanism that doesn't meet regulatory requirements.
5. **Cross-border data transfer checker** — Identify data flows crossing jurisdictional boundaries without adequate legal basis. Success: find data transfer to jurisdiction without adequate protection.
6. **Data retention policy auditor** — Verify data retention policies exist and are enforced, expired data is deleted. Success: find data retained beyond stated or required retention period.
7. **Third-party data sharing auditor** — Map all third-party data flows, verify data processing agreements, consent coverage. Success: find data shared with third party without adequate legal basis.
8. **Anonymization verifier** — Test anonymized datasets for re-identification risk, quasi-identifier combinations. Success: find "anonymized" data that can be re-identified.
9. **Children's data (COPPA) checker** — Test for age verification, parental consent, data minimization for users under 13. Success: find missing age gate or children's data handled without COPPA compliance.
10. **Health data (HIPAA) checker** — Identify health information in non-HIPAA-compliant storage or transmission paths. Success: find health data processed without HIPAA-required safeguards.
11. **Dark pattern detector** — Test UI flows for manipulative patterns: trick questions, hidden costs, forced continuity. Success: find dark pattern that manipulates user into unintended data sharing.
12. **Browser fingerprinting auditor** — Test for device fingerprinting, canvas fingerprinting, tracking without consent. Success: find fingerprinting technique used without informed consent.
13. **Metadata leakage hunter** — Test for metadata in uploaded files, API responses, documents that leaks user information. Success: find metadata leaking location, device, or identity information.
14. **Correlation attack analyst** — Test for combinable data points across features that enable user tracking or de-anonymization. Success: find data combination that enables user identification or tracking.
15. **Supercookie/persistent tracking hunter** — Test for tracking mechanisms that survive cookie deletion: ETags, HSTS, localStorage. Success: find tracking mechanism that persists beyond standard cookie clearing.
16. **DSAR weaponization analyst** — Test data subject access request handling for abuse: data exfiltration, enumeration. Success: find DSAR mechanism that can be abused to access others' data.
17. **Privacy-preserving analytics auditor** — Test analytics implementation for individual tracking, missing aggregation, raw data exposure. Success: find analytics collecting individual-level data without privacy controls.
18. **Cross-regulation conflict analyst** — Identify data handling that satisfies one regulation but violates another. Success: find compliance approach that creates conflict between regulatory frameworks.

---

### Language-Triggered (2 squads — 32 personas)

These squads activate when specific programming languages or patterns are detected.

### Squad 20: Memory Safety (16 personas)
*Activation: Trigger: .c, .cpp, .h, .rs, .wasm files, C/C++ toolchain detected*

1. **Buffer overflow hunter** — Test for stack and heap buffer overflows in string operations, array access, memory copies. Success: find buffer write that exceeds allocated size.
2. **Use-after-free detector** — Test for dangling pointer dereference after memory deallocation. Success: find memory access after free on any code path.
3. **Out-of-bounds read/write** — Test array and pointer operations for reads or writes beyond allocated bounds. Success: find array/pointer access beyond allocated boundary.
4. **Format string exploiter** — Test for user-controlled format strings in printf-family functions. Success: find format string with user-controlled content.
5. **Double-free detector** — Test for multiple free calls on same pointer via different code paths. Success: find pointer freed more than once.
6. **Null pointer dereference** — Test for null pointer dereference on error paths, uninitialized pointers, failed allocations. Success: find null pointer dereference on reachable code path.
7. **WASM memory safety auditor** — Test WebAssembly modules for memory boundary violations, linear memory escapes. Success: find WASM memory access that violates intended boundaries.
8. **Unsafe Rust auditor** — Test unsafe Rust blocks for soundness violations, undefined behavior, invariant violations. Success: find unsafe Rust block with potential undefined behavior.
9. **Type confusion exploiter** — Test for type confusion via unions, void pointers, unsafe casts, variant misinterpretation. Success: find type confusion that enables memory corruption or logic bypass.
10. **Uninitialized memory reader** — Test for reads of uninitialized memory leaking sensitive data from previous allocations. Success: find uninitialized memory read on reachable code path.
11. **Stack canary bypass analyst** — Test for stack buffer overflows that could bypass or avoid stack canary protections. Success: find overflow that avoids stack canary check.
12. **ROP/JOP chain indicator detector** — Identify code patterns that facilitate return-oriented or jump-oriented programming chains. Success: find gadget-rich code region without CFI protection.
13. **Integer-to-pointer conversion analyst** — Test for unsafe integer-to-pointer conversions that violate pointer provenance. Success: find integer-to-pointer conversion that bypasses memory safety.
14. **Speculative-execution attacker** — Test for Spectre-style speculative execution vulnerabilities in security-critical code. Success: find branch prediction exploitable for data leakage.
15. **Memory allocator exploitation specialist** — Test for heap metadata corruption, use-after-free exploitation via allocator behavior. Success: find heap corruption exploitable through allocator behavior.
16. **ASLR/DEP/CFI bypass analyst** — Test for information leaks or techniques that weaken memory safety mitigations. Success: find information leak that defeats ASLR, DEP, or CFI.

### Squad 21: Deserialization & Type Safety (16 personas)
*Activation: Trigger: serialize/deserialize/marshal/unmarshal patterns in any language*

1. **Java deserialization** — Test for unsafe ObjectInputStream usage, gadget chain availability, type filtering bypass. Success: find Java deserialization of untrusted data without type filtering.
2. **Python unsafe deserialization** — Test for unsafe deserialization usage with untrusted data, module loading. Success: find Python unsafe deserialization of user-controlled data.
3. **DotNET BinaryFormatter hunter** — Test for BinaryFormatter, NetDataContractSerializer, or similar dangerous formatters. Success: find .NET deserialization using dangerous formatter with untrusted data.
4. **Node.js deserialization chains** — Test for node-serialize, funcster, or custom deserialization with code execution. Success: find Node.js deserialization that enables code execution.
5. **Ruby Marshal/YAML** — Test for Ruby Marshal.load or YAML.load with untrusted data, gadget chains. Success: find Ruby deserialization of untrusted data without safe loading.
6. **PHP unserialize** — Test for unserialize with user data, magic method abuse, property-oriented programming chains. Success: find PHP unserialize processing user-controlled data.
7. **JSON parser confusion** — Test for JSON parsing differences between components, duplicate key handling, type confusion. Success: find JSON parsing inconsistency that bypasses validation.
8. **Protobuf/MessagePack safety** — Test for unsafe protobuf/msgpack deserialization, unknown field handling, type coercion. Success: find binary format deserialization with type safety bypass.
9. **XXE injector** — Test XML parsers for external entity processing, parameter entities, entity expansion. Success: find XML parser that processes external entities from untrusted input.
10. **XSLT injection specialist** — Test for XSLT processing of untrusted stylesheets, code execution via extensions. Success: find XSLT processor accepting untrusted transformations.
11. **YAML anchor/alias bomb specialist** — Test YAML parsers for billion laughs via anchors/aliases, resource exhaustion. Success: find YAML parser vulnerable to exponential entity expansion.
12. **SOAP/XML-RPC injection analyst** — Test SOAP/XML-RPC endpoints for injection, DTD attacks, method enumeration. Success: find SOAP endpoint vulnerable to XML injection or entity attacks.
13. **Protobuf/gRPC confusion specialist** — Test for protobuf schema mismatch, field number reuse, type confusion across versions. Success: find protobuf handling that confuses field types or versions.
14. **GraphQL/JSON schema coercion exploiter** — Test for type coercion differences between schema validation and runtime processing. Success: find schema validation bypass via type coercion.
15. **Polyglot file confusion specialist** — Test for files that are valid in multiple formats, bypass type checking, confuse parsers. Success: find file accepted as one type but parsed as another.
16. **Server-side template deserialization specialist** — Test for template engines that deserialize objects from template strings. Success: find template engine that deserializes untrusted data during rendering.

---

### Context-Triggered (1 squad — 16 personas)

### Squad 22: Vibecoder Detection (16 personas)
*Activation: Always active (runs on every audit)*

1. **AI-generated secret finder** — Detect placeholder secrets, fake API keys, example tokens left by AI code generation. Success: find AI-generated placeholder secret or credential in code.
2. **Missing parameterization detector** — Find hardcoded SQL, queries, commands that should use parameterized statements. Success: find string-concatenated query that should be parameterized.
3. **Client-side auth checker** — Find authentication or authorization logic that only runs on the client side. Success: find auth check with no corresponding server-side enforcement.
4. **Insecure storage detector** — Find sensitive data stored in localStorage, plain cookies, or unencrypted files. Success: find sensitive data stored without appropriate protection.
5. **Missing security header checker** — Find responses missing standard security headers that AI generators often omit. Success: find endpoint missing security headers that should be present.
6. **Copy-paste vulnerability detector** — Find code patterns that match known vulnerable snippets from tutorials or Stack Overflow. Success: find copy-pasted code pattern with known security vulnerability.
7. **Slopsquatting detector** — Find dependencies that don't exist in registries (hallucinated by AI code generators). Success: find imported package that doesn't exist in any package registry.
8. **Incomplete-code completion detector** — Find TODO/FIXME in security-critical paths, stub implementations, placeholder validation. Success: find incomplete implementation in security-critical code path.
9. **Client-side validation truther** — Find validation that exists only on client with no server-side equivalent. Success: find client-side validation with no server-side counterpart.
10. **Missing rate limit detector** — Find endpoints with authentication, payment, or sensitive operations lacking rate limits. Success: find sensitive endpoint with no rate limiting.
11. **Exposed debug endpoint finder** — Find debug routes, test endpoints, admin panels accessible without authentication. Success: find debug or test endpoint accessible in production.
12. **Input length limit neglector** — Find input fields and API parameters with no maximum length validation. Success: find input that accepts unbounded length without server-side limit.
13. **Framework misuse pattern detector** — Find incorrect usage of security framework features that nullifies their protection. Success: find framework security feature used incorrectly.
14. **AI-generated test inadequacy detector** — Find security tests that only test happy paths, missing edge cases and attack vectors. Success: find security test that doesn't actually test the security property.
15. **Dependency over-trust detector** — Find dependencies used without version pinning, integrity checks, or update monitoring. Success: find dependency used without any version constraint or integrity check.
16. **Silent failure/swallowed error detector** — Find empty catch blocks, ignored return values, suppressed errors in security paths. Success: find swallowed error in security-critical code path.

---

### Squad 23: Multi-Agent, Agentic Infrastructure & NHI Security (14 personas)
*Activation: Trigger: mcp, tool_use, function_calling patterns, .claude/ directory, A2A protocol patterns, multi-agent orchestration patterns, service account or NHI patterns*

1. **Inter-agent communication auditor** — Test multi-agent message passing for injection, impersonation, routing manipulation. Success: find inter-agent message that can be spoofed or injected.
2. **Agent supply chain auditor** — Test agent dependencies: tool servers, model providers, data sources for compromise vectors. Success: find agent dependency that could be compromised to control agent.
3. **Shadow MCP server hunter** — Test for unauthorized or hidden MCP server connections, unexpected tool sources. Success: find undocumented MCP server connected to the agent.
4. **Cross-server request forgery attacker** — Test for ability to trigger actions on one MCP server via another server's context. Success: find cross-server action triggered without proper authorization.
5. **Multi-agent consensus manipulation specialist** — Test for ability to manipulate multi-agent voting, consensus, or orchestration. Success: find technique to bias multi-agent decision-making.
6. **Viral agent loop propagator** — Test for compromised agents that spread malicious behaviors to other agents in multi-agent systems through shared context, tool outputs, or inter-agent messages. Success: find agent interaction that propagates unauthorized behavior across agents.
7. **Reasoning-layer attack specialist** — Test for manipulation of agent planning and chain-of-thought to alter decisions without traditional prompt injection, targeting the reasoning process itself. Success: find input that alters agent reasoning without triggering injection defenses.
8. **A2A protocol exploitation specialist** — Test Agent-to-Agent protocol implementations for malicious agent cards, adversarial instructions in discovery, and trust boundary violations between cooperating agents. Success: find A2A agent card or discovery mechanism that can be exploited.
9. **HITL overwhelm attacker** — Test for flooding approval queues to reduce human scrutiny, causing auto-approve behavior or reviewer fatigue that degrades the quality of human oversight. Success: find approval workflow that degrades under volume pressure.
10. **Agent framework supply chain poisoner** — Test agent framework ecosystems (skills, plugins, tools, MCP registries) for malicious components that could compromise agents loading them at runtime. Success: find agent framework component loaded without integrity verification.
11. **Non-human identity lifecycle auditor** — Test machine and service identities for over-permission, stale credentials, missing rotation, and ungoverned access patterns across AI agent deployments. Success: find NHI with excessive permissions or stale credentials.
12. **Zero-click AI agent exploit tester** — Test for exploits requiring no user interaction that target the agent's reasoning process through data it processes, such as poisoned documents, emails, or tool outputs. Success: find agent vulnerable to zero-interaction exploitation via data it processes.
13. **Delegated identity chain auditor** — Test for credential and identity propagation through agent delegation chains, where downstream agents inherit broader privileges than intended through token forwarding or scope escalation. Success: find delegation chain where downstream agent inherits broader privileges than intended.
14. **Cascading failure propagation tester** — Test for error amplification across multi-agent orchestration graphs, including blast radius of single agent failure and hallucinating planners issuing destructive tasks to downstream agents. Success: find single agent failure that propagates uncontrolled to multiple downstream systems.

---

### Wildcard (1 squad — 18 personas)

### Red Team (18 personas)
*Activation: Always active*

1. **Script kiddie** — Attempt common automated attacks: default credentials, known CVEs, public exploits, common misconfigs. Success: find vulnerability exploitable with publicly available tools.
2. **Insider threat** — Test for damage possible with legitimate access: data exfiltration, privilege abuse, audit evasion. Success: find path for authorized user to exceed intended access.
3. **Bot farm operator** — Test for automated abuse: account creation, scraping, resource exhaustion, captcha bypass paths. Success: find automation-exploitable endpoint without bot protection.
4. **Social engineer** — Test for information disclosure useful in social engineering: org charts, error messages, metadata. Success: find information leakage useful for social engineering attacks.
5. **Nation-state actor** — Test for advanced persistent threat vectors: supply chain, zero-day patterns, long-term access. Success: find sophisticated attack vector requiring advanced capabilities.
6. **Competitive analyst** — Test for competitive intelligence exposure: pricing logic, algorithms, customer data leakage. Success: find business-sensitive information accessible without authorization.
7. **Disgruntled employee** — Test for sabotage vectors: backdoors, time bombs, data destruction, credential persistence. Success: find path for insider to cause damage after access revocation.
8. **Supply chain compromiser** — Test for supply chain attack vectors: dependency injection, build process manipulation. Success: find supply chain link that could be compromised to affect all users.
9. **Physical proximity attacker** — Test for attacks requiring network proximity: ARP spoofing, mDNS, local service exposure. Success: find service exposed on local network without authentication.
10. **Regulatory/legal attacker** — Test for compliance violations that could trigger regulatory action or legal liability. Success: find compliance violation that creates regulatory or legal risk.
11. **AI-augmented attacker** — Test for vulnerabilities that AI tools can chain together or exploit at scale. Success: find vulnerability chainable by automated AI-driven attack tools.
12. **Persistence-focused attacker** — Test for persistent access mechanisms: backdoor accounts, API keys, scheduled tasks, cron jobs. Success: find mechanism that maintains access after initial compromise.
13. **Competitive intelligence extractor** — Test for business logic exposure: pricing algorithms, recommendation engines, proprietary formulas. Success: find proprietary business logic exposed or reverse-engineerable.
14. **Ransomware/destructive attack simulator** — Test for mass data encryption, deletion, or corruption vectors accessible via application. Success: find path enabling mass data destruction or encryption.
15. **Purple team detection gap tester** — Test whether the codebase's defenses (logging, monitoring, alerting) would actually detect the attacks found by other squads, bridging offense and defense perspectives. Success: find confirmed vulnerability where no detection or alerting mechanism exists.
16. **Gold team crisis response tester** — Test incident response readiness: whether breaches can be contained, disclosed within regulatory timelines (EU AI Act 72-hour, CRA 24-hour), and forensically preserved with adequate logging. Success: find gap in incident response capability that would delay breach response.
17. **Orange team developer trust exploiter** — Test for attacks that exploit developer trust: malicious packages disguised as internal tools, IDE plugin compromise, weaponized repositories, and developer-targeted social engineering paths in the codebase. Success: find developer-targeted attack vector in the project's development workflow.
18. **Green team deployment gap analyst** — Test the gap between what developers built and what defenders need: logging blind spots, deployment configurations that pass CI but fail security review, and infrastructure that technically works but is indefensible. Success: find deployment configuration that passes automated checks but creates defensive blind spot.

---

**Persona Taxonomy Totals (per-squad counts):**
| Category | Squads | Personas |
|----------|--------|----------|
| Always Active | 5 (Squads 1-5) | 22 + 21 + 16 + 16 + 17 = 92 |
| Conditionally Active | 15 (Squads 6-19, 23) | 16 + 17 + 18 + 19 + 20 + 19 + 19 + 18 + 22 + 14 + 19 + 19 + 18 + 18 + 14 = 290 |
| Language-Triggered | 2 (Squads 20-21) | 16 + 16 = 32 |
| Context-Triggered | 1 (Squad 22) | 16 |
| Wildcard | 1 (Red Team) | 18 |
| **Total** | **24 squads** | **448 personas** |

> **Note for orchestrator:** During Phase 1, extract only the squads selected by detection heuristics. Each squad agent receives its own persona list as part of the context packet. The full taxonomy is never loaded into a single agent's context.

---

## Section 12: Standards Reference

> **Note for orchestrator:** This section is the single source of truth for standards enrichment in Phase 4 (Step 4.2). When enriching a finding, look up CWE here, use the CVSS decision tree to compute the vector string, and map to OWASP/NIST/ATT&CK categories. Do NOT invent mappings not present in these tables.

### 12.1: CWE Lookup Table

The top 62 most security-relevant CWEs covering all CWEs referenced by the persona library. Use this table during Phase 4 to map squad findings to standard CWE identifiers.

| CWE | Name | Category |
|-----|------|----------|
| CWE-20 | Improper Input Validation | Input |
| CWE-22 | Path Traversal | Input |
| CWE-77 | Command Injection | Injection |
| CWE-78 | OS Command Injection | Injection |
| CWE-79 | Cross-site Scripting (XSS) | Injection |
| CWE-89 | SQL Injection | Injection |
| CWE-94 | Code Injection | Injection |
| CWE-116 | Improper Encoding or Escaping | Output |
| CWE-120 | Buffer Overflow | Memory |
| CWE-121 | Stack-based Buffer Overflow | Memory |
| CWE-122 | Heap-based Buffer Overflow | Memory |
| CWE-125 | Out-of-bounds Read | Memory |
| CWE-200 | Information Exposure | Disclosure |
| CWE-209 | Error Message Information Exposure | Disclosure |
| CWE-250 | Unnecessary Privileges | AuthZ |
| CWE-269 | Improper Privilege Management | AuthZ |
| CWE-284 | Improper Access Control | AuthZ |
| CWE-287 | Improper Authentication | AuthN |
| CWE-288 | Authentication Bypass via Alternate Path | AuthN |
| CWE-306 | Missing Authentication for Critical Function | AuthN |
| CWE-307 | Excessive Auth Attempts | AuthN |
| CWE-311 | Missing Encryption | Crypto |
| CWE-319 | Cleartext Transmission | Crypto |
| CWE-323 | Reusing Nonce/IV | Crypto |
| CWE-327 | Broken Cryptographic Algorithm | Crypto |
| CWE-330 | Insufficient Randomness | Crypto |
| CWE-345 | Insufficient Data Authenticity Verification | Integrity |
| CWE-352 | Cross-Site Request Forgery | Web |
| CWE-362 | Race Condition | Concurrency |
| CWE-367 | TOCTOU Race Condition | Concurrency |
| CWE-400 | Uncontrolled Resource Consumption | DoS |
| CWE-416 | Use After Free | Memory |
| CWE-426 | Untrusted Search Path | Supply Chain |
| CWE-434 | Unrestricted File Upload | Input |
| CWE-436 | Interpretation Conflict | Input |
| CWE-470 | Unsafe Reflection | Injection |
| CWE-476 | NULL Pointer Dereference | Memory |
| CWE-489 | Active Debug Code | Config |
| CWE-502 | Unsafe Deserialization | Injection |
| CWE-521 | Weak Password Requirements | AuthN |
| CWE-522 | Insufficiently Protected Credentials | AuthN |
| CWE-532 | Log File Information Leak | Disclosure |
| CWE-601 | Open Redirect | Web |
| CWE-602 | Client-Side Enforcement of Server-Side Security | Logic |
| CWE-611 | XML External Entity (XXE) | Injection |
| CWE-613 | Insufficient Session Expiration | Session |
| CWE-614 | Sensitive Cookie Without Secure Flag | Web |
| CWE-636 | Not Failing Securely | Logic |
| CWE-639 | Authorization Bypass via User-Controlled Key | AuthZ |
| CWE-640 | Weak Password Recovery | AuthN |
| CWE-668 | Exposure of Resource to Wrong Sphere | AuthZ |
| CWE-681 | Incorrect Numeric Type Conversion | Logic |
| CWE-732 | Incorrect Permission Assignment | AuthZ |
| CWE-770 | Allocation Without Limits | DoS |
| CWE-778 | Insufficient Logging | Logging |
| CWE-787 | Out-of-bounds Write | Memory |
| CWE-798 | Hard-coded Credentials | AuthN |
| CWE-829 | Inclusion from Untrusted Source | Supply Chain |
| CWE-862 | Missing Authorization | AuthZ |
| CWE-863 | Incorrect Authorization | AuthZ |
| CWE-918 | Server-Side Request Forgery | Web |
| CWE-943 | Improper Neutralization in Data Query | Injection |

### 12.2: CVSS 4.0 Decision Tree

Use this decision tree during Phase 4 to compute the CVSS 4.0 Base Score vector string for each finding. Walk through each metric in order, answer the question, and record the value.

**Base Metric Group -- Exploitability Metrics:**

**AV (Attack Vector):**
- Can be exploited over the network (remotely, no adjacency required)? -> **N** (Network)
- Requires adjacent network (same LAN, Bluetooth, Wi-Fi)? -> **A** (Adjacent)
- Requires local system access (shell, logged-in user)? -> **L** (Local)
- Requires physical access to the hardware? -> **P** (Physical)

**AC (Attack Complexity):**
- Requires race conditions, specialized configurations, or conditions beyond attacker control? -> **H** (High)
- Straightforward exploitation with no special conditions? -> **L** (Low)

**AT (Attack Requirements):**
- No special deployment or configuration conditions needed? -> **N** (None)
- Requires specific deployment topology, configuration, or prerequisite conditions? -> **P** (Present)

**PR (Privileges Required):**
- No authentication or authorization needed? -> **N** (None)
- Requires low-level privileges (normal user account)? -> **L** (Low)
- Requires admin/root/high-level privileges? -> **H** (High)

**UI (User Interaction):**
- No user interaction required? -> **N** (None)
- Passive interaction (user visits a page, receives a message)? -> **P** (Passive)
- Active interaction (user must click, type, approve, or perform specific action)? -> **A** (Active)

**Base Metric Group -- Vulnerable System Impact:**

**VC (Confidentiality Impact to Vulnerable System):**
- No confidentiality impact? -> **N** (None)
- Partial information disclosure (some data exposed)? -> **L** (Low)
- Total confidentiality loss (all data accessible)? -> **H** (High)

**VI (Integrity Impact to Vulnerable System):**
- No integrity impact? -> **N** (None)
- Partial data modification possible? -> **L** (Low)
- Total integrity loss (all data modifiable)? -> **H** (High)

**VA (Availability Impact to Vulnerable System):**
- No availability impact? -> **N** (None)
- Partial degradation of performance/service? -> **L** (Low)
- Total denial of service or system crash? -> **H** (High)

**Base Metric Group -- Subsequent System Impact:**

**SC (Confidentiality Impact to Subsequent Systems):**
- No confidentiality impact on other systems? -> **N** (None)
- Partial information disclosure in downstream/related systems? -> **L** (Low)
- Total confidentiality loss in downstream/related systems? -> **H** (High)

**SI (Integrity Impact to Subsequent Systems):**
- No integrity impact on other systems? -> **N** (None)
- Partial data modification in downstream/related systems? -> **L** (Low)
- Total integrity loss in downstream/related systems? -> **H** (High)

**SA (Availability Impact to Subsequent Systems):**
- No availability impact on other systems? -> **N** (None)
- Partial degradation in downstream/related systems? -> **L** (Low)
- Total denial of service in downstream/related systems? -> **H** (High)

**Vector String Format:**
```
CVSS:4.0/AV:{AV}/AC:{AC}/AT:{AT}/PR:{PR}/UI:{UI}/VC:{VC}/VI:{VI}/VA:{VA}/SC:{SC}/SI:{SI}/SA:{SA}
```

**Scoring Band Cross-Check (sanity validation):**

| Score Range | Severity | Typical Profile |
|-------------|----------|-----------------|
| 9.0 - 10.0 | Critical | Remote + No Auth + No Interaction + Full Impact (e.g., AV:N/AC:L/AT:N/PR:N/UI:N/VC:H/VI:H/VA:H) |
| 7.0 - 8.9 | High | Remote + Low Auth or Passive Interaction + Significant Impact |
| 4.0 - 6.9 | Medium | Auth Required + Partial Impact, or Local + Full Impact |
| 0.1 - 3.9 | Low | High Privilege or Physical Access + Limited Impact |

> **Sanity rule:** If a finding scores Critical but requires admin access or physical proximity, re-evaluate. If a finding scores Low but enables remote unauthenticated data theft, re-evaluate. The profile must match the score band.

### 12.3: OWASP Top 10:2025 Mapping

Map each finding to the applicable OWASP Web Top 10 (2025 edition) category during Phase 4.

| ID | Category | Common CWEs |
|----|----------|-------------|
| A01 | Broken Access Control | CWE-22, CWE-250, CWE-269, CWE-284, CWE-639, CWE-668, CWE-732, CWE-862, CWE-863, CWE-918 |
| A02 | Security Misconfiguration | CWE-16, CWE-209, CWE-434, CWE-489, CWE-732, CWE-1004, CWE-1032 |
| A03 | Software Supply Chain Failures | CWE-426, CWE-502, CWE-829, CWE-937, CWE-1035, CWE-1104 |
| A04 | Cryptographic Failures | CWE-311, CWE-319, CWE-323, CWE-327, CWE-330, CWE-522, CWE-798 |
| A05 | Injection | CWE-20, CWE-77, CWE-78, CWE-79, CWE-89, CWE-94, CWE-116, CWE-470, CWE-502, CWE-611, CWE-943 |
| A06 | Insecure Design | CWE-256, CWE-284, CWE-306, CWE-352, CWE-602, CWE-636 |
| A07 | Authentication Failures | CWE-287, CWE-288, CWE-306, CWE-307, CWE-521, CWE-522, CWE-613, CWE-640, CWE-798 |
| A08 | Software or Data Integrity Failures | CWE-345, CWE-426, CWE-502, CWE-829 |
| A09 | Security Logging and Alerting Failures | CWE-117, CWE-223, CWE-532, CWE-778 |
| A10 | Mishandling of Exceptional Conditions | CWE-248, CWE-252, CWE-390, CWE-391, CWE-395, CWE-476, CWE-754, CWE-755 |

### 12.4: OWASP Top 10 LLM:2025 Mapping

Map AI/LLM-related findings to the OWASP Top 10 for LLM Applications (2025 edition).

| ID | Category | Description |
|----|----------|-------------|
| LLM01 | Prompt Injection | Direct or indirect manipulation of LLM via crafted input |
| LLM02 | Sensitive Information Disclosure | LLM reveals private data, PII, or proprietary info in responses |
| LLM03 | Supply Chain | Compromised training data, models, plugins, or dependencies |
| LLM04 | Data and Model Poisoning | Tampered training data or fine-tuning that alters model behavior |
| LLM05 | Improper Output Handling | Unvalidated LLM output used in downstream operations |
| LLM06 | Excessive Agency | LLM granted excessive permissions, functions, or autonomy |
| LLM07 | System Prompt Leakage | Extraction of system prompts revealing internal logic or secrets |
| LLM08 | Vector and Embedding Weaknesses | Manipulation of RAG embeddings, vector DBs, or retrieval pipelines |
| LLM09 | Misinformation | LLM generates false information relied upon for decisions |
| LLM10 | Unbounded Consumption | Resource exhaustion via token abuse, repeated queries, or context flooding |

### 12.5: OWASP Top 10 Agentic:2026 Mapping

Map AI agent and MCP-related findings to the OWASP Agentic Security Initiatives (2026 edition).

| ID | Category | Description |
|----|----------|-------------|
| ASI01 | Agent Goal Hijack | Manipulating agent goals, plans, or decision paths via direct or indirect instruction injection |
| ASI02 | Tool Misuse and Exploitation | Agents misusing tools through unsafe composition, recursion, or excessive execution |
| ASI03 | Agent Identity and Privilege Abuse | Delegated authority, ambiguous identity, or trust assumptions leading to unauthorized actions |
| ASI04 | Agentic Supply Chain Compromise | Compromise of external agents, tools, schemas, or prompts dynamically trusted at runtime |
| ASI05 | Unexpected Code Execution | Agent-generated or agent-triggered code executing without validation or isolation |
| ASI06 | Memory and Context Poisoning | Injection or leakage of agent memory or context influencing future reasoning across sessions |
| ASI07 | Insecure Inter-Agent Communication | Manipulation of inter-agent messages including interception, injection, and spoofing |
| ASI08 | Cascading Agent Failures | Small agent failures propagating through connected systems causing large-scale impact |
| ASI09 | Human-Agent Trust Exploitation | Exploiting human over-reliance on agents through misleading explanations or false authority |
| ASI10 | Rogue Agents | Agents acting beyond intended objectives due to goal drift, collusion, or runaway autonomy |

### 12.6: NIST 800-53 Control Reference

Map findings to the most relevant NIST 800-53 Rev 5 controls during Phase 4 compliance enrichment.

| Control | Name | Assign When |
|---------|------|-------------|
| AC-3 | Access Enforcement | Finding involves access control bypass, BOLA/IDOR, authorization failures |
| AC-6 | Least Privilege | Finding involves excessive permissions, privilege escalation, unnecessary access |
| AU-2 | Event Logging | Finding involves missing audit events, incomplete logging, log gaps |
| CM-3 | Configuration Change Control | Finding involves configuration drift, insecure defaults, missing change tracking |
| IA-5 | Authenticator Management | Finding involves weak credentials, credential storage, password policy |
| RA-5 | Vulnerability Monitoring and Scanning | Finding discovered by automated scanning, dependency audit, known CVE |
| SA-11 | Developer Testing and Evaluation | Finding relates to insufficient security testing, missing SAST/DAST coverage |
| SC-8 | Transmission Confidentiality and Integrity | Finding involves cleartext transmission, missing TLS, transport security |
| SC-13 | Cryptographic Protection | Finding involves weak algorithms, key management, crypto implementation |
| SI-2 | Flaw Remediation | Finding involves unpatched vulnerability, outdated dependency, known defect |
| SI-7 | Software, Firmware, and Information Integrity | Finding involves code integrity, supply chain, tampered artifacts |
| SR-3 | Supply Chain Controls and Processes | Finding involves dependency risk, package provenance, supply chain attack vectors |

### 12.7: MITRE ATT&CK Technique Reference

Map findings to the most code-audit-relevant ATT&CK techniques during Phase 4.

| ID | Technique | Code-Level Indicator |
|----|-----------|---------------------|
| T1059 | Command and Scripting Interpreter | Unsanitized input passed to shell, eval(), exec(), child_process |
| T1068 | Exploitation for Privilege Escalation | Privilege boundary bypass, role escalation, admin access via user input |
| T1078 | Valid Accounts | Hardcoded credentials, default passwords, credential leakage in code |
| T1110 | Brute Force | Missing rate limiting on auth endpoints, no account lockout |
| T1190 | Exploit Public-Facing Application | Input validation failures in web endpoints, injection vectors |
| T1210 | Exploitation of Remote Services | SSRF, service-to-service auth bypass, internal API exposure |
| T1485 | Data Destruction | Unprotected delete operations, missing soft-delete, cascade deletes without authorization |
| T1499 | Endpoint Denial of Service | ReDoS, resource exhaustion, allocation without limits |
| T1525 | Implant Internal Image | Compromised build pipeline, tampered container images, CI/CD injection |
| T1530 | Data from Cloud Storage Object | Exposed S3 buckets, public blob storage, missing storage ACLs |
| T1552 | Unsecured Credentials | Secrets in source code, .env in repo, credentials in logs or error messages |
| T1565 | Data Manipulation | Unsigned data, missing integrity checks, tampered API responses |

### 12.8: MITRE ATLAS Reference

Map AI/ML-specific findings to MITRE ATLAS (Adversarial Threat Landscape for AI Systems) techniques.

| ID | Technique | Description |
|----|-----------|-------------|
| AML.T0015 | Evade ML Model | Crafting inputs that cause model misclassification or bypass |
| AML.T0018 | Backdoor ML Model | Inserting hidden triggers in model that activate on specific inputs |
| AML.T0020 | Poison Training Data | Contaminating training data to alter model behavior |
| AML.T0024 | Exfiltration via ML Inference API | Extracting training data or model details through query patterns |
| AML.T0025 | Exfiltration via Cyber Means | Using ML system access to exfiltrate data through side channels |
| AML.T0031 | Erode ML Model Integrity | Gradually degrading model performance through adversarial feedback |
| AML.T0043 | Craft Adversarial Data | Creating specifically crafted inputs to manipulate model outputs |
| AML.T0051 | LLM Prompt Injection | Direct or indirect prompt injection to override system instructions |
| AML.T0058 | AI Agent Context Poisoning | Manipulating agent LLM context to influence actions |
| AML.T0059 | Activation Triggers | Triggering hidden behaviors in AI agents via pre-planted conditions |
| AML.T0061 | AI Agent Tools | Abusing agent tool-calling capabilities for unintended actions |
| AML.T0062 | Exfiltration via AI Agent Tool Invocation | Data theft through agent tool calls |
| AML.T0096 | AI Service API | Exploiting AI service APIs as attack vectors |
| AML.T0098 | AI Agent Tool Credential Harvesting | Stealing credentials via agent tool interfaces |
| AML.T0099 | AI Agent Tool Data Poisoning | Poisoning data sources where agents invoke tools |
| AML.T0101 | Data Destruction via AI Agent Tool Invocation | Using agent tools to destroy data |

---

## Section 14: Detection Heuristics

> **Note for orchestrator:** Evaluate these heuristics during Phase 1 (Step 1.1) to determine which conditional squads to activate. A squad activates if ANY trigger in its set matches. Use Glob and Grep tools to check patterns. Squads 1-5, 22, and Red Team are always active and do not need heuristic checks.

### Squad 6-7: Web Security (Injection & XSS + Headers, CORS & Transport)

**File triggers:**
- `**/*.html`
- `**/*.htm`
- `**/*.jsx`
- `**/*.tsx`
- `**/*.vue`
- `**/*.svelte`
- `**/*.astro`
- `**/*.ejs`
- `**/*.hbs`
- `**/*.pug`

**Package triggers (in package.json, package-lock.json, or equivalent manifest dependencies):**
- `react`
- `react-dom`
- `next`
- `vue`
- `nuxt`
- `angular`
- `@angular/core`
- `svelte`
- `@sveltejs/kit`
- `astro`
- `solid-js`
- `preact`
- `lit`
- `htmx.org`

**Import triggers (indicates server-side rendering / web server):**
- `express`
- `fastify`
- `hono`
- `koa`
- `hapi`
- `@hapi/hapi`
- `nestjs`
- `@nestjs/core`
- `django`
- `flask`
- `fastapi`
- `gin`
- `fiber`
- `actix-web`
- `axum`
- `rocket`
- `rails`
- `sinatra`
- `phoenix`

**Activation rule:** ANY file trigger OR ANY package trigger OR ANY import trigger -> activate Squads 6 AND 7.

---

### Squad 8: WebSocket, GraphQL & gRPC

**Code/import triggers (search source files for these patterns):**
- `ws://`
- `wss://`
- `new WebSocket`
- `socket.io`
- `ws` (as package import)
- `graphql`
- `@apollo/server`
- `@apollo/client`
- `apollo-server`
- `type-graphql`
- `graphql-yoga`
- `mercurius`
- `@grpc/grpc-js`
- `@grpc/proto-loader`
- `google.protobuf`
- `protobufjs`
- `grpcio`
- `tonic` (Rust gRPC)

**File triggers:**
- `**/*.graphql`
- `**/*.gql`
- `**/*.proto`

**Activation rule:** ANY code/import trigger OR ANY file trigger -> activate Squad 8.

---

### Squad 9: API Security (REST & Endpoints)

**Directory triggers:**
- `**/api/` directory exists
- `**/routes/` directory exists
- `**/controllers/` directory exists
- `**/endpoints/` directory exists

**File triggers:**
- `**/openapi.yaml`
- `**/openapi.json`
- `**/swagger.yaml`
- `**/swagger.json`
- `**/*.openapi.yml`

**Code pattern triggers (search source files):**
- `app.get(`, `app.post(`, `app.put(`, `app.delete(`, `app.patch(` (Express/Fastify route patterns)
- `@Get(`, `@Post(`, `@Put(`, `@Delete(` (NestJS/decorator-based route patterns)
- `@app.route`, `@app.get`, `@app.post` (Flask/FastAPI route patterns)
- `router.get`, `router.post`, `router.put`, `router.delete` (generic router patterns)
- `path(`, `re_path(` (Django URL patterns)
- `Route{`, `HandleFunc(` (Go route patterns)
- `#[get(`, `#[post(` (Rust Actix/Rocket route patterns)

**Activation rule:** ANY directory trigger OR ANY file trigger OR 2+ code pattern matches -> activate Squad 9.

---

### Squad 10: OAuth, JWT & Sessions

**Package/import triggers:**
- `jsonwebtoken`
- `jose`
- `@auth/core`
- `next-auth`
- `@clerk/nextjs`
- `@clerk/clerk-sdk-node`
- `auth0`
- `@auth0/nextjs-auth0`
- `passport`
- `passport-jwt`
- `passport-oauth2`
- `oauth`
- `oauth2orize`
- `openid-client`
- `oidc-provider`
- `express-session`
- `express-jwt`
- `PyJWT`
- `python-jose`
- `authlib`
- `django-oauth-toolkit`
- `oauthlib`
- `golang-jwt`
- `actix-identity`

**Code pattern triggers:**
- `jwt.sign`, `jwt.verify`, `jwt.decode`
- `JWT`, `Bearer` in auth header references
- `access_token`, `refresh_token`, `id_token`
- `session.`, `req.session`, `request.session`
- `passport.authenticate`
- `OAuth2Client`, `OAuth2Strategy`

**Activation rule:** ANY package/import trigger OR 2+ code pattern matches -> activate Squad 10.

---

### Squad 11: Auth & Identity

**Code pattern triggers (search source files):**
- `login`, `signin`, `sign_in`, `signIn`
- `signup`, `sign_up`, `signUp`, `register`, `registration`
- `logout`, `signout`, `sign_out`, `signOut`
- `password`, `passwd`, `credential`
- `authenticate`, `authorization`, `isAuthenticated`, `isAuthorized`
- `requireAuth`, `withAuth`, `authMiddleware`, `auth_required`
- `role`, `permission`, `privilege`, `rbac`, `acl`
- `forgot_password`, `reset_password`, `change_password`
- `mfa`, `two_factor`, `2fa`, `totp`, `otp`

**File pattern triggers:**
- `**/auth/**`
- `**/login/**`
- `**/users/**` or `**/user/**`
- `**/accounts/**` or `**/account/**`
- `**/identity/**`

**Model/schema triggers (look for user/account model definitions):**
- `User` model or schema with `password`, `email`, `role` fields
- `Account` model or schema with authentication fields
- Database tables named `users`, `accounts`, `sessions`, `roles`

**Activation rule:** ANY file pattern trigger with auth-related content OR 3+ code pattern matches -> activate Squad 11.

---

### Squad 12: Payments & Financial Operations

**Package/import triggers:**
- `stripe`
- `@stripe/stripe-js`
- `@stripe/react-stripe-js`
- `@solana/web3.js`
- `@solana/spl-token`
- `ethers`
- `web3`
- `web3.js`
- `@paypal/checkout-server-sdk`
- `braintree`
- `square`
- `adyen`
- `@adyen/api-library`
- `razorpay`
- `paddle`
- `lemonSqueezy`

**Code pattern triggers:**
- `payment`, `checkout`, `purchase`, `transaction`
- `price`, `amount`, `total`, `subtotal`, `fee`, `commission`
- `refund`, `chargeback`, `dispute`
- `subscription`, `recurring`, `billing`
- `coupon`, `discount`, `promo`, `voucher`
- `wallet`, `balance`, `transfer`, `withdraw`, `deposit`
- `invoice`, `receipt`, `order`
- `lamports`, `sol`, `wei`, `gwei`, `ether`

**Activation rule:** ANY package/import trigger OR 3+ code pattern matches in financial context -> activate Squad 12.

---

### Squad 13: Database & Data

**Package/import triggers:**
- `prisma`, `@prisma/client`
- `sequelize`
- `mongoose`, `mongodb`
- `pg`, `postgres`, `postgresql`
- `mysql`, `mysql2`
- `sqlite3`, `better-sqlite3`
- `redis`, `ioredis`
- `drizzle-orm`
- `typeorm`
- `knex`
- `objection`
- `mikro-orm`
- `sqlalchemy`
- `django.db`
- `peewee`
- `tortoise-orm`
- `diesel` (Rust)
- `sqlx` (Rust/Go)
- `gorm` (Go)
- `ent` (Go)

**File triggers:**
- `**/prisma/schema.prisma`
- `**/migrations/**`
- `**/seeds/**` or `**/seeders/**`
- `**/*.sql`
- `**/models/**` (with ORM imports)
- `**/drizzle.config.*`

**Code pattern triggers:**
- `SELECT`, `INSERT`, `UPDATE`, `DELETE` (raw SQL in string literals)
- `query(`, `execute(`, `raw(` (raw query execution)
- `createConnection`, `createPool`, `getConnection`

**Activation rule:** ANY package/import trigger OR ANY file trigger with DB content OR raw SQL patterns detected -> activate Squad 13.

---

### Squad 14: AI/LLM Security

**Package/import triggers:**
- `openai`
- `@anthropic-ai/sdk`, `anthropic`
- `langchain`, `@langchain/core`
- `llamaindex`, `llama-index`
- `@ai-sdk/openai`, `@ai-sdk/anthropic`, `ai` (Vercel AI SDK)
- `ollama`
- `cohere`
- `@huggingface/inference`
- `replicate`
- `together-ai`
- `groq-sdk`
- `@mistralai/mistralai`
- `google-generativeai`, `@google/generative-ai`
- `transformers` (Python)
- `torch`, `tensorflow`, `keras`

**Code pattern triggers:**
- `chat.completions`, `createCompletion`, `createChatCompletion`
- `system_prompt`, `system_message`, `systemPrompt`
- `embedding`, `embeddings`, `vectorStore`, `vector_store`
- `rag`, `retrieval`, `chunking`
- `fine_tune`, `finetune`, `training`
- `model.generate`, `model.predict`, `model.invoke`

**File triggers:**
- `**/prompts/**`
- `**/agents/**`
- `**/*.prompt`
- `**/models/**` (with ML/AI imports)

**Activation rule:** ANY package/import trigger OR 2+ code pattern matches -> activate Squad 14.

---

### Squad 15: Single-Agent & MCP Exploitation

**Package/import triggers:**
- `@modelcontextprotocol/sdk`
- `mcp` (as package)
- `claude-code`
- `@anthropic-ai/claude-code`
- `autogen`
- `crewai`
- `langchain/agents`
- `langchain.agents`
- `@langchain/langgraph`
- `semantic-kernel`
- `agent-protocol`

**Directory/file triggers:**
- `.claude/` directory exists
- `**/.claude/settings.json`
- `**/mcp.json`
- `**/mcp-config.*`
- `**/tools/**` (with function_calling or tool_use patterns)
- `**/*agent*config*`
- `**/*skill*` files

**Code pattern triggers:**
- `tool_use`, `function_calling`, `function_call`
- `tools:`, `tool_choice`, `toolChoice`
- `MCP`, `MCPServer`, `MCPClient`
- `create_tool`, `register_tool`, `tool_definition`
- `agent.run`, `agent.execute`, `agent.invoke`
- `human_in_the_loop`, `human_approval`, `require_approval`
- `memory.save`, `memory.load`, `conversationHistory`

**Activation rule:** ANY package/import trigger OR ANY directory/file trigger OR 2+ code pattern matches -> activate Squad 15.

---

### Squad 23: Multi-Agent, Agentic Infrastructure & NHI Security

**Activation rule:** Same triggers as Squad 15 (ANY match activates). Additionally activated by multi-agent orchestration patterns or NHI/service account patterns.

**Additional triggers beyond Squad 15:**

**Code pattern triggers (search across all source files):**
- `agent`, `multi_agent`, `multi-agent`, `orchestrat` (multi-agent orchestration)
- `a2a`, `agent_card`, `agent-to-agent` (A2A protocol)
- `service_account`, `serviceAccount`, `machine_identity` (NHI patterns)
- `delegation`, `delegate_to`, `chain_of_trust` (delegation patterns)
- `consensus`, `voting`, `quorum` (multi-agent decision patterns)

**Activation rule:** If Squad 15 is activated, Squad 23 is ALWAYS co-activated. Squad 23 can also activate independently if multi-agent or NHI patterns are found without Squad 15 triggers.

---

### Squad 16: Blockchain/Web3

**Package/import triggers:**
- `@solana/web3.js`
- `@solana/spl-token`
- `@project-serum/anchor`, `@coral-xyz/anchor`
- `ethers`
- `web3`, `web3.js`
- `hardhat`
- `@openzeppelin/contracts`
- `foundry` (via foundry.toml)
- `truffle`
- `wagmi`
- `viem`
- `@thirdweb-dev/sdk`
- `@metaplex-foundation/js`
- `near-api-js`
- `cosmjs`

**File triggers:**
- `**/*.sol` (Solidity)
- `**/*.rs` in `programs/` directory (Anchor/Solana)
- `**/hardhat.config.*`
- `**/foundry.toml`
- `**/truffle-config.js`
- `**/Anchor.toml`
- `**/Move.toml`

**Code pattern triggers:**
- `PublicKey`, `Keypair`, `Transaction`, `SystemProgram` (Solana)
- `ethers.Contract`, `ethers.Provider`, `ethers.Wallet`
- `web3.eth`, `web3.utils`
- `msg.sender`, `msg.value`, `require(`, `modifier` (Solidity)
- `#[program]`, `#[account]` (Anchor)

**Activation rule:** ANY package/import trigger OR ANY file trigger OR 3+ code pattern matches -> activate Squad 16.

---

### Squad 17: Cloud, Container & Serverless

**File triggers:**
- `**/Dockerfile*`
- `**/docker-compose*.yml`, `**/docker-compose*.yaml`
- `**/*.tf`, `**/*.tfvars` (Terraform)
- `**/serverless.yml`, `**/serverless.yaml`, `**/serverless.ts`
- `**/cloudformation*.yml`, `**/cloudformation*.yaml`, `**/*.template.json`
- `**/k8s/**`, `**/kubernetes/**`
- `**/*.k8s.yml`, `**/*.k8s.yaml`
- `**/helm/**`, `**/Chart.yaml`
- `**/.github/workflows/*.yml` (CI/CD pipeline)
- `**/.gitlab-ci.yml`
- `**/Pulumi.yaml`
- `**/cdk.json` (AWS CDK)

**Package/import triggers:**
- `aws-sdk`, `@aws-sdk/*`
- `@google-cloud/*`, `google-cloud-*`
- `@azure/*`, `azure-*`
- `serverless`
- `@pulumi/*`
- `aws-cdk-lib`

**Code pattern triggers:**
- `AWS.S3`, `AWS.Lambda`, `AWS.DynamoDB`, `AWS.IAM`
- `s3://`, `gs://`, `az://`
- `arn:aws:`, `projects/*/locations/*/`
- `IMDS`, `169.254.169.254` (metadata service)
- `kubectl`, `helm`, `docker`

**Activation rule:** ANY file trigger OR ANY package/import trigger -> activate Squad 17.

---

### Squad 18: Cryptography & Encryption

**Package/import triggers:**
- `crypto` (Node.js built-in)
- `bcrypt`, `bcryptjs`
- `argon2`
- `scrypt`
- `node-forge`
- `tweetnacl`
- `libsodium`, `sodium-native`
- `@noble/hashes`, `@noble/ciphers`, `@noble/curves`
- `openssl` (via bindings)
- `cryptography` (Python)
- `pycryptodome`, `pycryptodomex`
- `hashlib` (Python)
- `ring` (Rust)
- `rustls` (Rust)
- `boring` (Go BoringCrypto)
- `golang.org/x/crypto`

**Code pattern triggers:**
- `encrypt`, `decrypt`, `cipher`, `decipher`
- `createHash`, `createHmac`, `createSign`, `createVerify`
- `AES`, `RSA`, `ECDSA`, `Ed25519`, `ChaCha20`
- `SHA-256`, `SHA-384`, `SHA-512`, `MD5`, `SHA-1` (especially MD5/SHA-1 as concerns)
- `pbkdf2`, `scrypt`, `bcrypt.hash`, `argon2.hash`
- `generateKey`, `generateKeyPair`, `randomBytes`
- `tls`, `ssl`, `certificate`, `cert`, `x509`
- `nonce`, `iv`, `salt`, `pepper`

**Activation rule:** ANY package/import trigger OR 3+ code pattern matches -> activate Squad 18.

---

### Squad 19: Privacy & Compliance

**Code pattern triggers (data model fields and PII indicators):**
- `email`, `phone`, `address`, `ssn`, `social_security`
- `date_of_birth`, `dob`, `birthdate`
- `first_name`, `last_name`, `full_name`
- `credit_card`, `card_number`, `cvv`, `expiry`
- `ip_address`, `ipAddress`, `user_agent`, `userAgent`
- `geolocation`, `latitude`, `longitude`, `location`
- `biometric`, `fingerprint`, `face_id`
- `medical`, `health`, `diagnosis`, `prescription`
- `passport`, `driver_license`, `national_id`

**Keyword triggers (in source files, configs, or documentation):**
- `gdpr`, `GDPR`
- `ccpa`, `CCPA`
- `hipaa`, `HIPAA`
- `coppa`, `COPPA`
- `consent`, `data_processing_agreement`, `dpa`
- `right_to_delete`, `right_to_erasure`, `data_subject_request`
- `privacy_policy`, `cookie_consent`, `cookie_banner`
- `data_retention`, `retention_policy`
- `anonymize`, `pseudonymize`, `redact`

**File triggers:**
- `**/privacy/**`
- `**/consent/**`
- `**/gdpr/**`
- `**/compliance/**`

**Activation rule:** 5+ PII field patterns detected across data models OR ANY keyword trigger OR ANY file trigger -> activate Squad 19.

---

### Squad 20: Memory Safety

**File triggers:**
- `**/*.c`
- `**/*.cpp`, `**/*.cc`, `**/*.cxx`
- `**/*.h`, `**/*.hpp`, `**/*.hxx`
- `**/*.rs` (Rust -- focus on `unsafe` blocks)
- `**/*.wasm`
- `**/*.wat`
- `**/*.zig`

**Build system triggers:**
- `**/CMakeLists.txt`
- `**/Makefile`, `**/makefile`, `**/GNUmakefile`
- `**/Cargo.toml` (Rust -- activate for `unsafe` audit)
- `**/meson.build`
- `**/BUILD`, `**/BUILD.bazel` (Bazel with C/C++)
- `**/*.vcxproj` (Visual Studio C++)
- `**/build.zig`

**Code pattern triggers (for Rust projects, only if unsafe is found):**
- `unsafe {`, `unsafe fn`, `unsafe impl`
- `malloc`, `calloc`, `realloc`, `free` (C/C++)
- `memcpy`, `memset`, `memmove`, `strcpy`, `strcat`, `sprintf` (C/C++)
- `new[]`, `delete[]` (C++)
- `raw pointer`, `*const`, `*mut` (Rust raw pointers)

**Activation rule:** ANY C/C++/Zig file trigger OR (Rust file trigger AND `unsafe` pattern found) OR WASM file trigger -> activate Squad 20.

---

### Squad 21: Deserialization & Type Safety

**Code pattern triggers (search across all source files):**
- `serialize`, `deserialize`, `Serialize`, `Deserialize`
- `marshal`, `unmarshal`, `Marshal`, `Unmarshal`
- `pickle.load`, `pickle.loads`, `cPickle` (Python)
- `yaml.load`, `yaml.unsafe_load` (Python -- safe: `yaml.safe_load`)
- `ObjectInputStream`, `readObject` (Java)
- `BinaryFormatter`, `XmlSerializer` (C#/.NET)
- `unserialize`, `__wakeup`, `__destruct` (PHP)
- `Marshal.load`, `YAML.load` (Ruby)
- `JSON.parse` with reviver functions
- `eval(`, `Function(` on serialized data
- `xml.etree`, `lxml.etree`, `xml.dom`, `xml.sax` (XML parsers)
- `DOMParser`, `XMLHttpRequest` with XML
- `protobuf`, `msgpack`, `MessagePack`
- `avro`, `thrift`

**File triggers:**
- `**/*.xml` (with parsing logic)
- `**/*.xslt`, `**/*.xsl`
- `**/*.proto` (protobuf definitions)
- `**/*.avsc` (Avro schemas)
- `**/*.thrift`

**Activation rule:** ANY code pattern trigger detected in source files OR ANY file trigger with associated parsing code -> activate Squad 21.

---

### Detection Heuristic Summary Table

Quick reference for Phase 1 squad activation:

| Squad | Primary Signal | Secondary Signal | Threshold |
|-------|---------------|-----------------|-----------|
| 6-7 | .html/.jsx/.tsx/.vue/.svelte files | Web framework in manifest | ANY match |
| 8 | .graphql/.proto files, ws:// in code | GraphQL/gRPC package imports | ANY match |
| 9 | api/routes/ directories | Route definition patterns | ANY dir OR 2+ patterns |
| 10 | JWT/OAuth packages | Token/session code patterns | ANY pkg OR 2+ patterns |
| 11 | auth/ directory with auth content | login/signup/password patterns | ANY dir+content OR 3+ patterns |
| 12 | Payment SDK packages | Payment/financial code terms | ANY pkg OR 3+ patterns |
| 13 | ORM/DB packages, .sql files | Raw SQL in string literals | ANY match |
| 14 | AI/ML SDK packages | Prompt/embedding patterns | ANY pkg OR 2+ patterns |
| 15 | MCP packages, .claude/ directory | Tool/agent code patterns | ANY pkg/dir OR 2+ patterns |
| 16 | .sol files, Anchor.toml | Blockchain SDK imports | ANY file OR 3+ patterns |
| 17 | Dockerfile, *.tf, k8s manifests | Cloud SDK imports | ANY match |
| 18 | Crypto packages (bcrypt, argon2) | encrypt/decrypt/hash patterns | ANY pkg OR 3+ patterns |
| 19 | PII fields in data models | GDPR/CCPA/HIPAA keywords | 5+ PII OR ANY keyword |
| 20 | .c/.cpp/.h/.rs/.wasm files | C toolchain (CMake, Make) | ANY C/C++ OR Rust+unsafe |
| 21 | serialize/deserialize patterns | XML parser imports, pickle | ANY pattern match |
| 23 | MCP/agent triggers (same as 15) | Multi-agent, A2A, NHI patterns | Co-activates with Squad 15 |

> **Note for orchestrator:** After evaluating all heuristics, merge related squads if total count exceeds the cap (8 full / 5 quick). Priority order for merging: combine Squads 6+7 (already related), combine 9+10 (API+auth), combine 14+15+23 (AI+agent+multi-agent). Always preserve the most relevant squad's full persona list and add key personas from the merged squad.
