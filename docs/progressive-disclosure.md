# A2A-CDSL Progressive Disclosure

## Overview

The human gateway supports graduated detail levels. By default, users receive a brief summary. Expanded detail is available on request. This prevents information overload while preserving full access to underlying findings.

## Disclosure Levels

### Level 1 — Brief Summary (Default)

**Trigger:** Task completes, no detail level specified.

**Output format:** 3–5 sentences covering:
- What was done (high-level actions)
- Key findings (most important results only)
- Overall status (success / partial / failure)

**Rules:**
- Omit intermediate reasoning steps.
- Omit per-agent breakdown.
- Omit structured data details unless critical.
- Use plain language, no jargon.
- MUST still be self-contained (no references to internal state).

**Example:**

Given this CDSL chain:
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

**Level 1 output:**
> "System diagnostics completed. A database performance bottleneck was identified: the users table is missing an index on the email column, causing query latency. A security audit of the authentication code found no token leak vulnerabilities."

---

### Level 2 — Expanded Detail (On Request)

**Trigger:** User explicitly requests more detail (e.g., "show details", "explain further", "what did each agent find?").

**Output format:** Full report including:
- Per-agent findings with individual results
- Intermediate reasoning steps
- Structured data rendered in full
- File paths, identifiers, metrics
- Task lifecycle information (what succeeded, what failed, timing)

**Rules:**
- Include all findings from all sub-agents.
- Render structured values completely.
- Include quantitative data (counts, percentages, timings).
- Organize by agent/task for clarity.
- Still no CDSL tokens in output.

**Example (same CDSL chain as above):**

> "Two specialized agents were assigned:
>
> **Database Performance Agent:**
> - Searched database latency metrics and analyzed slow queries.
> - Finding: The `users` table is missing an index on the `email` column. This causes full table scans on every email-based query, resulting in elevated latency.
> - Status: Completed successfully.
>
> **Security Audit Agent:**
> - Examined all files under `src/auth/` for token leak vulnerabilities.
> - Finding: No token leak vulnerabilities detected.
> - Status: Completed successfully.
>
> Both tasks completed without errors."

---

### Level 3 — Full Trace (Debug Mode)

**Trigger:** User requests full trace, or debugging mode is enabled.

**Output format:** Complete reconstruction including:
- Every CDSL expression translated to annotated natural language
- Full task lifecycle timeline
- State pointer contents resolved and shown inline
- All intermediate results, including partial and failed attempts

**Rules:**
- Annotate each step with what it did and why.
- Show timing information for each task.
- Include stale/retry events if any.
- Still no raw CDSL tokens — everything is translated.

---

## Level Selection Logic

| Condition | Level |
|-----------|-------|
| Default (no user preference) | Level 1 |
| User asks "what happened?" or "show details" | Level 2 |
| User asks "full trace" or debug mode enabled | Level 3 |
| Task had failures or partial results | Level 1 includes failure summary; Level 2 includes full failure analysis |

## Fidelity Across Levels

- Level 1 MUST NOT contradict Level 2 or Level 3.
- Level 1 is a strict subset of Level 2 (same facts, fewer details).
- Level 2 is a strict subset of Level 3 (same facts, fewer annotations).
- No level adds information not present in the CDSL state.
- No level omits critical failures (even Level 1 must mention failures).

## Progressive Disclosure in Practice

```
User: "Check my database for performance issues and security vulnerabilities."

Agent (internal):
  [THOUGHT] ... CDSL ... [/THOUGHT]
  [SUB_RETURN] ... CDSL ... [/SUB_RETURN]

Gateway → Level 1 output:
  "System diagnostics completed. A database performance bottleneck was identified..."

User: "What specifically was wrong with the database?"

Gateway → Level 2 output:
  "The `users` table is missing an index on the `email` column..."
```
