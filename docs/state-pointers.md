# A2A-CDSL State Pointer Passing

## Overview

Agents share context and state through compact hash-based pointers rather than duplicating full content in each message. This reduces token usage and enables deduplication.

## Pointer Format

```
#ctx:<hash_id>
```

- `#` — state opcode
- `ctx` — key indicating a context pointer reference
- `<hash_id>` — short content hash (6–12 hex characters) identifying the referenced state

## Hash Generation

Hash identifiers are generated from content:

1. Compute a short hash (e.g., first 6–12 hex chars of SHA-256) of the content being referenced.
2. The hash serves as a unique, deduplicable identifier.
3. Same content → same hash → same pointer (deduplication).

**Example:**
```
Content: "Database config: host=localhost, port=5432, db=production"
Hash:    #ctx:a3f2b1c8d4e0
```

## Pointer Operations

### Setting State

```
#set:<key>:<value>
```

Stores a value in shared state under a key.

**Example:**
```
#set:task_status:in_progress
#set:target_files:src/db/**
```

### Referencing State

```
#ctx:<hash_id>
```

References previously stored content by its hash.

**Example:**
```
#ctx:a3f2b1
```

### Deleting State

```
#del:<key>
```

Removes a state entry.

**Example:**
```
#del:temp_cache
```

### Referencing Shared Resources

```
#ref:<resource_id>
```

References an external resource (file, config, API endpoint) by identifier.

**Example:**
```
#ref:project_config
#ref:db_connection_string
```

## Resolution Semantics

When a sub-agent receives a `#ctx:<hash_id>` reference:

1. The agent runtime resolves the hash to the actual stored content.
2. The resolved content is made available to the sub-agent as context.
3. The sub-agent does NOT need to request the content again — resolution is automatic.

**Resolution flow:**
```
Master emits:  >sub1:task | #ctx:a3f2b1
Runtime:       resolves a3f2b1 → actual content
Sub-agent:     receives content as part of its context window
```

## Pointer Rules

1. Hash identifiers MUST be 6–12 hexadecimal characters (`[0-9a-f]`).
2. Pointers are read-only from the sub-agent's perspective — sub-agents MUST NOT modify referenced state.
3. A pointer reference is valid only if the content was previously stored via `#set` or was produced by a prior `$result`.
4. Pointers MUST NOT be used to reference state that has been deleted via `#del`.
5. Maximum pointer chain depth: 2 (a pointer may reference content that itself contains pointers, but not deeper).

## State Lifecycle

```
created (#set) → referenced (#ctx) → deleted (#del)
```

State entries persist within a single task chain session. They are NOT persistent across sessions.

## Token Efficiency

| Approach | Tokens (approx) |
|----------|----------------|
| Inline full context (500 chars) | ~150 tokens |
| Pointer reference `#ctx:a3f2b1` | ~4 tokens |
| **Savings** | **~97%** |

## Examples

### Master shares context with sub-agent
```
[THOUGHT]
#set:audit_scope:src/db/**
>sub1:sec_audit | #ctx:a3f2b1 | !vuln:sql_injection
[/THOUGHT]
```

### Sub-agent references prior findings
```
[THOUGHT]
>sub2:refactor | #ctx:findings_a7f3 | +exec:apply_fixes
[/THOUGHT]
```

### Cleanup after task completion
```
[THOUGHT]
#del:audit_scope | #del:temp_cache
[/THOUGHT]
```

## Invalid Pointer Examples

```
#ctx:xyz          ← hash too short (minimum 6 chars)
#ctx:GHIJKLMNOP   ← non-hex characters
#ctx:             ← empty hash
#ctx:a3f2b1:extra ← extra content after hash
```
