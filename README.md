# Goal Parallel

A [Codex](https://github.com/openai/codex) skill for automatic parallel task decomposition with full sub-agent lifecycle management.

## What It Does

When you give Codex a complex task spanning multiple modules or files, this skill automatically:

1. **Decomposes** the task into independent sub-tasks with non-overlapping write sets
2. **Spawns parallel agents** (workers + explorers) up to 12 concurrently
3. **Monitors liveness** — detects stalled agents without fixed timeouts
4. **Collects results** and auto-recycles all agents (no resource leaks)
5. **Integrates** everything and verifies the final result

## Trigger Conditions

The skill activates **automatically** when ≥2 of these are true:

- Task spans ≥3 files or ≥2 modules
- Both exploration and implementation needed
- ≥2 independent sub-tasks with no data dependency
- Multi-dimensional verification required
- User says "parallel", "并行", "同时"

No manual invocation needed — the model evaluates and decides.

## Installation

```bash
# Via skill-installer (if available)
# Or clone directly:
git clone https://github.com/coder-wangbin/goal-parallel.git ~/.codex/skills/goal-parallel
```

Restart Codex after installation.

## Structure

```
goal-parallel/
├── SKILL.md              # Main skill instructions (loaded into context)
├── agents/
│   └── openai.yaml       # UI metadata for skill lists
├── references/
│   ├── lifecycle.md      # Full agent lifecycle state machine
│   └── patterns.md       # Decomposition patterns & anti-patterns
├── assets/
│   └── icon.svg          # Skill icon
└── LICENSE.txt
```

## License

MIT
