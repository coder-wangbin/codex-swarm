# Agent Lifecycle — Full Reference

## State Machine (Detailed)

```
                    ┌──────────┐
                    │  CREATED  │  spawn_agent returns successfully
                    └────┬─────┘
                         │ first tool call / output observed
                         ▼
                    ┌──────────┐
              ┌─────│ RUNNING  │──────┐
              │     └────┬─────┘      │
              │          │            │
    3 silent  │    wait_agent         │ agent returns error
    rounds    │    returns final      │ status or crashes
              │          │            │
              ▼          ▼            ▼
         ┌─────────┐ ┌──────────┐ ┌───────┐
         │ STALLED │ │COMPLETED │ │ ERROR │
         └────┬────┘ └────┬─────┘ └───┬───┘
              │           │           │
              └───────────┼───────────┘
                          │ collect result, close_agent
                          ▼
                     ┌──────────┐
                     │ COLLECTED │  result extracted by main agent
                     └────┬─────┘
                          │ close_agent
                          ▼
                     ┌──────────┐
                     │  CLOSED  │  resources freed, removed from pool
                     └──────────┘
```

## Pool Internals

### Data Structures

```
pool = {
  active: Map<agent_id, AgentState>,   // RUNNING agents
  completed: Queue<AgentState>,         // COMPLETED/ERROR/STALLED, waiting for collection
  recycle_bin: Queue<agent_id>,         // COLLECTED, waiting for close_agent
}

queue = Queue<SubTask>  // pending sub-tasks waiting for a slot
```

### Poll Loop Pseudocode

```
poll():
  results = []

  // 1. Drain completed agents
  for agent_id in wait_for_completed(timeout=2000ms):
    state = pool.active.remove(agent_id)
    if state.status == COMPLETED:
      result = extract_result(agent_id)
      results.append(result)
    state.status = COLLECTED
    pool.completed.push(state)

  // 2. Close all collected agents
  for state in pool.completed.drain():
    close_agent(state.agent_id)
    pool.recycle_bin.push(state.agent_id)

  // 3. Liveness check on remaining active
  for agent_id, state in pool.active:
    if agent_has_new_output(agent_id):
      state.silent_rounds = 0
    else:
      state.silent_rounds += 1
      if state.silent_rounds >= 3:
        state.status = STALLED
        close_agent(agent_id)
        pool.active.remove(agent_id)
        queue.push(state.subtask)  // re-queue

  // 4. Dequeue pending tasks into free slots
  while pool.active.size() < 12 and queue.not_empty():
    subtask = queue.pop()
    agent_id = spawn_agent(subtask)
    pool.active.add(agent_id, AgentState(status=RUNNING))

  return results
```

## Edge Cases

### Agent Returns Empty Result

Treat as COMPLETED but with zero changes. Collect, close, do not re-queue.
The sub-task may have determined no changes were needed.

### Agent Produces Output Then Goes Silent

The liveness counter resets on ANY output. An agent that produces output,
thinks for 2 minutes, produces more output, thinks for 2 minutes — is alive
the whole time and never reaches 3 silent rounds. Only agents that truly hang
(produce zero output across 3 poll rounds) get marked STALLED.

### All Workers Fail

If all workers for a goal fail and retries are exhausted:
1. Mark goal as `blocked`
2. Batch close all remaining agents
3. Report to user with failure summary

### Goal Complete But Agents Still Running

When `update_goal("complete")` is called:
1. Close ALL remaining agents (RUNNING, COMPLETED, ERROR)
2. Accept only results already collected
3. Do NOT wait for RUNNING agents to finish

### Agent Count Exceeds Pool Limit

`spawn_agent` calls that would exceed `max_concurrent_agents=12` are not issued.
Instead, the sub-task is pushed to `queue`. It dequeues automatically when
an active agent completes and its slot is freed.

## Concurrency Bounds Rationale

| Bound | Value | Reason |
|-------|-------|--------|
| Total agents | 12 | Balance between parallelism and context/resource pressure |
| Workers | ≤9 | Write-set isolation becomes harder to verify above this |
| Explorers | ≤6 | Read-only exploration scales well but hits diminishing returns |

If a task genuinely needs more than 12 agents, queue the excess — they'll
execute as slots free up. The queue is unlimited.
