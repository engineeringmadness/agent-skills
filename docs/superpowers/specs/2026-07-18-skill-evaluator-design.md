# Skill Evaluator — Meta-Skill Design Spec

## Overview

Skill Evaluator is a meta-skill (a SKILL.md file) that teaches any coding agent how to evaluate other skills. When activated, the agent becomes a test harness: it loads a JSON test suite, runs each prompt with and without the target skill in an A/B comparison, then judges whether the skill produced the expected behavioral change.

## Test Suite Format

A JSON array of test cases. Each case defines:

| Field | Type | Description |
|-------|------|-------------|
| `title` | string | Human-readable name for the test case |
| `skill` | object | `{ "name": "...", "path": "./skills/foo/SKILL.md" }` |
| `prompt` | string | The input prompt to run through the agent |
| `expectedChange` | string | Plain English description of how the output should differ when the skill is active vs baseline |

**Example:**

```json
[
  {
    "title": "Code review follows structured format",
    "skill": {
      "name": "code-review",
      "path": "./skills/code-review/SKILL.md"
    },
    "prompt": "Review this function for bugs: function add(a,b) { return a+b; }",
    "expectedChange": "The output should include organized sections (Summary, Issues, Suggestions) and avoid informal language"
  }
]
```

## Execution Protocol

For each test case in the suite:

1. **Run baseline** — execute the prompt without the target skill active. Capture the output.
2. **Load target skill** — activate the skill identified by `skill.path`.
3. **Run with skill** — execute the same prompt with the target skill active. Capture the output.
4. **Judge** — compare both outputs against `expectedChange`. Determine pass/fail with reasoning.
5. **Record result** — add to the results set and continue to the next case.

## Judgment Protocol

The agent acts as its own LLM-judge. For each test case, evaluate:

- Does the "with skill" output differ from baseline in the way described by `expectedChange`?
- Is the difference attributable to the skill's influence, not random variation?

A test passes only when the described behavioral change is clearly observable in the with-skill output and absent from the baseline.

The judge must provide reasoning — a brief explanation of *why* it passed or failed, referencing specific differences between the two outputs.

## Results Format

A structured report in the agent's output:

```
| # | Test Title | Result | Reasoning |
|---|------------|--------|-----------|
| 1 | Code review follows structured format | PASS | Output with skill has Summary/Issues/Suggestions sections; baseline is unstructured |
| 2 | ... | FAIL | Both outputs are nearly identical; no evidence of behavioral change |

Summary: 1/2 passed
```

## Constraints

- **No configuration** — the evaluator is pure procedure, no knobs or settings.
- **File-based only** — test suites are loaded from JSON files. No ad-hoc single-case evaluation.
- **Agent-agnostic** — works with any coding agent that can toggle skill activation during execution.
- **No external tooling** — the agent itself does all work: running, comparing, judging, reporting.
