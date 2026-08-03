---
tags: [claude-code, engineering, prompts, llm]
type: engineering
related:
  - "[[Context-Engineering]]"
  - "[[Harness-Engineering]]"
  - "[[CLAUDE-md]]"
  - "[[Loop-Engineering]]"
created: 2026-08-02
status: stable
---

# Prompt Engineering

> **Source**: notes from *AI Builders Meetup Guadalajara #1*, hosted by Wizeline (Guadalajara, 2026-08-02).

## Fundamentals

- **Clarity**: specificity over ambiguity. "Refactor this function" is ambiguous; "reduce this function's cyclomatic complexity below 5 without changing its external behavior" is specific.
- **Context**: what Claude already knows vs what it needs to be told. Claude doesn't know your project without [[CLAUDE-md]] or without being told explicitly in the prompt.
- **Constraints**: explicit limits on expected behavior — what to do and what NOT to do.
- **Format**: specify the expected output format — JSON, markdown with defined structure, numbered list, etc.

## Anatomy of an Effective Prompt

```
[Role / Persona]
[Project context]
[Specific task]
[Expected output format]
[Constraints / Anti-patterns to avoid]
[Examples (few-shot)]
```

- **Role/Persona**: optional but useful when a specific level of expertise is needed ("act as a security engineer")
- **Project context**: mandatory if not already covered by [[CLAUDE-md]] — without it, Claude assumes generically
- **Specific task**: always mandatory, the heart of the prompt
- **Expected format**: mandatory when the output is consumed programmatically (downstream parsing)
- **Constraints**: mandatory when there's prior incorrect behavior to avoid
- **Few-shot examples**: optional, but critical when format or style is hard to describe in prose

## Advanced Techniques

### Few-shot prompting
2-3 input→output examples before the real task, to anchor the expected pattern.

```
Example 1:
Input: "user can't log in"
Output: {"category": "auth", "severity": "high"}

Example 2:
Input: "button is 2px misaligned"
Output: {"category": "ui", "severity": "low"}

Real task:
Input: "payments endpoint returns 500 intermittently"
Output:
```

### Chain-of-Thought (CoT)
Explicitly asking for step-by-step reasoning before the final answer improves tasks requiring multi-step logic.

```
Before answering, think step by step:
1. What condition triggers the bug?
2. What code handles that condition?
3. What's the minimal fix?
Then give the final answer.
```

### ReAct Pattern
Reason → Act → Observe → Repeat. The fundamental pattern for tool-using agents — see [[Loop-Engineering]] for the complete implementation of the loop.

### Role Prompting
Assigning specific expertise adjusts the vocabulary, depth, and priorities of the response.

```
You are a senior backend engineer with 10 years of experience in
high-availability payment systems. Review this PR focusing on
race conditions and idempotency.
```

### Output Format Specification
Specifying the exact format reduces post-processing and parsing errors.

```
Respond ONLY with valid JSON matching this schema:
{"bug_found": bool, "file": str, "line": int, "explanation": str}
```

### Negative Prompting
Explicit instructions on what to avoid, when the default behavior tends toward error.

```
DO NOT add error handling for cases that cannot occur.
DO NOT use new external libraries without asking first.
DO NOT rename existing public function names.
```

### Response Prefill
Forcing the start of the output guides the structure of what follows (more applicable in direct API use than interactive Claude Code, but relevant for sub-agent prompts).

## Prompts for Code vs Analysis

| Characteristic | Code Prompt | Analysis Prompt |
|---|---|---|
| Level of detail | High — exact file names, functions, contracts | Medium — the goal matters more than exact detail |
| Output format | Almost always code + brief explanation | Structured prose, tables, lists |
| Constraints | Critical (what NOT to touch, what patterns to follow) | Less critical, more about scope of analysis |
| Few-shot examples | Very effective (existing code pattern) | Less common, mostly for output format |
| Verifiability | High — run it, test it, lint it | More subjective — requires human review |

## Prompt Testing and Iteration

- Method: prompt → test → evaluate → refine, iteratively (see [[Harness-Engineering]] to systematize this)
- How to measure quality: accuracy (is it correct?), completeness (does it cover everything asked?), format compliance (does it respect the requested format?)
- A/B testing prompts: run the same input with 2 prompt variants and compare outputs systematically, not just "by eye"
- High temperature (more randomness) for creative/exploratory generation; low temperature (more deterministic) for code and data extraction tasks where consistency matters more than variety

## Anti-patterns

**Vague prompt**
```
❌ "improve this code"
✅ "refactor this function to reduce its cyclomatic complexity
   below 5, without changing its observable external behavior"
```

**No context**
```
❌ Assuming Claude knows the project's conventions without CLAUDE.md
   or mentioning them in the prompt
✅ Explicitly reference the expected pattern or make sure
   CLAUDE.md documents it (see [[CLAUDE-md]])
```

**Contradictory instructions**
```
❌ "Always use async/await" ... further down ... "here's a
   synchronous example of how it should look"
✅ Consistency across all parts of the prompt
```

**Asking for multiple things without priority**
```
❌ "fix the bug, improve performance, add tests, and document everything"
✅ "Priority 1: fix the race condition bug. Priority 2 (only
   if 1 is resolved): add a regression test."
```

## Related Notes

- [[Context-Engineering]] — what context to put in the prompt vs what lives in higher layers
- [[Harness-Engineering]] — how to systematically evaluate a prompt's quality
- [[CLAUDE-md]] — project context that shouldn't be repeated in every prompt
- [[Loop-Engineering]] — the ReAct pattern applied to tool-using agents
