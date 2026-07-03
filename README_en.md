# Swarm

> A CodeX skill for parallel agent orchestration, built on two foundational pillars: First Principles Thinking and Adversarial Review.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE.txt)
[![CodeX Skill](https://img.shields.io/badge/CodeX-Skill-339cff)](https://github.com/openai/codex)

[中文](README.md)

---

## What It Does

`swarm` is a CodeX skill that automatically decomposes complex tasks into parallel sub-tasks with full agent lifecycle management. It's built on two "god-tier" prompting techniques:

| Pillar | Role | Core Logic |
|--------|------|------------|
| **First Principles** | Generation | Break analogical reasoning, re-derive from fundamental facts — fix root causes, not symptoms |
| **Adversarial Review** | Verification | Spawn multi-agent attack squads to break your system before users do |

Together they form a **complete closed loop**: First Principles → Root-Cause Decomposition → Parallel Implementation → Adversarial Review → Fix → Re-Verify.

## Trigger Conditions

The skill activates **automatically** when ≥2 of these are met:

- Task spans ≥3 files or ≥2 modules
- Both exploration and implementation needed
- ≥2 independent sub-tasks with no data dependency
- Multi-dimensional verification required
- You say "parallel", "concurrent", "simultaneously"
- You say "first principles", "from scratch", "root cause"
- You say "adversarial review", "attack test", "break it"

**Standalone trigger**: Saying "review", "audit", "check for bugs" triggers adversarial review mode directly.

## Features

### 1. Intelligent Task Decomposition
First-principles root-cause analysis before decomposition. Tasks are split by causal structure, not just file locations.

### 2. Parallel Agent Orchestration
Up to 12 concurrent agents (≤9 workers, ≤6 explorers). Automatic queuing when at capacity. Write-set isolation guarantees safety.

### 3. Adversarial Review
6 attack vector categories (temporal, data, concurrency, resource, state, security) with 40+ concrete triggers. Multi-agent attack → collect → fix → re-attack.

### 4. Full Lifecycle Management
Automatic agent lifecycle from creation to cleanup. Liveness-based stall detection (no fixed timeouts). Auto-recycle prevents resource leaks.

### 5. Periodic Audits
Every 2-4 weeks: global adversarial audit covering architecture, dependencies, code quality, and documentation consistency.

## Installation

```bash
git clone https://github.com/coder-wangbin/codex-swarm.git ~/.codex/skills/swarm
# Restart CodeX
```

## Quick Examples

**Multi-module feature**: "Add audit logging to both knowledge-base and permission modules" → 2 parallel workers + adversarial review → done.

**Bug fix**: "Fix the OpenAI scraper" → First principles finds routing layer design flaw → fixes root cause instead of patching symptom.

**Adversarial review**: "Review pkg/logic/qa.go" → 6 attack agents cover time, data, concurrency, resource, state, and security vectors → ranked finding report.

## Structure

```
swarm/
├── SKILL.md                    # Main skill file
├── README.md / README_en.md    # Documentation
├── LICENSE.txt                 # MIT
├── agents/openai.yaml          # UI metadata
├── assets/icon.svg             # Skill icon
└── references/
    ├── lifecycle.md            # Agent lifecycle state machine
    ├── patterns.md             # 9 decomposition patterns + anti-patterns
    └── adversarial-review.md   # Attack vector taxonomy + orchestration
```

## Design Philosophy

The skill itself was designed from first principles.

What is a skill fundamentally? Not adding features to a model — changing how a model **thinks**.

Most skills tell the model *what to do*. This skill tells the model *how to think*: first re-derive from fundamentals, then find every crack from the opposing side. These two mental habits, once internalized, elevate code quality dramatically — across any domain.

## License

MIT
