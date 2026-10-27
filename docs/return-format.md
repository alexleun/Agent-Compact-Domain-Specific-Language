# A2A-CDSL Result Return Format

## Overview

Sub-agents report results to the master agent using `$` opcode expressions enclosed in `[SUB_RETURN]` markers. All returns follow a strict status + payload structure.

## Return Structure

```
[SUB_RETURN id=<agent_id>]
$status:<OK|FAIL|PARTIAL> | $<payload_segments>
[/SUB_RETURN]
```

## Status Values

| Status | Meaning | When to use |
|--------|---------|-------------|
| `OK` | Task completed successfully | All requested work finished |
| `FAIL` | Task could not be completed | Error, timeout, or impossible condition |
| `PARTIAL` | Task partially completed | Timeout, partial coverage, or degraded results |

## Payload Formats

### Success (`$status:OK`)

Required segments:
- `$status:OK`
- `$result:<findings>` — structured or simple value describing findings

Optional segments:
- `$metrics:<data>` — quantitative results (counts, scores, timings)
- `$files:<list>` — affected file paths (glob or structured)

**Example:**
```
[SUB_RETURN id=sub_db]
$status:OK | $bottleneck:idx_missing(tbl:users, col:email)
[/SUB_RETURN]
```

**Example (security audit):**
```
[SUB_RETURN id=sub_sec]
$status:OK | $result:no_vulnerabilities_found | $metrics:files_scanned(42)
[/SUB_RETURN]
```

---

### Failure (`$status:FAIL`)

Required segments:
- `$status:FAIL`
- `$reason:<error_code>` — machine-readable error identifier
- `$detail:<diagnostic>` — human-readable explanation (underscore-separated words)

Optional segments:
- `$context:<hash>` — pointer to state at time of failure

**Example:**
```
[SUB_RETURN id=sub_db]
$status:FAIL | $reason:timeout | $detail:connection_refused_to_db_host
[/SUB_RETURN]
```

**Example (permission error):**
```
[SUB_RETURN id=sub_sec]
$status:FAIL | $reason:permission_denied | $detail:cannot_read_src_auth_config
[/SUB_RETURN]
```

---

### Partial (`$status:PARTIAL`)

Required segments:
- `$status:PARTIAL`
- `$coverage:<percentage>` — integer 0–100 indicating completion percentage
- `$result:<partial_findings>` — whatever was completed

Optional segments:
- `$reason:<limitation>` — why the task could not complete fully
- `$remaining:<scope>` — what was not covered

**Example:**
```
[SUB_RETURN id=sub_scan]
$status:PARTIAL | $coverage:75 | $result:3_of_4_files_scanned | $reason:timeout | $remaining:src/legacy/**
[/SUB_RETURN]
```

---

## Structured Result Values

When findings contain multiple fields, use structured value syntax:

```
$result:<identifier>(<field>:<value>, <field>:<value>, ...)
```

**Example:**
```
$result:perf_issue(type:missing_index, table:users, column:email, impact:high)
```

**Rules:**
- Maximum 1 nesting level (fields are simple values only).
- Maximum 8 fields per structured value.
- Field keys follow the 1–4 character constraint.

---

## Return Validation Rules

1. `$status` MUST be the first segment in any return expression.
2. `$status` value MUST be exactly `OK`, `FAIL`, or `PARTIAL` (case-sensitive).
3. A return with `$status:OK` MUST include at least one `$result` or `$metrics` segment.
4. A return with `$status:FAIL` MUST include both `$reason` and `$detail`.
5. A return with `$status:PARTIAL` MUST include `$coverage` and `$result`.
6. The `[SUB_RETURN id=<agent_id>]` marker MUST match the agent_id from the original `>` delegation.

## Invalid Return Examples

```
$status:UNKNOWN | $result:done
```
**Invalid:** `UNKNOWN` is not a valid status.

```
$result:found_issues
```
**Invalid:** Missing required `$status` segment.

```
$status:FAIL | $reason:timeout
```
**Invalid:** Missing required `$detail` segment for FAIL status.

```
$status:PARTIAL | $result:partial
```
**Invalid:** Missing required `$coverage` segment for PARTIAL status.
