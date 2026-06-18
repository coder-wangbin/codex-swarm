# Decomposition Patterns

## Pattern 1: Multi-Module Feature

**Task shape**: "Add feature X to modules A, B, and C"

```
Decomposition:
├── Worker-1: module_a/feature.go
├── Worker-2: module_b/feature.go
└── Worker-3: module_c/feature.go
```

Each worker implements the same feature pattern in a different module.
Write sets are naturally disjoint by module directory.

## Pattern 2: Explore-Then-Implement

**Task shape**: "Understand how X works, then add Y"

```
Phase 1 (parallel explorers):
├── Explorer-1: explore data flow through module A
├── Explorer-2: explore data flow through module B
└── Explorer-3: explore handler/router structure

Phase 2 (after collection, serial or parallel depending on write sets):
├── Worker-1: implement Y in module A
└── Worker-2: implement Y in module B
```

Phase 1 runs fully parallel (all read-only). Phase 2 workers start only
after all explorer results are collected and analyzed.

## Pattern 3: Implement + Verify (Dual-Track)

**Task shape**: "Implement X with tests"

```
Decomposition:
├── Worker-1: implement X in source files
├── Worker-2: write tests for X (separate test files)
└── Explorer-1: explore existing test patterns for style reference
```

Worker-1 and Worker-2 have disjoint write sets (source vs test files).
Explorer-1 is read-only.

## Pattern 4: Cross-Cutting Refactor

**Task shape**: "Rename function F across the codebase"

**⚠️ NOT parallelizable** — all files share the same function signature.
This triggers exactly 0 parallel conditions. Execute serially.

However, if the refactor has independent sub-refactors:
```
"Rename F in pkg/a and restructure pkg/b independently"
├── Worker-1: rename F in pkg/a/** (write set: pkg/a/)
└── Worker-2: restructure pkg/b/** (write set: pkg/b/)
```
This IS parallelizable because write sets are disjoint.

## Pattern 5: Multi-Repo / Multi-Service

**Task shape**: "Update API in service-A, consume in service-B"

```
Phase 1 (can be parallel if API contract is pre-agreed):
├── Worker-1: update API in service-A
└── Worker-2: update consumer in service-B

Phase 2 (after both complete):
└── Main agent: verify integration
```

## Pattern 6: Build + Test + Lint (Verification Fan-Out)

**Task shape**: "After changes, verify everything"

```
Parallel verification:
├── Worker-1: go build ./...
├── Worker-2: go test ./pkg/...
├── Worker-3: go test ./internal/...
└── Worker-4: golangci-lint run
```

All read-only (or write to separate log files). Fully parallel.

## Anti-Patterns

### Anti-Pattern: Overlapping Write Sets

```
❌ Worker-1: pkg/logic/qa.go
❌ Worker-2: pkg/logic/qa.go, pkg/logic/helper.go  ← CONFLICT on qa.go
```

Fix: merge into one worker, or split the file responsibilities clearly.

### Anti-Pattern: Spawn-Wait Spam

```
❌ spawn_agent(worker1)
❌ wait_agent(worker1)    ← blocks everything
❌ spawn_agent(worker2)
❌ wait_agent(worker2)
```

Fix: spawn all first, then cycle through collecting.

### Anti-Pattern: Phantom Agents

```
❌ spawn_agent(worker1)
❌ worker1 completes → result extracted
❌ ... never close_agent(worker1)   ← resource leak
```

Fix: close immediately after collecting result.

### Anti-Pattern: Main Agent Does Sub-Agent's Work

```
❌ Worker-1: implements feature in auth.go
❌ Main agent: also modifies auth.go "to fix things"   ← conflicts
```

Fix: if worker's output is wrong, re-queue or fix after integration, but never
modify files in a worker's write set while that worker is running.
