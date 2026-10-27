# A2A-CDSL Opcode Registry

Version: 1.0

## Core Opcodes

### `?` — Query

**Category:** Information retrieval  
**Meaning:** Retrieve information — file search, data lookup, context fetch.

**Operand format:** `?<key>:<value>`

| Key | Meaning | Value type |
|-----|---------|-----------|
| `f` | File glob search | glob pattern |
| `d` | Data lookup | identifier |
| `c` | Context fetch | hash reference |
| `s` | Search by content | search term |

**Valid examples:**
```
?f:src/db/**
?d:config.database.host
?c:a3f2b1
?s:SQL injection
```

**Invalid examples:**
```
? :f:src/db/**        ← space between opcode and operand
?f:                   ← empty value
?xyzw:target          ← key exceeds 4 characters
```

---

### `>` — Delegate

**Category:** Task delegation  
**Meaning:** Create or assign a task to a sub-agent.

**Operand format:** `><agent_id>:<role>`

| Component | Meaning | Format |
|-----------|---------|--------|
| `agent_id` | Sub-agent identifier | alphanumeric, 1–16 chars |
| `role` | Task role/scope | alphanumeric with underscores |

**Valid examples:**
```
>sub1:sec_audit
>db_opt:performance_analysis
>agent42:code_review
```

**Invalid examples:**
```
>sub1:              ← empty role
>:generic           ← empty agent_id
>sub1:role with spaces  ← spaces not allowed
```

---

### `+` — Compute

**Category:** Computation / transformation  
**Meaning:** Perform a computation, transformation, or program execution.

**Operand format:** `+<key>:<value>`

| Key | Meaning | Value type |
|-----|---------|-----------|
| `calc` | Arithmetic/logic computation | expression |
| `xform` | Data transformation | transform identifier |
| `exec` | Program/code execution | code reference |
| `gen` | Generate output | template identifier |

**Valid examples:**
```
+calc:sum(1,2,3)
+xform:normalize_whitespace
+exec:run_tests:unit
+gen:report_summary
```

**Invalid examples:**
```
+ :calc:sum(1)      ← space between opcode and operand
+calc:              ← empty value
```

---

### `!` — Validate

**Category:** Verification / testing  
**Meaning:** Check, test, or verify a condition.

**Operand format:** `!<key>:<value>`

| Key | Meaning | Value type |
|-----|---------|-----------|
| `vuln` | Security vulnerability check | vulnerability name |
| `lint` | Code style/lint check | rule identifier |
| `test` | Test execution | test identifier |
| `check` | General condition check | condition expression |

**Valid examples:**
```
!vuln:sql_injection
!lint:no_unused_imports
!test:integration:api
!check:token_expiry
```

**Invalid examples:**
```
!vuln:              ← empty value
! :vuln:xss         ← space between opcode and operand
```

---

### `#` — State

**Category:** Shared state / memory  
**Meaning:** Update shared state, reference state, or pass context pointers.

**Operand format:** `#<key>:<value>`

| Key | Meaning | Value type |
|-----|---------|-----------|
| `ctx` | Context pointer reference | hash identifier |
| `set` | Set state value | key=value pair |
| `del` | Delete state entry | key identifier |
| `ref` | Reference shared resource | resource identifier |

**Valid examples:**
```
#ctx:a3f2b1
#set:task_status:in_progress
#del:temp_cache
#ref:project_config
```

**Invalid examples:**
```
#ctx:               ← empty hash reference
# :set:x:1          ← space between opcode and operand
```

---

### `$` — Return

**Category:** Result reporting  
**Meaning:** Report results, status, or findings back to the calling agent.

**Operand format:** `$<key>:<value>`

| Key | Meaning | Value type |
|-----|---------|-----------|
| `status` | Task status | `OK` / `FAIL` / `PARTIAL` |
| `result` | Structured findings | structured value or identifier |
| `reason` | Error/failure code | error identifier |
| `detail` | Brief diagnostic | diagnostic text (underscore-separated) |
| `coverage` | Completion percentage | numeric value |

**Valid examples:**
```
$status:OK
$status:FAIL | $reason:timeout | $detail:connection_refused
$status:PARTIAL | $coverage:75 | $result:partial_findings
$bottleneck:idx_missing(tbl:users, col:email)
```

**Invalid examples:**
```
$status:UNKNOWN     ← status must be OK, FAIL, or PARTIAL
$ :status:OK        ← space between opcode and operand
```

---

## Extension Registry (Reserved for Future Versions)

| Opcode | Category | Status |
|--------|----------|--------|
| `@` | Cache | Reserved v2.0 |
| `~` | Notify/Event | Reserved v2.0 |

Extensions MUST NOT change the semantics of existing opcodes. Existing valid expressions remain valid across versions.
