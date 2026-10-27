# A2A-CDSL Task Lifecycle

## Overview

Every delegated task progresses through a defined state machine. The lifecycle is observable through CDSL expressions and enables auditability, stale detection, and proper orchestration.

## State Machine

```
init → dispatched → executing → returned → integrated
                  ↘ stale ↗
```

### States

| State | Meaning | Entry Condition | Exit Condition |
|-------|---------|-----------------|----------------|
| `init` | Task identified, not yet delegated | Master agent identifies a subtask | Master emits `>` expression |
| `dispatched` | Task sent to sub-agent | `>` expression emitted | Sub-agent acknowledges (implicit) |
| `executing` | Sub-agent is working | Sub-agent begins processing | Sub-agent emits `$` return |
| `returned` | Result received, not yet processed | `$` expression in `[SUB_RETURN]` | Master processes result |
| `integrated` | Result incorporated into master's plan | Master uses result in next step | Terminal state |
| `stale` | Task exceeded timeout without returning | Timeout expires | Re-delegated or escalated |

### Transitions

| From | To | Trigger | CDSL Evidence |
|------|----|---------|---------------|
| init | dispatched | Master emits `>` | `>sub_id:role` in `[THOUGHT]` |
| dispatched | executing | Sub-agent starts work | (implicit — no CDSL marker) |
| executing | returned | Sub-agent emits `$` | `$status:...` in `[SUB_RETURN]` |
| returned | integrated | Master processes result | (internal — no CDSL marker) |
| executing | stale | Timeout expires | `#status:stale` emitted by master |
| stale | dispatched | Re-delegation | New `>` expression for same task |
| stale | integrated | Escalation to human | Gateway produces failure report |

## Lifecycle Tracking Expressions

### Task Initialization
```
[THOUGHT]
#task:init | #id:task_001 | #assign:sub1
[/THOUGHT]
```

### Task Dispatch
```
[THOUGHT]
>sub1:sec_audit | ?f:src/db/** | !vuln:sql_injection
[/THOUGHT]
```

### Task Return
```
[SUB_RETURN id=sub1]
$status:OK | $result:no_vulnerabilities_found
[/SUB_RETURN]
```

### Task Integration
```
[THOUGHT]
#task:integrated | #id:task_001 | #result:a3f2b1
[/THOUGHT]
```

### Stale Detection
```
[THOUGHT]
#task:stale | #id:task_001 | #reason:timeout | #elapsed:300s
[/THOUGHT]
```

### Re-delegation
```
[THOUGHT]
>sub1_retry:sec_audit | ?f:src/db/** | !vuln:sql_injection
[/THOUGHT]
```

## Stale Detection Rules

1. Each delegated task has a configurable timeout (default: 300 seconds).
2. If no `$` return is received within the timeout, the master agent MUST emit a `#task:stale` expression.
3. Upon detecting stale status, the master agent MUST take one of:
   - **Re-delegate**: emit a new `>` for the same task (max 2 retries).
   - **Escalate**: pass the failure to the human gateway.
   - **Abandon**: mark the task as permanently failed and continue with remaining tasks.
4. Maximum re-delegation attempts per task: 2. After 2 failures, the task is permanently `stale`.

## Auditability

Every state transition is recorded. The full lifecycle of a task can be reconstructed from the CDSL expression log:

```
#task:init → >sub1:role → $status:OK → #task:integrated
```

This enables:
- Post-task analysis of which sub-agents succeeded/failed.
- Timing analysis (time between `dispatched` and `returned`).
- Retry tracking (count of re-delegations per task).

## Terminal States

| State | Meaning |
|-------|---------|
| `integrated` | Task completed successfully, result used |
| `failed` | Task permanently failed after max retries |
| `abandoned` | Task cancelled by master agent decision |

Terminal states are final — no further transitions occur.
