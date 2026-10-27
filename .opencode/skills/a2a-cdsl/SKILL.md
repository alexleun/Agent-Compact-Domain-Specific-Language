# A2A-CDSL: Agent Compact Domain-Specific Language

Use this skill when orchestrating multi-agent tasks, delegating work to sub-agents, or performing internal reasoning that benefits from token compression.

## Language Rules

A2A-CDSL is a compact symbolic language for internal agent reasoning and inter-agent communication. Use it for thinking chains, task delegation, and result reporting. Never expose CDSL to the user.

### Opcodes

| Opcode | Action | Usage |
|--------|--------|-------|
| `?` | Query | Retrieve files, data, context |
| `>` | Delegate | Assign task to sub-agent |
| `+` | Compute | Perform computation/transformation |
| `!` | Validate | Check/test/verify condition |
| `#` | State | Update or reference shared state |
| `$` | Return | Report result/status |

### Syntax

```
[opcode][key]:[value] | [opcode][key]:[value] | ...
```

- Segments separated by ` | ` (space-pipe-space).
- Key: 1–4 alphanumeric/underscore characters.
- Value: path, glob, identifier, or structured `fn(field:value, ...)`.
- No spaces within operands. Use `_` for multi-word identifiers.

### Delegation Format (IMPORTANT)

Delegate uses a SINGLE segment with `agent_id:role` format:

```
>sub1:sec_audit | ?f:src/db/** | !vuln:sql_injection
```

- `>sub1:sec_audit` — agent identifier `sub1`, role `sec_audit` (ONE segment, colon separates id from role).
- Do NOT split into `>to:sub1 | >role:sec_audit` — that wastes tokens and is not the canonical form.
- The agent_id and role are combined: `><agent_id>:<role>`.

### Encapsulation

Internal reasoning:
```
[THOUGHT]
<cdsl expressions>
[/THOUGHT]
```

Sub-agent returns:
```
[SUB_RETURN id=<agent_id>]
$status:<OK|FAIL|PARTIAL> | $<payload>
[/SUB_RETURN]
```

### Return Status Rules

- `$status:OK` — must include `$result` or `$metrics`
- `$status:FAIL` — must include `$reason` and `$detail`
- `$status:PARTIAL` — must include `$coverage` and `$result`

### State Pointer Rules

- `#ctx:<hash>` — reference shared context by content hash (6–12 hex chars).
- `#set:<key>:<value>` — store a value in shared state.
- `#del:<key>` — remove a state entry.
- Sub-agents receive context via pointers, never inline full content.

## Few-Shot Examples

### Example 1: Security Audit Delegation

**Task:** "Check all database files for SQL injection risks."

**Internal thinking (CDSL):**
```
[THOUGHT]
>sub1:sec_audit | ?f:src/db/** | !vuln:sql_injection
[/THOUGHT]
```

**Sub-agent return:**
```
[SUB_RETURN id=sub1]
$status:OK | $result:no_vulnerabilities_found | $metrics:files_scanned(42)
[/SUB_RETURN]
```

**Human output:** "A security audit was performed on all files under src/db/. No SQL injection vulnerabilities were found across 42 files scanned."

---

### Example 2: Multi-Agent Performance + Security Check

**Task:** "Diagnose database performance and check auth code for token leaks."

**Internal thinking (CDSL):**
```
[THOUGHT]
>sub_db:perf_opt | ?metrics:db_latency | +analyze:slow_query
>sub_sec:vulnerability | ?code:src/auth/ | !check:token_leak
[/THOUGHT]
```

**Sub-agent returns:**
```
[SUB_RETURN id=sub_db]
$status:OK | $bottleneck:idx_missing(tbl:users, col:email)
[/SUB_RETURN]

[SUB_RETURN id=sub_sec]
$status:OK | $result:no_vulnerabilities_found
[/SUB_RETURN]
```

**Human output:** "System diagnostics completed. A database performance bottleneck was identified: the users table is missing an index on the email column, causing query latency. A security audit of the authentication code found no token leak vulnerabilities."

---

### Example 3: Chained Task with State Pointer

**Task:** "Scan code for issues, then apply fixes based on findings."

**Internal thinking (CDSL):**
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

[SUB_RETURN id=sub_fix]
$status:OK | $result:fixes_applied(3)
[/SUB_RETURN]
```

**Human output:** "Code analysis completed. Three issues were identified and automatically fixed."

---

### Example 4: Failure with Retry (Error Recovery)

**Task:** "Run integration tests on the API module."

**Internal thinking (CDSL):**
```
[THOUGHT]
>sub_test:integration | !test:integration:api
[/THOUGHT]

[SUB_RETURN id=sub_test]
$status:FAIL | $reason:timeout | $detail:test_suite_exceeded_300s
[/SUB_RETURN]

[THOUGHT]
#task:stale | #id:task_test | #reason:timeout | #elapsed:300s
>sub_test_retry:integration | !test:integration:api | #set:timeout:600s
[/THOUGHT]

[SUB_RETURN id=sub_test_retry]
$status:OK | $result:all_tests_passed(28) | $metrics:duration(487s)
[/SUB_RETURN]
```

**Human output:** "The initial test run timed out after 300 seconds. A retry was dispatched with an extended timeout. All 28 integration tests passed on the second attempt (487 seconds)."

---

### Example 5: Partial Result Handling

**Task:** "Scan all Python files for type errors."

**Internal thinking (CDSL):**
```
[THOUGHT]
>sub_type:type_check | ?f:src/**/*.py | !check:type_errors
[/THOUGHT]

[SUB_RETURN id=sub_type]
$status:PARTIAL | $coverage:75 | $result:errors_found(12) | $remaining:src/legacy/**
[/SUB_RETURN]
```

**Human output:** "Type checking partially completed (75%). 12 type errors were found in the scanned files. The legacy directory (src/legacy/) was not covered."

---

### Example 6: Enrichment Loop (Iterative Refinement)

**Task:** "Analyze API performance — refine findings until root cause is clear."

**Internal thinking (CDSL):**
```
[THOUGHT]
>sub_api:perf_scan | ?metrics:api_latency | +analyze:slow_endpoints
[/THOUGHT]

[SUB_RETURN id=sub_api]
$status:PARTIAL | $coverage:60 | $result:suspicious_endpoints(3) | $remaining:auth_service
[/SUB_RETURN]

[THOUGHT]
>sub_api2:deep_analysis | #ctx:suspicious_endpoints | +analyze:auth_service | !check:connection_pool
[/THOUGHT]

[SUB_RETURN id=sub_api2]
$status:OK | $root_cause:pool_exhaustion(svc:auth, max_conn:10, active:10)
[/SUB_RETURN]
```

**Human output:** "API performance analysis required two passes. Initial scan identified 3 suspicious endpoints. Deep analysis found the root cause: the auth service connection pool is exhausted (10/10 connections active)."

## Decompression Rules

When presenting results to the user:
1. Translate all CDSL to natural language.
2. Never show opcodes, pipes, or bracket markers.
3. Default to brief summary (3–5 sentences).
4. Expand detail only when the user asks.
5. Preserve all semantic content exactly — no additions, no omissions.

## Token Budget

Target: ≤20 tokens per multi-agent delegation expression. Prefer short keys, glob patterns over file lists, and pointer references over inline context.

## Self-Check (MANDATORY before outputting CDSL)

Before emitting any CDSL expression, verify ALL of the following:

1. **No spaces in values** — replace every space with `_`. Example: `no_vulnerabilities_found` NOT `No vulnerabilities found`.
2. **Keys are 1–4 characters** — `$status` (6 chars) is INVALID. Use `$st` or the defined key `status` is an exception (5 chars, allowed as reserved key). All other keys MUST be ≤4 chars.
3. **Status uses reserved keys only** — use `$status:OK` NOT `$ok:...` or `$result:OK`.
4. **Required segments present** — `$status:OK` must be followed by `$result` or `$metrics`. `$status:FAIL` must include `$reason` AND `$detail`.
5. **No natural language in CDSL** — if you find yourself writing a sentence, stop and convert to `key:value` format.
6. **Delegation is single-segment** — use `>agent_id:role` NOT `>to:agent | >role:role`.

### Common Errors to Avoid

| WRONG | CORRECT | Why |
|-------|---------|-----|
| `$result:No SQL injection found` | `$result:no_sql_injection_found` | Spaces forbidden |
| `$ok:users.email.missing_index` | `$status:OK \| $bottleneck:idx_missing(tbl:users, col:email)` | Wrong key + wrong format |
| `$status:OK \| $result:found 3 issues` | `$status:OK \| $result:issues_found(3)` | Spaces + wrong structure |
| `?f:src db files` | `?f:src/db/**` | Spaces in path, no glob syntax |
| `>to:sub1 \| >role:sec_audit` | `>sub1:sec_audit` | Split delegation wastes tokens |

### Correct Generation Examples

**Reporting success:**
```
$status:OK | $result:no_vulnerabilities_found | $metrics:files_scanned(42)
```

**Reporting a specific finding:**
```
$status:OK | $bottleneck:idx_missing(tbl:users, col:email)
```

**Reporting failure:**
```
$status:FAIL | $reason:timeout | $detail:connection_refused_to_db_host
```

**Reporting partial:**
```
$status:PARTIAL | $coverage:75 | $result:3_of_4_files_scanned
```

**Reporting root cause (after enrichment loop):**
```
$status:OK | $root_cause:pool_exhaustion(svc:auth, max_conn:10, active:10)
```
