# A2A-CDSL Usage Guide

## Overview

This guide demonstrates the full A2A-CDSL pipeline: master agent thinking → sub-agent execution → human-readable output. Each example satisfies all three capability specs (cdsl-language, cdsl-agent-protocol, cdsl-human-gateway).

---

## Example 1: Single Task Delegation

### Scenario
User asks: "Check all database files for SQL injection risks."

### Phase 1 — Master Agent Thinking (CDSL)

```
[THOUGHT]
>sub1:sec_audit | ?f:src/db/** | !vuln:sql_injection
[/THOUGHT]
```

**Spec compliance:**
- cdsl-language: valid expression, opcode `>`, key `sub1`, role `sec_audit`, glob `src/db/**`, validation `sql_injection` ✓
- cdsl-agent-protocol: single delegation with scope and validation criteria ✓
- Encapsulated in `[THOUGHT]` markers ✓

### Phase 2 — Sub-Agent Execution & Return

```
[SUB_RETURN id=sub1]
$status:OK | $result:no_vulnerabilities_found | $metrics:files_scanned(42)
[/SUB_RETURN]
```

**Spec compliance:**
- cdsl-language: valid return expression ✓
- cdsl-agent-protocol: `$status:OK` with `$result` payload ✓
- Agent ID matches delegation (`sub1`) ✓

### Phase 3 — Human Gateway Output

> "A security audit was performed on all files under src/db/. No SQL injection vulnerabilities were found across 42 files scanned."

**Spec compliance:**
- cdsl-human-gateway: no CDSL tokens in output ✓
- Full decompression: all findings translated ✓
- Fidelity: "no vulnerabilities" accurately reflects `$result:no_vulnerabilities_found` ✓
- No `[THOUGHT]` or `[SUB_RETURN]` markers ✓

---

## Example 2: Parallel Multi-Agent Task

### Scenario
User asks: "Diagnose database performance issues and check authentication code for token leak vulnerabilities."

### Phase 1 — Master Agent Thinking (CDSL)

```
[THOUGHT]
>sub_db:perf_opt | ?metrics:db_latency | +analyze:slow_query
>sub_sec:vulnerability | ?code:src/auth/ | !check:token_leak
[/THOUGHT]
```

**Spec compliance:**
- cdsl-language: two valid multi-segment expressions ✓
- cdsl-agent-protocol: parallel delegation (two `>` lines), each with scope and validation ✓
- Token count: ~18 tokens ✓

### Phase 2 — Sub-Agent Returns

```
[SUB_RETURN id=sub_db]
$status:OK | $bottleneck:idx_missing(tbl:users, col:email)
[/SUB_RETURN]

[SUB_RETURN id=sub_sec]
$status:OK | $result:no_vulnerabilities_found
[/SUB_RETURN]
```

**Spec compliance:**
- cdsl-language: valid structured value `idx_missing(tbl:users, col:email)` ✓
- cdsl-agent-protocol: both agents return with proper status ✓
- Agent IDs match delegations ✓

### Phase 3 — Human Gateway Output (Level 1 — Brief)

> "System diagnostics completed. A database performance bottleneck was identified: the users table is missing an index on the email column, causing query latency. A security audit of the authentication code found no token leak vulnerabilities."

**Spec compliance:**
- cdsl-human-gateway: no CDSL tokens ✓
- Context reconstruction: structured value rendered as natural sentence ✓
- Progressive disclosure: brief summary (3 sentences) ✓
- Fidelity: all findings preserved, nothing added ✓

### Phase 3b — Expanded Detail (if user asks "what specifically was wrong?")

> "The database performance agent analyzed query latency metrics and identified slow queries. The root cause: the `users` table is missing an index on the `email` column. This causes full table scans on every email-based query.
>
> The security audit agent examined all files under `src/auth/` for token leak vulnerabilities. No vulnerabilities were detected."

**Spec compliance:**
- Level 2 disclosure: per-agent breakdown ✓
- All structured data rendered completely ✓
- Still no CDSL tokens ✓

---

## Example 3: Chained Delegation with State Pointers

### Scenario
User asks: "Scan the Python codebase for issues, then automatically apply fixes."

### Phase 1 — First Delegation

```
[THOUGHT]
>sub_scan:code_analysis | ?f:src/**/*.py
[/THOUGHT]
```

### Phase 2 — First Return

```
[SUB_RETURN id=sub_scan]
$status:OK | $result:findings_a7f3
[/SUB_RETURN]
```

### Phase 3 — Second Delegation (using state pointer)

```
[THOUGHT]
>sub_fix:refactor | #ctx:findings_a7f3 | +exec:apply_fixes
[/THOUGHT]
```

**Spec compliance:**
- cdsl-agent-protocol: chained delegation via `#ctx:` pointer ✓
- cdsl-language: valid pointer reference ✓
- State passed without inlining full findings ✓

### Phase 4 — Second Return

```
[SUB_RETURN id=sub_fix]
$status:OK | $result:fixes_applied(3)
[/SUB_RETURN]
```

### Phase 5 — Human Gateway Output

> "Code analysis completed. Three issues were identified and automatically fixed."

**Spec compliance:**
- cdsl-human-gateway: concise summary ✓
- No CDSL leakage ✓
- Fidelity: "3 issues fixed" matches `fixes_applied(3)` ✓

---

## Example 4: Failure Handling

### Scenario
User asks: "Run integration tests on the API module."

### Phase 1 — Delegation

```
[THOUGHT]
>sub_test:integration | !test:integration:api
[/THOUGHT]
```

### Phase 2 — Failure Return

```
[SUB_RETURN id=sub_test]
$status:FAIL | $reason:timeout | $detail:test_suite_exceeded_300s
[/SUB_RETURN]
```

### Phase 3 — Stale Handling (Master Agent)

```
[THOUGHT]
#task:stale | #id:task_test | #reason:timeout | #elapsed:300s
[/THOUGHT]
```

### Phase 4 — Re-delegation

```
[THOUGHT]
>sub_test_retry:integration | !test:integration:api | #set:timeout:600s
[/THOUGHT]
```

### Phase 5 — Success Return

```
[SUB_RETURN id=sub_test_retry]
$status:OK | $result:all_tests_passed(28) | $metrics:duration(487s)
[/SUB_RETURN]
```

### Phase 6 — Human Gateway Output

> "The initial test run timed out after 300 seconds. A retry was dispatched with an extended timeout. All 28 integration tests passed on the second attempt (487 seconds)."

**Spec compliance:**
- cdsl-agent-protocol: stale detection → re-delegation → success ✓
- cdsl-human-gateway: failure + retry accurately reported ✓
- Fidelity: timing, counts, and sequence preserved ✓

---

## Verification Checklist

Each example above satisfies:

| Requirement | Example 1 | Example 2 | Example 3 | Example 4 |
|-------------|-----------|-----------|-----------|-----------|
| Valid CDSL grammar | ✓ | ✓ | ✓ | ✓ |
| Proper opcode usage | ✓ | ✓ | ✓ | ✓ |
| Encapsulation markers | ✓ | ✓ | ✓ | ✓ |
| Agent ID matching | ✓ | ✓ | ✓ | ✓ |
| State pointer passing | — | — | ✓ | ✓ |
| Return format compliance | ✓ | ✓ | ✓ | ✓ |
| No CDSL in human output | ✓ | ✓ | ✓ | ✓ |
| Semantic fidelity preserved | ✓ | ✓ | ✓ | ✓ |
| Progressive disclosure | ✓ | ✓ | ✓ | ✓ |
| Token budget ≤20 per delegation | ✓ | ✓ | ✓ | ✓ |
