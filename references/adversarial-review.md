# Adversarial Review — Methodology & Attack Vector Taxonomy

## Core Principle

> 你永远需要一个站在你对面的力量来告诉你，你可能是错的。
> You always need an opposing force to tell you that you might be wrong.

Adversarial review is NOT "testing." Testing verifies expected behavior. Adversarial
review finds unexpected failure modes by actively trying to break the system.

## Mindset Shift

| Passive Testing | Adversarial Review |
|----------------|-------------------|
| "Does it work correctly?" | "How can I make it fail?" |
| Run existing test suite | Invent new attack vectors |
| Verify happy path | Explore every edge case |
| Same agent that wrote code | Fresh agents with attacker mindset |
| Pass/fail result | Ranked finding report |

## Attack Vector Taxonomy

When generating attack vectors, systematically cover these dimensions:

### 1. Temporal (时间)

| Vector | Description | Example trigger |
|--------|-------------|-----------------|
| Future time | Timestamp from the future | `publish_time = now() + 7 days` |
| Zero time | Unix epoch (1970-01-01) | `created_at = 0` |
| Overflow | Time beyond int64 range | `expires_at = 2^63 - 1` |
| Negative time | Before epoch | `last_login = -1` |
| Timezone edge | DST transitions, UTC±12 | Samoa timezone skip day |
| Clock skew | Server clocks out of sync | `created_at > updated_at` |
| Monotonic violation | Time going backwards | NTP adjustment mid-operation |

### 2. Data (数据)

| Vector | Description | Example trigger |
|--------|-------------|-----------------|
| Empty | Zero-length strings, nil slices, null values | `name = ""`, `tags = nil` |
| Massive | Extreme size inputs | 100MB HTML, 10M-row CSV |
| Nested | Deeply recursive structures | JSON 1000 levels deep |
| Malformed | Invalid encoding, truncated | UTF-8 with stray bytes |
| Boundary | Min/max values, off-by-one | `limit = 0`, `limit = -1`, `limit = 2^31` |
| Type confusion | String where int expected | `"123abc"` in numeric field |
| Duplicate | Same unique key twice | Two users with same email |
| Encoding collision | Same content, different encoding | `"café"` NFC vs NFD |

### 3. Concurrency (并发)

| Vector | Description | Example trigger |
|--------|-------------|-----------------|
| Race condition | Two writes to same data | Simultaneous user profile updates |
| Deadlock | Circular lock dependency | A locks X then Y, B locks Y then X |
| Partial write | Crash mid-transaction | Kill process during multi-row insert |
| Double-submit | Same request sent twice | Form double-click, retry logic |
| Read-after-write | Stale read before write propagates | Cache not invalidated yet |
| CAS failure | Compare-and-swap retry exhaustion | High-contention counter |
| Goroutine leak | Unbounded goroutine spawn | Channel never closed, WaitGroup never Done |

### 4. Resource (资源)

| Vector | Description | Example trigger |
|--------|-------------|-----------------|
| Memory exhaustion | Unbounded allocation | Read entire file into memory |
| File descriptor leak | Open but never close | Forgot `defer f.Close()` |
| Connection pool drain | All connections in use | Peak traffic, no timeout |
| Disk full | Write until no space | Unbounded log rotation |
| CPU spin | Infinite loop or tight polling | `for {}` without yield |
| Recursion depth | Stack overflow | Deeply nested data structure |
| Goroutine explosion | N concurrent spawns where N is unbounded | Process 100K items with one goroutine each |

### 5. State (状态)

| Vector | Description | Example trigger |
|--------|-------------|-----------------|
| Cache inconsistency | Cache says success, DB says failure | Cache write succeeds, DB write fails |
| Stale reference | Pointer to deleted/freed data | Object deleted while iterator holds reference |
| Interrupted workflow | Step 2 of 3 fails, state is now invalid | Payment succeeded but order not created |
| Orphaned data | Child records with no parent | Delete user, keep their posts |
| State machine violation | Invalid state transition | Order shipped before payment confirmed |
| Idempotency failure | Same operation applied twice gives wrong result | Double charge on retry |
| Rollback incomplete | Partial cleanup after error | Some resources freed, some not |

### 6. Input/Security (输入/安全)

| Vector | Description | Example trigger |
|--------|-------------|-----------------|
| SQL injection | Unescaped input in query | `name = "'; DROP TABLE users; --"` |
| Command injection | User input passed to shell | `filename = "; rm -rf /"` |
| Template injection | User input in template engine | `{{ .SystemConfig }}` in user bio |
| Path traversal | `../` in file paths | `path = "../../../etc/passwd"` |
| Null byte injection | `\x00` truncation | `filename = "good.txt\x00.php"` |
| Unicode confusion | Homoglyph attacks | `admin` vs `аdmin` (Cyrillic 'a') |
| ReDoS | Regex catastrophic backtracking | `(a+)+b` on `aaaaaaaaac` |
| Deserialization | Malicious serialized object | Pickle/JSON bombs |

## Multi-Agent Attack Orchestration

### Agent Assignment Strategy

```
Attack scope: pkg/logic/qa.go (changed in this PR)

Attacker-1: temporal + state (these often interact — stale cache + clock skew)
Attacker-2: data + input (malformed inputs through data pipelines)
Attacker-3: concurrency + resource (stress-test the shared paths)
```

Group complementary vectors together. Don't assign one vector per agent if the
vectors are simple — combine 2-3 related vectors per agent.

### Attack Agent Prompt Template

When spawning an attack agent, use this structure:

```
"Act as an adversarial reviewer. Your mission: find every way this code can fail.
Focus on {vector_category_1}, {vector_category_2}, and {vector_category_3}.
Scope: {file_list_or_module}.
Do NOT fix anything. Only report findings with:
- Severity (CRITICAL/HIGH/MEDIUM/LOW)
- Trigger condition
- Expected vs actual behavior
- Affected code location
Be creative. Think like someone trying to break this, not someone trying to use it."
```

### Parallel Attack Rules

1. **All attackers spawn simultaneously** — do not wait for one before spawning the next
2. **All attackers are read-only** — they never modify code
3. **Collect all findings before any fix** — fixing mid-attack can invalidate other findings
4. **Deduplicate findings** — if two attackers find the same issue, merge into one report entry

## Fix Loop Protocol

```
for each finding (sorted by severity: CRITICAL → HIGH → MEDIUM → LOW):
    1. Main agent analyzes: is this a surface symptom or a root cause?
    2. If surface: ask "why?" until root cause is found
    3. Worker fixes root cause (write set: affected files)
    4. Spawn ONE attacker to re-verify the fixed scope
    5. If attacker finds new issues → go to step 1 for new issues
    6. Continue until attack on this scope yields zero findings
```

**Important**: After fixing a CRITICAL or HIGH finding, ALWAYS re-attack with at
least one fresh attacker. Medium/Low findings may be batched for efficiency.

## Regular Audit Checklist

For periodic audits (Pattern 8), the attack scope is the entire project.
Use this checklist to ensure coverage:

```
□ Architecture: Does code structure match documented design?
□ Dependencies: Circular? Stale? Security vulnerabilities?
□ Error handling: Swallowed errors? Missing recovery paths?
□ Concurrency: Races? Deadlocks? Goroutine leaks?
□ Resource management: Leaks? Unbounded growth?
□ State management: Inconsistencies? Orphaned data?
□ Security: Injection? Auth bypass? Data exposure?
□ Performance: N+1 queries? Unbounded loops? Memory pressure?
□ Documentation: Code-to-doc consistency? API accuracy?
□ Test coverage: Critical paths untested? Flaky tests?
```

## Success Metrics

After an adversarial review cycle:

- Found ≥1 issue you didn't know about → review was valuable
- Found 0 issues → either the code is genuinely solid OR the attack vectors were too narrow
- Every finding has a root-cause fix, not a surface patch
- Re-attack on fixed scope yields zero new findings
