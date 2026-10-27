# A2A-CDSL Delegation Protocol

## Overview

Master agents delegate tasks to sub-agents using the `>` opcode within their internal thinking chain. Delegation is expressed entirely in CDSL — no natural-language intermediate text is generated for task dispatch.

## Delegation Patterns

### 1. Single Task Delegation

A master agent identifies one subtask requiring a specialized agent.

**Pattern:**
```
[THOUGHT]
>sub_<id>:<role> | ?<target_scope> | !<validation>
[/THOUGHT]
```

**Example:**
```
[THOUGHT]
>sub1:sec_audit | ?f:src/db/** | !vuln:sql_injection
[/THOUGHT]
```

**Semantics:**
- `>sub1:sec_audit` — create/assign sub-agent `sub1` with role `sec_audit`
- `?f:src/db/**` — target scope: all files under `src/db/`
- `!vuln:sql_injection` — validation to perform: check for SQL injection

---

### 2. Parallel Multi-Task Delegation

A master agent identifies multiple independent subtasks and dispatches them simultaneously.

**Pattern:**
```
[THOUGHT]
>sub_<id1>:<role1> | ?<scope1> | !<check1>
>sub_<id2>:<role2> | ?<scope2> | !<check2>
>sub_<id3>:<role3> | ?<scope3> | !<check3>
[/THOUGHT]
```

**Example:**
```
[THOUGHT]
>sub_db:perf_opt | ?metrics:db_latency | +analyze:slow_query
>sub_sec:vulnerability | ?code:src/auth/ | !check:token_leak
[/THOUGHT]
```

**Semantics:**
- Each `>` line is an independent delegation.
- Sub-agents execute concurrently.
- The master agent waits for all `$` returns before proceeding.

---

### 3. Chained (Sequential) Delegation

A master agent delegates a task whose output feeds into a subsequent task.

**Pattern:**
```
[THOUGHT]
>sub_<id1>:<role1> | ?<scope1>
[/THOUGHT]

[SUB_RETURN id=sub_<id1>]
$status:OK | $result:<findings>
[/SUB_RETURN]

[THOUGHT]
>sub_<id2>:<role2> | #ctx:<hash_from_sub1> | !<check2>
[/THOUGHT]
```

**Example:**
```
[THOUGHT]
>sub_scan:code_analysis | ?f:src/**/*.py
[/THOUGHT]

[SUB_RETURN id=sub_scan]
$status:OK | $result:findings_a7f3
[/SUB_RETURN]

[THOUGHT]
>sub_fix:refactor | #ctx:findings_a7f3 | +exec:apply_fixes
[/THOUGHT]
```

**Semantics:**
- Sequential delegation uses `#ctx:<hash>` to pass results from one sub-agent to the next.
- The master agent resolves the hash pointer to provide context to the next sub-agent.

---

## Delegation Rules

1. **Agent identifiers** MUST be unique within a task chain. Format: `sub_<name>` or `<name><number>`.
2. **Roles** describe the sub-agent's purpose (e.g., `sec_audit`, `perf_opt`, `code_review`).
3. **Target scope** is specified via `?` segments in the same delegation expression.
4. **Validation criteria** are specified via `!` segments to tell the sub-agent what to check.
5. **State sharing** uses `#ctx:<hash>` pointers — never inline full context.
6. **Maximum parallel delegations** per expression: 8 (practical limit for context window management).
7. **Maximum chain depth**: 3 levels (master → sub → sub-sub). Beyond 3, the master agent should restructure the task.

## Delegation Lifecycle

```
init → dispatched → executing → returned → integrated
```

| Transition | Trigger | CDSL Evidence |
|-----------|---------|---------------|
| init → dispatched | Master emits `>` expression | `>` segment in `[THOUGHT]` block |
| dispatched → executing | Sub-agent acknowledges receipt | (implicit) |
| executing → returned | Sub-agent emits `$` expression | `$` segment in `[SUB_RETURN]` block |
| returned → integrated | Master processes result | (internal) |

## Stale Detection

If a sub-agent does not emit a `$` return within the configured timeout, the task transitions to `stale`:

```
[THOUGHT]
#status:stale | #task:<agent_id> | #reason:timeout
[/THOUGHT]
```

The master agent MAY then:
- Re-delegate: emit a new `>` for the same task.
- Escalate: report the failure to the human via the gateway.
- Abandon: mark the task as permanently failed.
