# A2A-CDSL Validity Test Suite

## Valid Expressions

Each case below MUST be accepted by the grammar parser.

### V1 — Single query with glob
```
?f:src/db/**
```
**Expected:** Valid. Query opcode `?`, key `f`, glob value `src/db/**`.

---

### V2 — Single delegate with role
```
>sub1:sec_audit
```
**Expected:** Valid. Delegate opcode `>`, agent_id `sub1`, role `sec_audit`.

---

### V3 — Multi-segment compound expression
```
>sub1:sec_audit | ?f:src/db/** | !vuln:sql_injection
```
**Expected:** Valid. Three segments: delegate, query, validate.

---

### V4 — Return with status and structured result
```
$status:OK | $bottleneck:idx_missing(tbl:users, col:email)
```
**Expected:** Valid. Two segments: status return, structured diagnostic.

---

### V5 — State pointer reference
```
#ctx:a3f2b1
```
**Expected:** Valid. State opcode `#`, key `ctx`, simple value (hash).

---

### V6 — Compute with structured fields
```
+calc:sum(a:1, b:2, c:3)
```
**Expected:** Valid. Compute opcode `+`, key `calc`, structured value with three fields.

---

### V7 — Validate with test target
```
!test:integration:api
```
**Expected:** Valid. Validate opcode `!`, key `test`, value `integration:api` (simple value with colon in value portion — the first `:` separates key from value, remaining colons are part of the value).

---

### V8 — Parallel multi-delegate
```
>agent1:code_review | >agent2:security_scan | >agent3:perf_check
```
**Expected:** Valid. Three delegate segments in parallel.

---

### V9 — Partial return with coverage
```
$status:PARTIAL | $coverage:75 | $result:partial_findings
```
**Expected:** Valid. Three return segments: status, coverage, result.

---

### V10 — Full encapsulated thought block
```
[THOUGHT]
#task_init
>sub_db:perf_opt | ?metrics:db_latency | +analyze:slow_query
>sub_sec:vulnerability | ?code:src/auth/ | !check:token_leak
[/THOUGHT]
```
**Expected:** Valid. Encapsulation markers frame two multi-segment expressions. `#task_init` is a state segment with key `task_init`... wait — key exceeds 4 characters. Correction: `#task:init` (key `task`, value `init`).

**Corrected:**
```
[THOUGHT]
#task:init
>sub_db:perf_opt | ?metrics:db_latency | +analyze:slow_query
>sub_sec:vulnerability | ?code:src/auth/ | !check:token_leak
[/THOUGHT]
```
**Expected:** Valid.

---

### V11 — Glob with single-char wildcard
```
?f:src/?ata/*.sql
```
**Expected:** Valid. Glob value with `?` wildcard matching one character in `?ata`.

---

### V12 — Failure return with reason and detail
```
$status:FAIL | $reason:timeout | $detail:connection_refused
```
**Expected:** Valid. Three return segments.

---

## Invalid Expressions

Each case below MUST be rejected by the grammar parser.

### I1 — Undefined opcode
```
@f:src/db/**
```
**Expected:** Invalid. `@` is not a defined opcode in v1.0.  
**Error:** `INVALID_OPCODE: '@' at position 0`

---

### I2 — Space between opcode and operand
```
? :f:src/db/**
```
**Expected:** Invalid. Space between `?` and operand.  
**Error:** `SYNTAX_ERROR: unexpected space after opcode at position 1`

---

### I3 — Empty value
```
?f:
```
**Expected:** Invalid. Operand has empty value.  
**Error:** `SYNTAX_ERROR: empty value in operand at position 3`

---

### I4 — Key exceeds 4 characters
```
?xyzw:target
```
**Expected:** Invalid. Key `xyzw` is 4 characters — actually valid. Let me use 5:  
```
?xyzwv:target
```
**Expected:** Invalid. Key `xyzwv` is 5 characters, exceeds maximum.  
**Error:** `SYNTAX_ERROR: key 'xyzwv' exceeds 4 characters at position 1`

---

### I5 — Missing operand entirely
```
?
```
**Expected:** Invalid. Opcode with no operand.  
**Error:** `SYNTAX_ERROR: missing operand after opcode at position 0`

---

### I6 — Bare pipe separator
```
?f:src/db/** || !vuln:xss
```
**Expected:** Invalid. Double pipe `||` is not a valid separator (must be ` | `).  
**Error:** `SYNTAX_ERROR: invalid separator '||' at position 13`

---

### I7 — Unmatched parenthesis in structured value
```
$bottleneck:idx_missing(tbl:users, col:email
```
**Expected:** Invalid. Missing closing `)`.  
**Error:** `SYNTAX_ERROR: unmatched '(' at position 12`

---

### I8 — Nested structured value (depth > 1)
```
+calc:outer(a:inner(b:1))
```
**Expected:** Invalid. Structured value nested inside a field. Maximum depth is 1.  
**Error:** `SYNTAX_ERROR: nested structured value at position 14`

---

### I9 — Space within operand value
```
!vuln:sql injection
```
**Expected:** Invalid. Space within value portion.  
**Error:** `SYNTAX_ERROR: unexpected space in value at position 8`

---

### I10 — Key starts with digit
```
?1abc:target
```
**Expected:** Invalid. Key `1abc` starts with a digit.  
**Error:** `SYNTAX_ERROR: key must start with a letter at position 1`

---

## Summary

| Category | Count |
|----------|-------|
| Valid expressions | 12 |
| Invalid expressions | 10 |

All cases are traceable to specific grammar rules in `docs/grammar.md` and `docs/operands.md`.
