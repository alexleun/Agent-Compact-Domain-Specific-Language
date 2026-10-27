# A2A-CDSL Decompressor Rules

## Overview

The decompressor translates CDSL expressions and state into fluent natural-language text for human consumption. It is the final stage of the A2A pipeline: all CDSL content is internal; only decompressed output reaches the user.

## Core Principle

**No CDSL leakage.** Human-facing output MUST contain zero instances of:
- CDSL opcodes (`?`, `>`, `+`, `!`, `#`, `$`)
- Pipe separators (`|`)
- Bracket markers (`[THOUGHT]`, `[SUB_RETURN]`, `[/THOUGHT]`, `[/SUB_RETURN]`)
- Key:value syntax (`status:OK`, `f:src/db/**`)

## Translation Rules

### 1. Query Expressions → Informational Statements

| CDSL Pattern | Natural Language |
|-------------|-----------------|
| `?f:<glob>` | "Searched files matching <glob>" |
| `?d:<identifier>` | "Retrieved data for <identifier>" |
| `?c:<hash>` | "Accessed stored context" |
| `?s:<term>` | "Searched for <term>" |

**Example:**
```
Input:  ?f:src/db/**
Output: "Searched all files under src/db/"
```

---

### 2. Delegate Expressions → Action Descriptions

| CDSL Pattern | Natural Language |
|-------------|-----------------|
| `>sub_id:role` | "Assigned a <role> task to a specialized agent" |
| `>sub_id:role | ?<scope>` | "Assigned a <role> task covering <scope>" |

**Example:**
```
Input:  >sub1:sec_audit | ?f:src/db/**
Output: "A security audit agent was assigned to examine all files under src/db/"
```

---

### 3. Compute Expressions → Process Descriptions

| CDSL Pattern | Natural Language |
|-------------|-----------------|
| `+calc:<expr>` | "Computed <human-readable expression>" |
| `+xform:<name>` | "Applied transformation: <name>" |
| `+exec:<target>` | "Executed: <target>" |
| `+gen:<template>` | "Generated: <template>" |

**Example:**
```
Input:  +analyze:slow_query
Output: "Analyzed slow query patterns"
```

---

### 4. Validate Expressions → Check Results

| CDSL Pattern | Natural Language |
|-------------|-----------------|
| `!vuln:<name>` | "Checked for <name> vulnerability" |
| `!lint:<rule>` | "Verified code style: <rule>" |
| `!test:<id>` | "Ran test: <id>" |
| `!check:<cond>` | "Verified condition: <condition>" |

**Example:**
```
Input:  !vuln:sql_injection
Output: "Checked for SQL injection vulnerabilities"
```

---

### 5. State Expressions → Context Statements

| CDSL Pattern | Natural Language |
|-------------|-----------------|
| `#ctx:<hash>` | (resolved inline — the actual content is rendered) |
| `#set:<k>:<v>` | "Recorded: <key> = <value>" |
| `#del:<key>` | "Removed: <key>" |
| `#ref:<resource>` | "Referenced resource: <resource>" |

**Example:**
```
Input:  #ctx:a3f2b1  (resolves to "Database config: host=localhost")
Output: "Context: Database config: host=localhost"
```

---

### 6. Return Expressions → Result Reports

| CDSL Pattern | Natural Language |
|-------------|-----------------|
| `$status:OK` | "Completed successfully" |
| `$status:FAIL` | "Failed" |
| `$status:PARTIAL` | "Partially completed" |
| `$result:<findings>` | "Findings: <human-readable findings>" |
| `$reason:<code>` | "Reason: <human-readable reason>" |
| `$detail:<text>` | "Details: <text with underscores replaced by spaces>" |
| `$coverage:<n>` | "Completed <n>% of the task" |

**Example:**
```
Input:  $status:OK | $bottleneck:idx_missing(tbl:users, col:email)
Output: "The analysis completed successfully. The users table is missing an index on the email column, causing query latency."
```

---

## Structured Value Rendering

Structured values are expanded into natural sentences:

```
Input:  idx_missing(tbl:users, col:email)
Output: "The users table is missing an index on the email column."
```

**Rules:**
- `tbl` → "table"
- `col` → "column"
- `idx` → "index"
- `type` → "type"
- `impact` → "impact level"
- Generic fields: render as "<field_name>: <value>"

**Multi-field example:**
```
Input:  perf_issue(type:missing_index, table:users, column:email, impact:high)
Output: "A performance issue was identified: the users table is missing an index on the email column. Impact level: high."
```

---

## Composite Expression Decompression

When multiple segments form a compound expression, the decompressor produces a coherent paragraph:

```
Input:
>sub1:sec_audit | ?f:src/db/** | !vuln:sql_injection

Output:
"A security audit was performed on all files under src/db/. The audit checked for SQL injection vulnerabilities."
```

```
Input:
$status:FAIL | $reason:timeout | $detail:connection_refused_to_db_host

Output:
"The task failed due to a timeout. Details: connection refused to database host."
```

---

## Fidelity Rules

1. Every semantic element in the CDSL MUST appear in the output — nothing is omitted.
2. No information may be added that is not present in the CDSL.
3. Status values map exactly: `OK` → success language, `FAIL` → failure language, `PARTIAL` → partial language.
4. Numeric values (coverage percentages, counts) are preserved exactly.
5. Identifiers (file paths, table names, column names) are preserved exactly.
