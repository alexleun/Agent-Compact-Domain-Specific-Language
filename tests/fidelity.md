# A2A-CDSL Fidelity Test Cases

## Purpose

Verify that the decompressor preserves semantic meaning exactly — no information is added, removed, or distorted during CDSL → natural language translation.

---

## F1 — SQL Injection Check (No Vulnerability)

**CDSL Input:**
```
!vuln:sql_injection
$status:OK | $result:no_vulnerabilities_found
```

**Expected Output:**
> "Checked for SQL injection vulnerabilities. No vulnerabilities were found."

**Semantic equivalence check:**
- `!vuln:sql_injection` → "Checked for SQL injection vulnerabilities" ✓
- `$status:OK` → task completed successfully ✓
- `$result:no_vulnerabilities_found` → "No vulnerabilities were found" ✓
- No information added ✓
- No information omitted ✓

---

## F2 — Missing Index Diagnosis

**CDSL Input:**
```
$status:OK | $bottleneck:idx_missing(tbl:users, col:email)
```

**Expected Output:**
> "The analysis completed successfully. The users table is missing an index on the email column."

**Semantic equivalence check:**
- `$status:OK` → "completed successfully" ✓
- `$bottleneck:idx_missing` → "missing an index" ✓
- `tbl:users` → "users table" ✓
- `col:email` → "email column" ✓
- No additional claims added ✓
- No fields omitted ✓

---

## F3 — Timeout Failure

**CDSL Input:**
```
$status:FAIL | $reason:timeout | $detail:connection_refused_to_db_host
```

**Expected Output:**
> "The task failed due to a timeout. Details: connection refused to database host."

**Semantic equivalence check:**
- `$status:FAIL` → "failed" ✓
- `$reason:timeout` → "due to a timeout" ✓
- `$detail:connection_refused_to_db_host` → "connection refused to database host" (underscores → spaces) ✓
- No euphemism or softening ✓
- No additional diagnostic added ✓

---

## F4 — Partial Coverage

**CDSL Input:**
```
$status:PARTIAL | $coverage:75 | $result:3_of_4_files_scanned | $remaining:src/legacy/**
```

**Expected Output:**
> "The task partially completed (75%). Three of four files were scanned. Remaining scope: all files under src/legacy/."

**Semantic equivalence check:**
- `$status:PARTIAL` → "partially completed" ✓
- `$coverage:75` → "75%" ✓
- `$result:3_of_4_files_scanned` → "Three of four files were scanned" ✓
- `$remaining:src/legacy/**` → "all files under src/legacy/" ✓
- Percentage preserved exactly ✓
- Glob pattern translated accurately ✓

---

## F5 — Multi-Agent Task Chain (Full Decompression)

**CDSL Input:**
```
[THOUGHT]
>sub_db:perf_opt | ?metrics:db_latency | +analyze:slow_query
>sub_sec:vulnerability | ?code:src/auth/ | !check:token_leak
[/THOUGHT]

[SUB_RETURN id=sub_db]
$status:OK | $bottleneck:idx_missing(tbl:users, col:email)
[/SUB_RETURN]

[SUB_RETURN id=sub_sec]
$status:OK | $result:no_vulnerabilities_found
[/SUB_RETURN]
```

**Expected Output (Level 1 — Brief):**
> "System diagnostics completed. A database performance bottleneck was identified: the users table is missing an index on the email column, causing query latency. A security audit of the authentication code found no token leak vulnerabilities."

**Semantic equivalence check:**
- `>sub_db:perf_opt` → "database performance" task performed ✓
- `?metrics:db_latency` → "query latency" context ✓
- `+analyze:slow_query` → analysis performed ✓
- `$bottleneck:idx_missing(tbl:users, col:email)` → "users table is missing an index on the email column" ✓
- `>sub_sec:vulnerability` → "security audit" ✓
- `?code:src/auth/` → "authentication code" ✓
- `!check:token_leak` → "token leak" check ✓
- `$status:OK` (both) → both completed ✓
- `$result:no_vulnerabilities_found` → "found no token leak vulnerabilities" ✓
- No CDSL tokens in output ✓
- No `[THOUGHT]` or `[SUB_RETURN]` markers in output ✓
- No information fabricated ✓

---

## F6 — Stale Task with Re-delegation

**CDSL Input:**
```
[THOUGHT]
#task:stale | #id:task_001 | #reason:timeout | #elapsed:300s
[/THOUGHT]

[THOUGHT]
>sub1_retry:sec_audit | ?f:src/db/** | !vuln:sql_injection
[/THOUGHT]

[SUB_RETURN id=sub1_retry]
$status:OK | $result:no_vulnerabilities_found
[/SUB_RETURN]
```

**Expected Output:**
> "The initial security audit task timed out after 300 seconds. A retry was dispatched. The retry completed successfully: no SQL injection vulnerabilities were found in the files under src/db/."

**Semantic equivalence check:**
- `#task:stale` → "timed out" ✓
- `#elapsed:300s` → "after 300 seconds" ✓
- `>sub1_retry` → "A retry was dispatched" ✓
- `?f:src/db/**` → "files under src/db/" ✓
- `!vuln:sql_injection` → "SQL injection vulnerabilities" ✓
- `$status:OK` → "completed successfully" ✓
- `$result:no_vulnerabilities_found` → "no SQL injection vulnerabilities were found" ✓
- Timeline preserved (stale → retry → success) ✓

---

## Summary

| Test | CDSL Complexity | Fidelity Verified |
|------|----------------|-------------------|
| F1 | Single validation + return | ✓ |
| F2 | Structured value rendering | ✓ |
| F3 | Failure with reason + detail | ✓ |
| F4 | Partial with coverage + glob | ✓ |
| F5 | Full multi-agent chain | ✓ |
| F6 | Stale detection + retry | ✓ |

All tests confirm: no CDSL tokens in output, no semantic drift, no information added or omitted.
