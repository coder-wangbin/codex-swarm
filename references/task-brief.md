# Structured Agent Task Brief — Template & Anti-Cheat Clauses

## Why Task Briefs Matter

Swarm spawns parallel agents. Without structure, each agent interprets its mission
differently — one writes perfect tests, another skips them entirely. Structured
briefs ensure consistency across all spawned agents.

Inspired by the [leader](https://github.com/KKKKhazix/khazix-skills/tree/main/leader)
skill's task book format: every spawned agent gets a properly gated mission with
boundaries, verification, and anti-cheat instructions.

## Task Brief Template

When spawning a worker agent, the mission description must follow this structure:

```
[AGENT ROLE]: worker-exec | worker-fix | explorer | attacker

[MISSION]: One sentence. What to accomplish, with numbers if possible.

[BOUNDARY — 地界]:
  WRITE: [exact file paths, comma separated]
  READ: [directories or files, or "any"]
  FREEZE (判卷标准): [test files, CI config, baselines — never touch these]

[BASELINE — 基线]:
  Before starting: [commands to run, expected output, numbers]
  This verifies your starting state is correct.

[VERIFICATION — 验收]:
  [commands to run after completion]
  [machine-readable pass/fail criteria]
  [baseline comparison: test count must be ≥ N, coverage ≥ X%]

[ANTI-CHEAT — 防作弊]:
  DO NOT:
  - Skip tests (.skip, t.Skip(), it.skip, xtest)
  - Relax assertions (widen tolerances, change expected values)
  - Mock core logic (replace the function you're testing with a stub)
  - Delete tests (remove failing test cases)
  - Mask failures (|| true, || exit 0, set +e)
  - Modify test baselines (change coverage thresholds, test counts)
  - Change verification commands or CI configuration

  Test count and coverage must be ≥ baseline. Violation = task failure.

[FORBIDDEN — 禁止顺手活]:
  If you notice unrelated bugs, refactoring opportunities, or missing features:
  Write them to your findings output. Do NOT fix them.
  Your write set boundary is absolute — stay within it.

[ADVERSARIAL SELF-CHECK — 反向验证]:
  After completing your changes:
  1. Intentionally trigger ONE failure condition
  2. Show the RED output (proves your verification catches errors)
  3. Restore the correct state
  4. Show the GREEN output

[PROGRESS]:
  On completion, output a single line:
  "PROGRESS: [agent-id] completed [mission summary] at [timestamp].
   Files changed: [list]. Baseline: [before→after]."

[OUTPUT]:
  - Changes made (files + diff summary)
  - Verification output (red → green evidence)
  - PROGRESS line
  - BLOCKED items (if any): things you couldn't resolve
```

## Anti-Cheat Clauses — Full Catalog

Use these as building blocks. Include relevant ones in each agent's mission.

### Standard Implementation Worker

```
DO NOT:
- Skip tests (.skip, t.Skip(), it.skip, xtest)
- Relax assertions (widen tolerances, change expected values)
- Replace tested functions with stubs or mocks
- Delete failing test cases
- Use ||true, ||exit 0, or set +e in verification commands
- Modify test count baselines or coverage thresholds
- Change files in [FREEZE] list
```

### Test-Writing Worker

```
DO NOT:
- Write tests that always pass (empty body, trivially true assertions)
- Test only the happy path — must include ≥2 error/edge cases per function
- Skip or mark as TODO any test case from the spec
- Modify production code to make tests pass unless explicitly allowed
- Test count after changes must be ≥ baseline + planned additions
```

### Fix Worker (adversarial review follow-up)

```
DO NOT:
- Fix the bug by removing the feature that had the bug
- Fix one instance and leave the same bug pattern elsewhere
- Change the verification command to exclude the bug trigger
```

### Explorer / Baseline Verification

```
DO NOT:
- Skip commands that fail — report the failure and its output
- Assume a command exists without running it
- Report "looks correct" without the actual command output
- Trust documentation over actual command output
```

## Adversarial Self-Check (反向验证) — Detailed Protocol

The core insight from leader: "凡是坏了没人会知道的检查，都要这一步" —
if a verification has no way to prove it detects failures, spawn an adversarial
self-check.

### When to Require

| Scenario | Self-Check Required? |
|----------|---------------------|
| Bug fix | YES — trigger the original bug, prove it's caught |
| New feature | YES — trigger a misuse scenario, prove validation catches it |
| Test addition | YES — make the test fail intentionally, prove it's real |
| Refactor | YES — break the refactored invariant, prove tests catch it |
| Config change | YES — apply invalid config, prove startup fails cleanly |
| Doc update | NO (no machine-verifiable check exists) |

### Protocol

```
1. Identify ONE failure condition that your change should catch
2. Temporarily introduce that failure
3. Run the verification command → must show RED/FAIL
4. Capture the output as evidence
5. Remove the temporary failure
6. Run the verification command → must show GREEN/PASS
7. Capture the output as evidence
8. Output both red and green evidence in your results
```

### Example Output

```
=== ADVERSARIAL SELF-CHECK ===
Condition: user with role=0 (普通用户) attempts to upload to public library
Step 1 (trigger): set test user role to 0
Step 2 (red): curl -X POST /api/adm/doc/upload → HTTP 403 Forbidden ✓
Step 3 (restore): set test user role to 1
Step 4 (green): curl -X POST /api/adm/doc/upload → HTTP 200 OK ✓
Evidence: red output captured, green output captured
=== CHECK PASSED ===
```

## PROGRESS.md Pattern

For multi-agent tasks spanning multiple sessions, each agent outputs a progress
marker. The main agent compiles these into a local `PROGRESS.md`:

```markdown
# PROGRESS — Swarm execution

## Agent: worker-1 (auth middleware fix)
Status: COMPLETED 2026-07-27 14:30
Files: pkg/middleware/auth.go (+12/-3)
Baseline: test count 47→47, coverage 82%→82%
Verification: green (red→green evidence captured)

## Agent: worker-2 (route access control)
Status: COMPLETED 2026-07-27 14:35
Files: pkg/router/router.go (+8/-1)
Baseline: test count 23→23, coverage 76%→76%
Verification: green (red→green evidence captured)

## Agent: worker-3 (JWT expiry fix)
Status: RUNNING — stalled at 14:32, re-queued
```

On session resume, main agent reads PROGRESS.md and continues from incomplete agents.
Completed agents' results are already collected — do NOT redo their work.

## Task Type Fork

### Executive (执行型) — when you can write verification commands NOW

```
Agent type: worker-exec
Verification: concrete commands with pass/fail
Success: all verification commands pass + reverse verification evidence
Failure: any verification command fails after 3 retries
```

### Exploratory (探索型) — when user wants answers, not code

```
Agent type: explorer
Verification: learning goals, not hard metrics
Success: N conclusions with evidence sources + reproduction steps
Failure: fabricated conclusions, unverifiable claims, "实测" without output
Anti-cheat focus: prevent hallucination, not test-skipping
```
