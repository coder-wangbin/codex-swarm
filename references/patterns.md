# Decomposition Patterns

## Pattern 1: Multi-Module Feature

**Task shape**: "Add feature X to modules A, B, and C"

```
First Principles: Is feature X the right abstraction for all modules?
                  If yes, what is the shared contract?

Decomposition:
├── Worker-1: module_a/feature.go
├── Worker-2: module_b/feature.go
└── Worker-3: module_c/feature.go
```

## Pattern 2: Explore-Then-Implement

**Task shape**: "Understand how X works, then add Y"

```
Phase 1 (parallel explorers):
├── Explorer-1: data flow through module A
├── Explorer-2: data flow through module B
└── Explorer-3: handler/router structure

Phase 2 (after collection):
├── Worker-1: implement Y in module A
└── Worker-2: implement Y in module B
```

## Pattern 3: Implement + Test (Dual-Track)

```
├── Worker-1: implement X in source files
├── Worker-2: write tests for X (separate test files)
└── Explorer-1: explore existing test patterns for style
```

## Pattern 4: Cross-Cutting Refactor

⚠️ NOT parallelizable when all files share the same signature.
Only parallelize when write sets are provably disjoint.

## Pattern 5: Multi-Repo / Multi-Service

```
Phase 1 (parallel, pre-agreed contract):
├── Worker-1: update API in service-A
└── Worker-2: update consumer in service-B
Phase 2: Main agent verifies integration
```

## Pattern 6: Build + Test + Lint (Verification Fan-Out)

```
├── Worker-1: go build ./...
├── Worker-2: go test ./pkg/...
├── Worker-3: go test ./internal/...
└── Worker-4: golangci-lint run
```

## Pattern 7: Adversarial Review (Attack-Then-Fix)

**Core idea**: After implementing ANY non-trivial feature, spawn attack agents before
declaring the work done. Attackers find bugs → workers fix root causes → re-attack.

```
Phase 0 — First Principles: What assumptions did we make? What invariants must hold?

Phase 1 — Attack Vector Generation (main agent):
  Brainstorm ≥5 edge cases per category:
  - Temporal: wrong/future/zero/overflow times
  - Data: empty, null, massive, malformed, nested, recursive
  - Concurrency: races, deadlocks, partial writes
  - Resource: memory, FDs, connections, disk
  - State: inconsistent cache, stale refs, interrupted workflows
  - Input: injection, encoding, boundary values, type confusion

Phase 2 — Parallel Attack (all read-only explorers):
├── Attacker-1: temporal edge cases
├── Attacker-2: data edge cases
├── Attacker-3: concurrency edge cases
├── Attacker-4: resource edge cases
├── Attacker-5: state edge cases
└── Attacker-6: input edge cases

Phase 3 — Collect & Categorize:
  CRITICAL: data loss, crash, security breach
  HIGH: incorrect behavior, wrong output
  MEDIUM: edge case, degraded UX
  LOW: cosmetic

Phase 4 — Fix Loop (per finding):
  Worker fixes ROOT CAUSE (not surface symptom)
  → Re-attack the fixed scope
  → Continue until no new findings

Phase 5 — Integration Adversarial Pass:
  One final attacker for the integrated whole
  → Fix any integration-level issues
```

### When to use Pattern 7

- After any implementation touching >3 files
- Before deploying to production
- When user says "审查", "review", "check for bugs"
- When user says "对抗式审查", "adversarial", "attack test"

## Pattern 8: Periodic Adversarial Audit

**When**: Every 2-3 weeks, or when user says "全面审查", "periodic audit", "定期审查".

**Scope**: Entire project — architecture, dependencies, code quality, documentation.

```
Phase 0 — First Principles: What is this project's core mission?
  What are the fundamental constraints? What assumptions have we made?
  Re-derive the architecture from scratch. Does current structure match?

Phase 1 — Architecture Audit (explorers, parallel):
├── Attacker-1: architecture consistency (does code match docs/design?)
├── Attacker-2: dependency analysis (circular? stale? bloated?)
└── Attacker-3: module boundaries (are they correct? leaking?)

Phase 2 — Code Quality Audit (explorers, parallel):
├── Attacker-4: code smells (duplication, god objects, long methods)
├── Attacker-5: error handling (swallowed errors, missing recovery)
└── Attacker-6: concurrency safety (missing locks, race conditions)

Phase 3 — Security & Robustness Audit (explorers, parallel):
├── Attacker-7: injection vectors (SQL, command, template)
├── Attacker-8: resource exhaustion (DoS, memory leaks, infinite loops)
└── Attacker-9: data integrity (corruption, inconsistency, loss)

Phase 4 — Documentation Audit (explorers, parallel):
├── Attacker-10: code-to-doc consistency
└── Attacker-11: API contract accuracy

Phase 5 — Report: Prioritized findings → User approval → Fix → Re-audit fixed scope
```

### Audit cadence recommendation

| Project maturity | Audit frequency |
|-----------------|-----------------|
| Active development | Every 2 weeks |
| Stable maintenance | Every 4 weeks |
| Pre-release | Full audit before every release |

## Pattern 9: First-Principles Bug Fix

**Task shape**: "Fix bug X" where the bug may be a symptom of deeper issues.

```
Phase 0 — First Principles (MANDATORY, main agent):
  1. What is the observed symptom?
  2. What is the ACTUAL root cause? (ask "why?" 5 times)
  3. What fundamental invariant is being violated?
  4. Is this a one-off bug or a class of bugs?
  5. If we fix the root cause, what ELSE might change?

Phase 1 — Exploration (if needed):
├── Explorer-1: trace the full data flow through the buggy path
└── Explorer-2: find all similar patterns in the codebase

Phase 2 — Fix:
  Worker: fix the ROOT CAUSE, not just the symptom.
  Write set: all files that need to change for the root-cause fix.

Phase 3 — Adversarial Review (Pattern 7):
  Attack the fix to ensure no regressions and no similar bugs remain.
```

### Example from article

Symptom: OpenAI scraper stopped working.
Surface fix: Repair the scraper.
First-principles fix: The underlying traffic routing mechanism had a design flaw
from April — fixing the routing layer prevents ALL future scraper failures, not
just this one.

## Anti-Patterns

### Surface Fix Pattern (VIOLATES first principles)
```
❌ Bug: scraper broken
❌ Fix: patch scraper → done
```
Root cause untouched. Bug will recur in another scraper.

### Fix-While-Attacking Pattern
```
❌ Attacker-1 finds concurrency bug
❌ Worker-1 starts fixing immediately
❌ Attacker-2 finds related state bug → Worker-1's fix is wrong
```
Fix: collect ALL findings before ANY fix.

### Symptom-Targeted Attack
```
❌ "Check if the login function works"
✓ "Find every way the login function can fail, including ways
   the user would never intentionally trigger"
```
Adversarial review must be creative, not checklist-driven.

### Overlapping Write Sets (unchanged)
```
❌ Worker-1: pkg/logic/qa.go
❌ Worker-2: pkg/logic/qa.go, pkg/logic/helper.go
```

### Spawn-Wait Spam (unchanged)
```
❌ spawn → wait → spawn → wait
```

### Phantom Agents (unchanged)
```
❌ Complete → collect → never close
```
