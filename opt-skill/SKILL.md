---
name: opt-skill
description: Use this skill when the user wants to optimize, improve, audit, or refactor an existing Agent Skill's SKILL.md file to meet best practices. Trigger when the user provides a path to a skill directory and wants to improve the skill's content quality, structure, detail level, instructions, or effectiveness — even if they say things like "improve my skill", "my skill isn't working well", "optimize this skill", "make this skill better", "audit my skill", or "refactor my skill's instructions". Use for any task involving reviewing or rewriting skill body content (not just the description field).
version: 1.0.0
---

# opt-skill

Optimize an existing skill's `SKILL.md` to meet Agent Skills best practices — improving content quality, structure, calibration, and effectiveness.

## Usage
```
opt-skill <skillpath>
```

- `<skillpath>`: Path to the skill directory containing `SKILL.md`

---

## Best Practices Reference

The following criteria come from https://agentskills.io/skill-creation/best-practices. Apply all of them when auditing and rewriting.

### 1. Add what the agent lacks, omit what it knows
Include project-specific conventions, non-obvious edge cases, and domain-specific procedures. Cut explanations of things the agent already knows (e.g., what HTTP is, what a PDF is).

### 2. Design coherent units
The skill should encapsulate one coherent unit of work. If it covers too many unrelated tasks, it's too broad. If it needs to be combined with several other skills for a single task, it's too narrow.

### 3. Aim for moderate detail
Concise, stepwise guidance with a working example outperforms exhaustive documentation. Avoid covering every edge case — let the agent handle the rest with its own judgment.

### 4. Progressive disclosure (size limit)
Keep `SKILL.md` under 500 lines and 5,000 tokens. Move detailed reference material to `references/` or `assets/` subdirectories. Reference these files conditionally: "Read `references/api-errors.md` if the API returns a non-200 status."

### 5. Match specificity to fragility
- **Flexible parts**: explain *why*, not a rigid procedure. Let the agent decide.
- **Fragile parts**: exact commands, exact sequences, explicit "do not modify" warnings.

### 6. Provide defaults, not menus
When multiple tools or approaches exist, pick one default and mention alternatives briefly. Do not present equal options for the agent to choose from.

### 7. Favor procedures over declarations
Teach the agent *how to approach* the class of problems, not what to output for a specific instance. Include output format templates for consistency, but the approach must generalize.

### 8. Gotchas section
List environment-specific facts that defy reasonable assumptions — not general advice. These go in `SKILL.md` directly so the agent reads them before encountering the issue.

### 9. Templates for output format
When output format matters, provide a concrete template. Agents pattern-match against concrete structures more reliably than prose descriptions.

### 10. Checklists for multi-step workflows
Explicit checklists help the agent track progress and avoid skipping steps, especially when steps have dependencies or validation gates.

### 11. Validation loops
Instruct the agent to validate its own work: do → validate → fix → repeat until passing. Reference a script, checklist, or self-check as the validator.

### 12. Plan-validate-execute for batch/destructive operations
Have the agent create an intermediate plan, validate it against a source of truth, then execute. This prevents hard-to-reverse mistakes.

---

## Instructions

### Step 1 — Read the target skill

1. Read `<skillpath>/SKILL.md` fully
2. Extract:
   - `name`, `description`, `version` from frontmatter
   - Full body content
3. Note the skill's stated purpose and workflow

---

### Step 2 — Audit against best practices

Evaluate the skill against each of the 12 criteria above. For each criterion, determine:
- **PASS**: The skill already meets this criterion
- **FAIL**: The skill violates or ignores this criterion
- **N/A**: The criterion does not apply to this skill type

Produce an internal audit table:

| # | Criterion | Status | Issue |
|---|-----------|--------|-------|
| 1 | Add what agent lacks / omit what it knows | PASS/FAIL/N/A | ... |
| 2 | Coherent unit | ... | ... |
| 3 | Moderate detail | ... | ... |
| 4 | Progressive disclosure (≤500 lines / ≤5000 tokens) | ... | ... |
| 5 | Specificity matched to fragility | ... | ... |
| 6 | Defaults not menus | ... | ... |
| 7 | Procedures over declarations | ... | ... |
| 8 | Gotchas section | ... | ... |
| 9 | Output format templates | ... | ... |
| 10 | Checklists for multi-step workflows | ... | ... |
| 11 | Validation loops | ... | ... |
| 12 | Plan-validate-execute | ... | ... |

---

### Step 3 — Show audit results to the user

Display the audit table. For each FAIL, briefly explain:
- What is wrong
- What should be done to fix it

Example:
```
=== opt-skill: Audit Results for <name> ===

PASS (6): #1, #3, #5, #7, #9, #11
FAIL (4): #2, #4, #6, #8
N/A  (2): #10, #12

Issues found:
[#2] Coherent unit: The skill covers both database querying AND schema migration.
     These should be separate skills or the migration section should be trimmed.

[#4] Progressive disclosure: SKILL.md is 620 lines (exceeds 500-line limit).
     Move the API reference table to references/api-reference.md and load
     conditionally.

[#6] Defaults not menus: The "Choose a parser" section lists 4 equal options.
     Pick pdfplumber as default; mention pdf2image only for OCR fallback.

[#8] Gotchas section missing: No gotchas section found. Add environment-specific
     facts that the agent would not know (e.g., soft-delete patterns, ID
     field naming inconsistencies, health-check endpoint quirks).
```

Then ask:
```
Proceed with rewriting SKILL.md to fix all FAIL items? (yes/no)
If you prefer, you can list specific issue numbers to fix (e.g., "2,4,6").
```

---

### Step 4 — Rewrite the skill

After user confirms (all issues or a subset):

1. Create `<skillpath>/optimize/` if it does not exist
2. Save the original as `<skillpath>/optimize/original_SKILL.md`
3. Rewrite `<skillpath>/SKILL.md`:
   - Fix all confirmed FAIL items
   - Preserve all PASS content without modification
   - Do not introduce new features or scope the user didn't ask for
   - If content moves to `references/`, create those files and update all references in SKILL.md
   - Keep SKILL.md under 500 lines and 5,000 tokens
   - Preserve frontmatter (`name`, `description`, `version`)

---

### Step 5 — Report

After rewriting, show:

```
=== opt-skill: Rewrite Complete ===

Skill: <name>
Path:  <skillpath>

Fixed issues: #2, #4, #6, #8
Preserved:    #1, #3, #5, #7, #9, #11

Changes made:
- [#2] Removed schema migration section (out of scope for this skill)
- [#4] Moved API reference table to references/api-reference.md (120 lines saved)
       SKILL.md is now 380 lines
- [#6] Set pdfplumber as default; demoted PyMuPDF/pypdf to footnote
- [#8] Added Gotchas section with 3 entries

Files written:
  <skillpath>/SKILL.md                        ← updated
  <skillpath>/optimize/original_SKILL.md      ← backup
  <skillpath>/references/api-reference.md     ← new (if applicable)
```

---

## Gotchas

- **Description field**: This skill audits the SKILL.md *body*. For description-only optimization (triggering reliability), use `opt-skill-desc` instead.
- **N/A is valid**: Not every skill needs checklists, validation loops, or plan-validate-execute. Only flag these as FAIL if the skill's workflow would clearly benefit from them (multi-step workflows, batch operations, destructive actions).
- **Preserve PASS items verbatim**: Do not rewrite sections that already pass — unnecessary rewrites introduce regressions and waste the user's review time.
- **References files**: When moving content to `references/`, always add a conditional load instruction in SKILL.md explaining when to read the file. A bare "see references/" without a trigger condition is not progressive disclosure.
