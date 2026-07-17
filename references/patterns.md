# Decomposition Patterns

## Pattern 1: Multi-Module Feature

```
First Principles: Is feature X the right abstraction for all modules?

Decomposition:
├── Worker-1: module_a/feature.go
├── Worker-2: module_b/feature.go
└── Worker-3: module_c/feature.go
```

## Pattern 2: Explore-Then-Implement

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
├── Worker-2: write tests for X
└── Explorer-1: explore existing test patterns
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
├── Worker-3: golangci-lint run
```

## Pattern 7: Adversarial Review (Attack-Then-Fix)

```
Phase 0 — First Principles: What assumptions did we make? What invariants must hold?

Phase 1 — Attack Vector Generation: ≥5 edge cases per category

Phase 2 — Parallel Attack (all read-only):
├── Attacker-1: temporal edge cases
├── Attacker-2: data edge cases
├── Attacker-3: concurrency edge cases
├── Attacker-4: resource edge cases
├── Attacker-5: state edge cases
└── Attacker-6: input edge cases

Phase 3 — Collect & Categorize: CRITICAL/HIGH/MEDIUM/LOW
Phase 4 — Fix Loop: fix root cause → re-attack → repeat
Phase 5 — Integration pass: one final attacker for the whole
```

## Pattern 8: Periodic Adversarial Audit

Every 2-4 weeks:
```
Phase 1 — Architecture: consistency, dependencies, boundaries (3 attackers)
Phase 2 — Code Quality: smells, error handling, concurrency (3 attackers)
Phase 3 — Security: injection, resource, data integrity (3 attackers)
Phase 4 — Documentation: code-to-doc consistency, API accuracy (2 attackers)
Phase 5 — Report → User approval → Fix → Re-audit
```

## Pattern 9: First-Principles Bug Fix

```
Phase 0 (MANDATORY):
  1. What is the observed symptom?
  2. What is the ROOT cause? (ask "why?" 5 times)
  3. What fundamental invariant is being violated?
  4. Is this a one-off or a class of bugs?

Phase 1 — Exploration (if needed):
├── Explorer-1: trace full data flow
└── Explorer-2: find all similar patterns

Phase 2 — Fix root cause
Phase 3 — Adversarial Review (Pattern 7)
```

## Pattern 10: Auth/Permission Security Audit

**Trigger**: "越权", "权限漏洞", "安全审查", "安全审计", "security audit"

```
Phase 0 — First Principles:
  What is the trust boundary? Who can access what?

Phase 1 — Surface Map:
  └── Explorer-1: middleware chain, route groups, role constants, JWT config

Phase 2 — Parallel Attack:
├── Attacker-1: middleware chain + route access control
│   (Does every route have correct auth? Any middleware ordering bugs?)
├── Attacker-2: JWT/Cookie/Session security
│   (Expiry, signing, domain scope, rotation, leakage)
├── Attacker-3: role/permission escalation
│   (Can low-privilege user reach admin endpoints? Default role bugs?)
└── Attacker-4: DB-level consistency
│   (Code RBAC vs DB RBAC drift, direct-DB bypass)

Phase 3 — Report: CRITICAL/HIGH/MEDIUM/LOW
Phase 4 — Fix + Re-attack (per CRITICAL/HIGH finding)
```

See `references/auth-review.md` for full Go-specific checklist.

## Pattern 11: Multi-Project Parallel Analysis

**Trigger**: When user asks about multiple projects simultaneously, or
switches between skb/dinghe/lma in one conversation.

```
"检查 skb 和 dinghe 的 auth 模块有什么差异"

├── Explorer-1: analyze skb auth (middleware, role, JWT)
└── Explorer-2: analyze dinghe auth (middleware, role, JWT)

→ Main agent: diff analysis, best-practice recommendations
→ Shared write set: NONE (read-only analysis)
```

## Anti-Patterns

### Surface Fix (violates first principles)
```
❌ Bug: scraper broken → Fix: patch scraper → done
```
Root cause untouched. Bug recurs in another scraper.

### Fix-While-Attacking
```
❌ Attacker finds bug → Worker fixes immediately
   → Attacker-2 finds related bug → fix is wrong
```
Fix: collect ALL findings before ANY fix.

### Symptom-Targeted Attack
```
❌ "Check if the login function works"
✓ "Find every way the login function can fail"
```

### Overlooking Auth in Backend Projects
```
❌ Adversarial review only checks data/concurrency
   → misses middleware ordering bug
✓ Always include auth-specific attackers for Go backend projects
```

### Overlapping Write Sets
### Spawn-Wait Spam
### Phantom Agents
(Unchanged from v2)
