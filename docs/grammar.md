# A2A-CDSL Formal Grammar

## Overview

A2A-CDSL is a flat, pipe-delimited symbolic language. Expressions are sequences of segments separated by `|`. Each segment is an opcode followed by an operand.

## BNF Grammar

```bnf
<expression>       ::= <segment> ( " | " <segment> )*

<segment>          ::= <opcode> <operand>

<opcode>           ::= "?" | ">" | "+" | "!" | "#" | "$"

<operand>          ::= <key> ":" <value>

<key>              ::= <alpha> { <alpha> | <digit> | "_" }
                       (* 1–4 characters *)

<value>            ::= <simple_value>
                     | <glob_value>
                     | <structured_value>

<simple_value>     ::= <alpha> { <alpha> | <digit> | "_" | "." | "/" | "-" }

<glob_value>       ::= <simple_value> { <simple_value> | "*" | "**" | "?" }

<structured_value> ::= <identifier> "(" <field_list> ")"

<field_list>       ::= <field> ( ", " <field> )*

<field>            ::= <key> ":" <simple_value>

<identifier>       ::= <alpha> { <alpha> | <digit> | "_" }

<alpha>            ::= "a"-"z" | "A"-"Z"

<digit>            ::= "0"-"9"
```

## Production Rules

### Expression

An expression is one or more segments joined by ` | ` (space-pipe-space). A single-segment expression is valid.

```
>sub1:sec_audit | ?f:src/db/** | !vuln:sql_injection
```

### Segment

Each segment is exactly one opcode character immediately followed by its operand (no space between opcode and operand).

```
?f:src/db/**
>sub1:sec_audit
$status:OK
```

### Opcode

Six reserved single-character opcodes:

| Opcode | Category | Meaning |
|--------|----------|---------|
| `?` | Query | Retrieve information |
| `>` | Delegate | Create/assign sub-agent task |
| `+` | Compute | Perform computation/transformation |
| `!` | Validate | Check/test/verify condition |
| `#` | State | Update or reference shared state |
| `$` | Return | Report result/status |

### Operand

An operand is a `key:value` pair:

- **key**: 1–4 alphanumeric/underscore characters (mnemonic identifier)
- **value**: one of three forms:
  - **simple**: alphanumeric with dots, slashes, hyphens (identifiers, paths)
  - **glob**: simple value extended with `*`, `**`, `?` wildcards
  - **structured**: `identifier(field:value, field:value, ...)` for nested data

### Encapsulation Markers

CDSL content is wrapped in boundary markers for context isolation:

```
[THOUGHT]
<cdsl expressions>
[/THOUGHT]

[SUB_RETURN id=<agent_id>]
<cdsl expressions>
[/SUB_RETURN]
```

These markers are NOT part of the expression grammar — they are transport-level framing.

## Validity Rules

1. Every segment MUST begin with a defined opcode.
2. Every opcode MUST be followed by a valid operand.
3. Operand keys MUST be 1–4 characters.
4. Segments MUST be separated by ` | ` (space-pipe-space).
5. No segment may contain a bare `|` within its operand (pipe is reserved as separator).
6. Structured values use `()` for nesting; maximum nesting depth is 1 level (fields within a structured value cannot themselves be structured values).
7. Whitespace within operands is not permitted (use `_` for multi-word identifiers).

## Token Budget

A typical multi-agent delegation expression (3 segments) MUST fit within 20 tokens in standard BPE tokenizers. The grammar is designed so that:
- Each opcode = 1 token
- Each key = 1–2 tokens
- Each simple value = 1–3 tokens
- Pipe separators = 1 token each

## Extensibility

New opcodes MAY be added via a versioned extension registry. Adding an opcode MUST NOT invalidate any previously valid expression. Reserved opcode characters are immutable within a version.
