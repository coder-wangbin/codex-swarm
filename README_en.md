# Swarm

> A CodeX skill for parallel agent orchestration — First Principles × Adversarial Review × Anti-Cheat Guardrails.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE.txt)
[![CodeX Skill](https://img.shields.io/badge/CodeX-Skill-339cff)](https://github.com/openai/codex)

[中文](README.md)

---

## What It Is

`swarm` is a CodeX skill that automatically decomposes complex tasks into parallel sub-tasks with full agent lifecycle management. It stands on three pillars, incorporating structured task brief methodology from the [leader](https://github.com/KKKKhazix/khazix-skills/tree/main/leader) skill:

| Pillar | Role | Core Logic |
|--------|------|------------|
| **First Principles** | Generation | Break analogical reasoning, re-derive from fundamentals — fix root causes |
| **Adversarial Review** | Verification | Multi-agent attack squads find every crack before users do |
| **Anti-Cheat Guardrails** | Quality | Explicitly forbid skip-tests, relax-assertions, mock-core, `\|\| true` |

Closed loop: Root-cause analysis → Structured task briefs → Parallel implementation (anti-cheat) → Adversarial review → Reverse verification → Fix → Re-verify.

## Trigger Conditions

Activates **automatically** when ≥2 of these are met:

- ≥3 files or ≥2 modules
- Exploration + implementation needed
- ≥2 independent sub-tasks
- Multi-dimensional verification required
- You say "parallel", "concurrent", "simultaneously"
- You say "first principles", "root cause"
- You say "adversarial review", "attack test"
- You say "security audit", "auth review", "permission bug"
- You say "code review", "audit"

**Standalone triggers**: "review", "audit", "security audit", "有什么问题" activate adversarial review / security audit mode directly.

## Features

### 1. Task Type Classification
Executive (hard metrics) vs Exploratory (evidence goals) — different verification strategies per type.

### 2. Structured Agent Task Briefs
Every spawned agent receives a mission with: BOUNDARY, BASELINE, VERIFICATION, ANTI-CHEAT, FORBIDDEN actions, ADVERSARIAL SELF-CHECK, and PROGRESS clauses.

### 3. Anti-Cheat Guardrails
Each worker is explicitly forbidden from: skipping tests, relaxing assertions, mocking core logic, deleting tests, using `|| true`, or modifying test baselines. Violation = immediate failure.

### 4. Parallel Agent Orchestration
Up to 12 concurrent agents (≤9 workers, ≤6 explorers/attackers). Automatic queuing. Write-set isolation.

### 5. Adversarial Review + Reverse Verification
7 attack vector categories (including Go backend Auth). Post-fix reverse verification: intentionally trigger the bug → show RED → fix → show GREEN.

### 6. Security Audit (Auth/Permission)
Go backend specialized: middleware chain audit, role escalation, route ACL, JWT/Cookie security, RBAC consistency, DB-level permissions.

### 7. Full Lifecycle Management
Auto-create → monitor → collect → recycle. Liveness-based stall detection. PROGRESS.md for session-resume safety.

## Installation

```bash
git clone https://github.com/coder-wangbin/codex-swarm.git ~/.codex/skills/swarm
# Restart CodeX
```

## Quick Examples

**Multi-module feature**: "Add audit logging to both knowledge-base and permission modules" → baseline check → 2 parallel workers with anti-cheat → adversarial review → reverse verification → done.

**Bug fix**: "Fix the scraper" → First principles finds routing layer design flaw → fixes root cause → reverse verification proves the fix.

**Security audit**: "Audit auth module for permission bypass" → 4 parallel attackers (middleware, route ACL, role escalation, JWT) → ranked report → fix + reverse verify.

## Structure

```
swarm/
├── SKILL.md                    # Main skill file (three pillars + protocol)
├── README.md / README_en.md    # Documentation
├── LICENSE.txt                 # MIT
├── agents/openai.yaml          # UI metadata
├── assets/icon.svg             # Skill icon
└── references/
    ├── lifecycle.md            # Agent lifecycle state machine
    ├── patterns.md             # 11 decomposition patterns + anti-patterns
    ├── adversarial-review.md   # 7-category attack vector taxonomy
    ├── auth-review.md          # Go backend auth security audit
    └── task-brief.md           # Structured agent task brief template + anti-cheat catalog
```

## Design Philosophy

What is a skill fundamentally? Not adding features — changing how a model **thinks**. Most skills tell the model *what to do*. Swarm tells it *how to think*: re-derive from fundamentals, find every crack, and guard against its own shortcuts.

Incorporates structured task brief methodology from [KKKKhazix/leader](https://github.com/KKKKhazix/khazix-skills/tree/main/leader).

## License

MIT
