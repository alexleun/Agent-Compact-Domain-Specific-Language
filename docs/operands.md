# A2A-CDSL Operand Encoding Rules

## Overview

Every CDSL segment consists of an opcode immediately followed by an operand. An operand is a `key:value` pair. This document defines the valid forms of keys and values.

## Key Rules

- Keys MUST be 1–4 characters.
- Allowed characters: `a-z`, `A-Z`, `0-9`, `_`.
- Keys MUST start with a letter.
- Keys are mnemonic identifiers that indicate the type of target.

### Standard Key Registry

| Key | Used with | Meaning |
|-----|-----------|---------|
| `f` | `?` | File glob pattern |
| `d` | `?` | Data lookup identifier |
| `c` | `?` | Context hash reference |
| `s` | `?` | Search term |
| `calc` | `+` | Computation expression |
| `xform` | `+` | Transformation identifier |
| `exec` | `+` | Execution target |
| `gen` | `+` | Generation template |
| `vuln` | `!` | Vulnerability name |
| `lint` | `!` | Lint rule identifier |
| `test` | `!` | Test identifier |
| `check` | `!` | Condition expression |
| `ctx` | `#` | Context pointer hash |
| `set` | `#` | State set operation |
| `del` | `#` | State delete operation |
| `ref` | `#` | Resource reference |
| `status` | `$` | Status enum |
| `result` | `$` | Structured findings |
| `reason` | `$` | Error code |
| `detail` | `$` | Diagnostic text |
| `coverage` | `$` | Completion percentage |

Custom keys MAY be defined by agents for domain-specific use, provided they follow the 1–4 character constraint.

## Value Forms

### 1. Simple Value

A single token: alphanumeric characters, dots, slashes, hyphens, underscores.

```
simple_value = [a-zA-Z0-9._/-]+
```

**Examples:**
```
?d:config.database.host
#ctx:a3f2b1
$reason:timeout
!check:token_expiry
```

**Rules:**
- No spaces (use `_` for multi-word identifiers).
- No `|` (reserved as segment separator).
- No `(` or `)` (reserved for structured values).

### 2. Glob Value

A file path pattern extended with wildcard characters.

```
glob_value = [a-zA-Z0-9._/-]+ [*/?]* 
```

Wildcards:
- `*` — matches any characters within a path segment
- `**` — matches any path segments (recursive)
- `?` — matches exactly one character (note: only valid within the value portion, not as an opcode)

**Examples:**
```
?f:src/**/*.ts
?f:src/db/**
?f:src/**/test_*.py
?f:src/?ata/*.sql
```

**Rules:**
- Glob patterns MUST start with a path-like prefix (contains `/` or `.`).
- `**` MUST be bounded by `/` or be at the end of the pattern.
- A bare `*` or `**` without a path prefix is invalid.

### 3. Structured Value

A parenthesized list of `key:value` fields for representing structured data.

```
structured_value = identifier "(" field (", " field)* ")"
field = key ":" simple_value
```

**Examples:**
```
$bottleneck:idx_missing(tbl:users, col:email)
+calc:sum(a:1, b:2, c:3)
!check:overflow(var:counter, max:255)
```

**Rules:**
- The identifier before `(` MUST follow key naming rules (1–4 chars for the key, but the structured identifier itself may be longer as it is the value, not the key).
- Fields within `()` are separated by `, ` (comma-space).
- Each field follows `key:value` format with the same key constraints.
- Nesting depth: maximum 1 level. Fields within a structured value MUST be simple values (no nested structured values).
- Empty structured values `()` are invalid.

## Disambiguation Rules

To ensure unambiguous parsing:

1. **Operand boundary**: An operand ends at the next ` | ` (space-pipe-space) or at the end of the expression/encapsulation block.
2. **Key boundary**: A key ends at the first `:` character.
3. **Structured value boundary**: A structured value ends at the first unmatched `)`.
4. **Glob vs simple**: If a value contains `*` or `**`, it is interpreted as a glob. If it contains `(`, it is interpreted as structured. Otherwise, it is simple.

## Multi-Word Identifiers

Use `_` (underscore) to join words:

```
!vuln:sql_injection
$detail:connection_refused
+xform:normalize_whitespace
```

## Token Efficiency Guidelines

- Prefer short keys (1–2 chars) for common operations.
- Use glob patterns rather than listing individual files.
- Use structured values only when multiple fields are needed; otherwise use simple values.
- Keep identifiers under 20 characters where possible.
