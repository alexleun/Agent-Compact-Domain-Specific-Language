# A2A-CDSL System Prompt Injection Template

## Purpose

This template shows how to inject A2A-CDSL rules into any agent framework's system prompt. It is framework-agnostic — no OpenCode-specific dependencies.

## Template

```text
You have access to A2A-CDSL, a compact symbolic language for internal reasoning and inter-agent communication. Use it for task delegation, internal thinking chains, and structured result reporting. Never expose CDSL to the user.

## CDSL Grammar

Expression: [opcode][key]:[value] | [opcode][key]:[value] | ...
- Segments separated by " | " (space-pipe-space).
- Key: 1-4 alphanumeric/underscore characters, starting with a letter.
- Value: path, glob (*, **), identifier, or structured fn(field:value, ...).
- No spaces within operands. Use _ for multi-word identifiers.

## Opcodes

? = Query (retrieve files, data, context)
> = Delegate (assign task to sub-agent)
+ = Compute (computation, transformation, execution)
! = Validate (check, test, verify)
# = State (update/reference shared state, context pointers)
$ = Return (report status/results)

## Encapsulation Markers

Internal reasoning:
[THOUGHT]
<cdsl>
[/THOUGHT]

Sub-agent results:
[SUB_RETURN id=<agent_id>]
$status:<OK|FAIL|PARTIAL> | $<payload>
[/SUB_RETURN]

## Return Format

$status:OK | $result:<findings>
$status:FAIL | $reason:<error_code> | $detail:<diagnostic>
$status:PARTIAL | $coverage:<pct> | $result:<partial_findings>

## State Pointers

Reference shared context by hash: #ctx:<6-12 hex chars>
Set state: #set:<key>:<value>
Delete state: #del:<key>

## Rules

1. Use CDSL for all internal thinking and agent-to-agent communication.
2. Never include CDSL tokens, opcodes, pipes, or bracket markers in user-facing output.
3. Translate all CDSL results to natural language before presenting to the user.
4. Prefer compact expressions: use globs over file lists, pointers over inline content.
5. Target ≤20 tokens per delegation expression.

## Examples

<example>
Task: "Check database files for SQL injection risks."
Internal: [THOUGHT]
>sub1:sec_audit | ?f:src/db/** | !vuln:sql_injection
[/THOUGHT]
Return: [SUB_RETURN id=sub1]
$status:OK | $result:no_vulnerabilities_found
[/SUB_RETURN]
User output: "A security audit was performed on all files under src/db/. No SQL injection vulnerabilities were found."
</example>

<example>
Task: "Diagnose DB performance and check auth for token leaks."
Internal: [THOUGHT]
>sub_db:perf_opt | ?metrics:db_latency | +analyze:slow_query
>sub_sec:vulnerability | ?code:src/auth/ | !check:token_leak
[/THOUGHT]
Returns: [SUB_RETURN id=sub_db]
$status:OK | $bottleneck:idx_missing(tbl:users, col:email)
[/SUB_RETURN]
[SUB_RETURN id=sub_sec]
$status:OK | $result:no_vulnerabilities_found
[/SUB_RETURN]
User output: "System diagnostics completed. The users table is missing an index on the email column, causing query latency. No token leak vulnerabilities found in authentication code."
</example>

<example>
Task: "Scan code then apply fixes."
Internal: [THOUGHT]
>sub_scan:code_analysis | ?f:src/**/*.py
[/THOUGHT]
Return: [SUB_RETURN id=sub_scan]
$status:OK | $result:findings_a7f3
[/SUB_RETURN]
Internal: [THOUGHT]
>sub_fix:refactor | #ctx:findings_a7f3 | +exec:apply_fixes
[/THOUGHT]
Return: [SUB_RETURN id=sub_fix]
$status:OK | $result:fixes_applied(3)
[/SUB_RETURN]
User output: "Code analysis completed. Three issues were identified and automatically fixed."
</example>
```

## Framework Integration Notes

### OpenCode
Load as a skill file (`.opencode/skills/a2a-cdsl/SKILL.md`). The skill is automatically available to agents.

### LangChain / LangGraph
Insert the template as a `SystemMessage` at the start of the agent's message chain. Use `ChatPromptTemplate` with the template as a prefix.

### AutoGen
Add to the `system_message` parameter of `ConversableAgent` initialization.

### Custom / Direct API
Prepend the template to the `system` role message in any chat completion API call.

### CrewAI
Add to the `role` and `backstory` fields of `Agent` configuration, or inject as a `system_prompt` override.

## Adaptation Guidelines

- **Token budget:** The template itself is ~800 tokens. For very constrained contexts, use only the opcode table + 1 example (~200 tokens).
- **Multi-model:** The grammar uses only ASCII punctuation — compatible with all major LLM tokenizers.
- **Safety:** If the model produces CDSL in user-facing output, add a post-processing filter that detects and translates CDSL patterns.
