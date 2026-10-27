# A2A-CDSL

**Agent Compact Domain-Specific Language** — a super-compressed symbolic language for AI agent internal reasoning and inter-agent communication.

Reduce multi-agent task delegation token usage by **65–70%** while preserving full semantic fidelity.

---

## What Is This?

A2A-CDSL replaces verbose natural-language agent instructions with compact symbolic expressions:

```
# Instead of this (35-50 tokens):
"Please create a security audit sub-agent that searches all files under src/db/
 and checks them for SQL injection vulnerabilities. Report the results back to me."

# Use this (12 tokens):
>sub1:sec_audit | ?f:src/db/** | !vuln:sql_injection
```

The language is designed for **AI-to-AI communication only**. A decompressor layer translates final results into natural language for human users.

---

## Quick Start

### 1. As an OpenCode Skill

Place the skill file in your project:

```
.opencode/skills/a2a-cdsl/SKILL.md
```

The skill is automatically available to agents in your OpenCode project.

### 2. As a System Prompt (Any Framework)

Copy the content from `docs/prompt-template.md` and inject it into your agent's system prompt. Works with:
- OpenCode (skill file)
- LangChain / LangGraph (SystemMessage)
- AutoGen (system_message parameter)
- CrewAI (agent role/backstory)
- Direct API (prepend to system role)

### 3. Grammar Reference

See `docs/grammar.md` for the full BNF specification.

---

## Language Overview

### Opcodes

| Opcode | Action | Example |
|--------|--------|---------|
| `?` | Query | `?f:src/db/**` — search files |
| `>` | Delegate | `>sub1:sec_audit` — assign task |
| `+` | Compute | `+analyze:slow_query` — run analysis |
| `!` | Validate | `!vuln:sql_injection` — check vulnerability |
| `#` | State | `#ctx:a3f2b1` — reference shared context |
| `$` | Return | `$status:OK \| $result:no_issues` — report result |

### Syntax

```
[opcode][key]:[value] | [opcode][key]:[value] | ...
```

- Segments separated by ` | ` (space-pipe-space)
- Keys: 1–4 characters (alphanumeric + underscore)
- Values: paths, globs, identifiers (no spaces — use `_`)

### Encapsulation

```
[THOUGHT]
>sub1:sec_audit | ?f:src/db/** | !vuln:sql_injection
[/THOUGHT]

[SUB_RETURN id=sub1]
$status:OK | $result:no_vulnerabilities_found
[/SUB_RETURN]
```

---

## Project Structure

```
a2a-cdsl/
├── README.md                          ← You are here
├── LICENSE                            ← MIT License
├── .opencode/
│   └── skills/
│       └── a2a-cdsl/
│           └── SKILL.md              ← Production skill module (main deliverable)
├── docs/
│   ├── grammar.md                    ← Formal BNF grammar
│   ├── opcodes.md                    ← Complete opcode registry
│   ├── operands.md                   ← Key/value encoding rules
│   ├── delegation.md                 ← Task delegation protocol
│   ├── return-format.md              ← Result return structures
│   ├── state-pointers.md             ← Hash-based context sharing
│   ├── lifecycle.md                  ← Task state machine
│   ├── decompressor.md               ← CDSL → natural language rules
│   ├── progressive-disclosure.md     ← 3-level output detail control
│   ├── prompt-template.md            ← Framework-agnostic injection guide
│   ├── usage-guide.md                ← End-to-end examples
│   └── token-benchmark.md            ← Quantified token savings
├── tests/
│   ├── cdsl-validity.md              ← 12 valid + 10 invalid expressions
│   ├── fidelity.md                   ← Semantic equivalence tests
│   └── integration-validation.md     ← Cross-spec coherence check
└── reports/
    ├── review-report-en.md           ← Full review (English)
    └── review-report-zh.md           ← Full review (Chinese)
```

---

## Measured Results

| Metric | Result |
|--------|--------|
| Token reduction (input) | 65–70% |
| Tokens per delegation | 12–18 (target: ≤20) |
| Processing speed | 5× faster (with cached grammar) |
| Comprehension accuracy | 100% |
| Generation grammar validity | 100% (with self-check prompt) |
| Fidelity preservation | 100% |
| Break-even point | 28 delegations |

Tested on: Qwen3.8-flash-Next (llama.cpp). See `reports/` for full methodology.

---

## How to Use

### For Agent Developers

1. Read `docs/usage-guide.md` for end-to-end examples
2. Copy `.opencode/skills/a2a-cdsl/SKILL.md` into your project (OpenCode)
3. Or copy `docs/prompt-template.md` into your agent's system prompt (any framework)
4. Use `docs/opcodes.md` as reference while writing agent prompts

### For Language Designers

1. Start with `docs/grammar.md` for the formal specification
2. Review `docs/operands.md` for encoding rules
3. Check `tests/cdsl-validity.md` for valid/invalid examples

### For Evaluators

1. Read `reports/review-report-en.md` (or `-zh.md`) for the full assessment
2. Review `docs/token-benchmark.md` for quantified savings
3. Check `tests/` for validation evidence

---

## Design Principles

1. **Micro opcodes** — single ASCII characters replace verb phrases
2. **Keyless positional schema** — drop verbose JSON keys
3. **State pointers** — pass hashes, not repeated context
4. **Flat structure** — pipe-delimited, no nesting (LLM-friendly)
5. **Prompt-injection only** — no fine-tuning required
6. **Human gateway** — CDSL is internal; humans only see natural language

---

## Limitations

- No runtime parser yet (grammar enforced via prompt only)
- No conditional branching in the language
- Requires self-check prompt for reliable generation
- Tested on one model family (Qwen3.8-flash-Next) — cross-model validation pending
- Single-session state only (no cross-session persistence)

---

## Roadmap

| Phase | Focus | Priority |
|-------|-------|----------|
| 2 | Runtime parser/validator | HIGH |
| 3 | Extended language (conditionals, composition) | MEDIUM |
| 4 | Developer tooling (LSP, linter, playground) | MEDIUM |
| 5 | Cross-model validation (GPT, Claude, Gemini) | MEDIUM |
| 6 | Security & isolation model | LOW |
| 7 | Adoption documentation & tutorials | LOW |

---

## License

Apache-2.0 — see [LICENSE](LICENSE)

---

## Credits

Developed with: OpenCode + OpenSpec spec-driven workflow
