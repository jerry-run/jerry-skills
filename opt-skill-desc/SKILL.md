---
name: opt-skill-desc
description: Use this skill when the user wants to optimize, improve, or evaluate the description field of an existing Agent Skill's SKILL.md file. Trigger this skill when the user provides a path to a skill directory and wants to make the skill trigger more reliably — even if they say things like "the skill isn't triggering", "improve my skill description", "my skill doesn't activate", or "tune my skill's description". Use this for any task involving testing whether a skill's description accurately matches user intent queries.
version: 1.0.0
---

# opt-skill-desc

Automatically optimize the `description` field of a skill's `SKILL.md` so it triggers reliably and precisely — not too narrow, not too broad.

## Usage
```
opt-skill-desc <skillpath>
```

- `<skillpath>`: Path to the skill directory containing `SKILL.md`

---

## Instructions

When invoked with `<skillpath>`, execute the following steps in order.

---

### Step 1 — Read and Parse the Target Skill

1. Read `<skillpath>/SKILL.md`
2. Extract from frontmatter:
   - `name`: the skill name
   - `description`: the current description (record as the **baseline description**)
3. Read the full SKILL.md body to understand what the skill does

---

### Step 2 — Understand the Skill's Intent

Analyze the SKILL.md body and identify:
- **Core capability**: the specific task this skill handles
- **Target user actions**: what a user is doing when this skill would help
- **Boundaries**: adjacent tasks this skill does NOT handle
- **Domain keywords**: central vs. peripheral terms

---

### Step 3 — Prepare the optimize Directory

1. Create `<skillpath>/optimize/` if it does not already exist
2. Copy the current `<skillpath>/SKILL.md` into the optimize directory as the baseline record:
```
   <skillpath>/optimize/iter0_SKILL.md
```

This step must complete before any files are written to the optimize directory.

---

### Step 4 — Generate Eval Query Set

Generate exactly 20 labeled queries:
- 10 `should_trigger: true`: realistic prompts where the skill should activate
- 10 `should_trigger: false`: near-miss prompts sharing keywords but needing something different

**Should-trigger quality guidelines:**
- Mix formal/casual phrasing; include typos and abbreviations
- Mix explicit domain mentions with implicit need descriptions
- Mix short terse prompts with long context-heavy ones
- Mix single-step with multi-step workflows
- Prioritize cases where the skill would help but the connection isn't obvious

**Should-not-trigger quality guidelines:**
- Focus on near-misses, not obviously irrelevant queries
- Include queries sharing domain keywords but needing a different capability
- Avoid easy negatives like completely unrelated topics

**Train/Validation Split (fixed, random shuffle):**
- Train set: 12 queries (6 positive + 6 negative) — used to guide description revisions
- Validation set: 8 queries (4 positive + 4 negative) — used only to measure generalization

Save the query set to `<skillpath>/optimize/eval_queries.json`:
```json
[
  { "query": "...", "should_trigger": true, "split": "train" },
  { "query": "...", "should_trigger": false, "split": "validation" }
]
```

---

### Step 5 — Optimization Loop

Repeat the following loop **without a fixed iteration limit**. Stop only when the **overall pass rate across all 20 queries reaches ≥ 90%** (i.e., at least 18 out of 20 queries pass).

At the start of each iteration N (N = 1, 2, 3, …):

#### 5a — Evaluate Current Description

For each query in the eval set, simulate agent behavior:
- Read only the `name` and `description` fields (as an agent would at startup)
- Decide: would this description cause the agent to load the skill for this query?
- A query **passes** if:
  - `should_trigger: true` AND the description would cause the skill to load, OR
  - `should_trigger: false` AND the description would NOT cause the skill to load

Compute:
- `train_pass_rate`: fraction of train-set queries that pass
- `validation_pass_rate`: fraction of validation-set queries that pass
- `overall_pass_rate`: fraction of all 20 queries that pass

Save the evaluation results to `<skillpath>/optimize/iterN_results.json`:
```json
{
  "iteration": N,
  "description_tested": "...",
  "train_pass_rate": 0.XX,
  "validation_pass_rate": 0.XX,
  "overall_pass_rate": 0.XX,
  "results": [
    {
      "query": "...",
      "should_trigger": true,
      "split": "train",
      "triggered": true,
      "passed": true
    }
  ]
}
```

#### 5b — Check Stop Condition

If `overall_pass_rate >= 0.90`, **stop the loop**. Go to Step 6.

#### 5c — Analyze Failures (Train Set Only)

Look at train-set failures only:
- **Missed triggers** (`should_trigger: true` but did not trigger): description is too narrow → broaden scope, add context about when the skill applies, include non-obvious use cases
- **False triggers** (`should_trigger: false` but did trigger): description is too broad → add specificity, clarify what the skill does NOT do, sharpen the boundary

Do not add specific keywords from failed queries — find the general category those queries represent and address that.

If the description has not improved after 3 consecutive iterations, try a **structurally different framing** (e.g., reorder clauses, switch from capability-first to intent-first, add explicit exclusions).

#### 5d — Revise the Description

Write an improved description following these principles:
- Use imperative phrasing: "Use this skill when…" not "This skill does…"
- Focus on user intent, not implementation details
- Explicitly list applicable contexts, including cases where the user doesn't name the domain directly
- Keep it concise: a few sentences to a short paragraph
- **Hard limit: must be under 1024 characters**

Save the revised SKILL.md to `<skillpath>/optimize/iterN_SKILL.md` — a full copy of the original SKILL.md with only the `description` field replaced by the new version.

Proceed to the next iteration (N+1).

---

### Step 6 — Select the Best Description

After the loop ends:

- Review all `iterN_results.json` files
- Select the iteration with the **highest `validation_pass_rate`** (use `overall_pass_rate` as tiebreaker)
- If multiple iterations tie, prefer the one with the lower N (simpler, less overfit)

Record the winning iteration number and description.

---

### Step 7 — Apply and Report

1. **Show the user a before/after comparison:**
```
=== opt-skill-desc: Optimization Complete ===

Skill: <name>
Path:  <skillpath>
Best iteration: N  (overall pass rate: XX%)

--- BEFORE ---
<original description>

--- AFTER ---
<optimized description>

--- ITERATION HISTORY ---
iter0  (baseline)  train: XX%  validation: XX%  overall: XX%
iter1              train: XX%  validation: XX%  overall: XX%
...
iterN              train: XX%  validation: XX%  overall: XX%

Optimization files saved to: <skillpath>/optimize/
```

2. **Ask the user for confirmation:**
```
Apply the optimized description to <skillpath>/SKILL.md? (yes/no)
```

3. If the user confirms:
   - Update the `description` field in `<skillpath>/SKILL.md` with the winning description
   - Verify the description is under 1024 characters
   - Print: `✓ Description updated in <skillpath>/SKILL.md`

4. If the user declines:
   - Print: `Optimization files are saved in <skillpath>/optimize/ for your reference.`
   - Do not modify the original SKILL.md

---

## File Structure After Running
```
<skillpath>/
├── SKILL.md                    ← original (updated if user confirms)
└── optimize/
    ├── eval_queries.json       ← labeled query set (train + validation)
    ├── iter0_SKILL.md          ← baseline copy
    ├── iter1_SKILL.md          ← description after iteration 1
    ├── iter1_results.json      ← eval results for iteration 1
    ├── iter2_SKILL.md
    ├── iter2_results.json
    └── ...
```

