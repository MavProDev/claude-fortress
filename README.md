# Fortress

446 attack personas. 25 squads. 9 phases. Every finding requires proof-of-exploit. Mapped to CWE, CVSS 4.0, AIVSS, OWASP, NIST, and MITRE ATT&CK/ATLAS. If it has a vulnerability, Fortress finds it.

A Claude Code skill that runs the most comprehensive adversarial security audit framework available for any codebase.

## Install

```bash
npx claudesuite install fortress
```

Or manually:

```bash
curl -sL https://raw.githubusercontent.com/MavProDev/claude-fortress/main/SKILL.md \
  --create-dirs -o ~/.claude/skills/fortress/SKILL.md
```

## Usage

In Claude Code:

```
/fortress
/fortress quick
/fortress focused auth
```

| Mode | What It Does |
|------|--------------|
| `/fortress` | Full 9-phase audit with all artifacts |
| `/fortress quick` | Phases 0-4 only — no fixes, fast assessment |
| `/fortress focused <domain>` | Full audit but only squads relevant to the domain (e.g. `auth`, `payments`, `ai`) |

## The 9 Phases

1. **RECON** — Auto-detects project stack, frameworks, languages, and attack surface
2. **SQUAD ASSEMBLY** — Selects from 446 attack personas across 25 domains
3. **ADVERSARIAL ASSAULT** — Squads probe the codebase, every finding requires proof-of-exploit
4. **VALIDATION** — Filters false positives. If it can't be reproduced, it doesn't ship
5. **REPORT** — Maps findings to CWE, CVSS 4.0, AIVSS, OWASP (Web + LLM + Agentic), NIST 800-53, DISA STIG, MITRE ATT&CK/ATLAS
6. **PROPOSE & APPROVE** — Presents fixes for your approval. Never auto-fixes
7. **EXECUTE** — Applies approved fixes only
8. **INTEGRATION VERIFY** — Re-runs affected tests, confirms fixes hold
9. **DEBRIEF** — Identifies systemic patterns across findings

## Standards Mapping

Every finding maps to:
- CWE classification
- CVSS 4.0 score (estimated)
- AIVSS score (estimated, OWASP v0.1 draft methodology — applied to agentic findings only: tool-calling agents, MCP servers, autonomous loops, multi-agent orchestration)
- OWASP Web 2025 / LLM 2025 / Agentic 2026
- NIST 800-53 controls
- NIST SSDF practices
- DISA STIG severity (CAT I/II/III)
- MITRE ATT&CK / MITRE ATLAS techniques

## Output: 10-Artifact Evidence Suite

Each audit produces:
1. Executive summary
2. Detailed markdown report
3. SARIF v2.1.0 file
4. CycloneDX SBOM
5. Compliance posture summary
6. POA&M template
7. Public security page
8. Delta report
9. Security posture snapshot
10. Execution log

## Part of ClaudeSuite

This skill is part of [ClaudeSuite](https://claudesuite.xyz) — a curated collection of open-source Claude Code skills.

## License

MIT
