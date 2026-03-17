# Command Writer

Creates commands following the CF1-CF10 and CP1-CP12 checklist (already loaded from the parent SKILL.md).

This document is internal to command-reviewer. Users should invoke command-reviewer directly, which will delegate to this writer when creating new commands.

## Table of Contents

- [Phase 1: Clarify Intent](#phase-1-clarify-intent)
- [Phase 2: Classify — Command](#phase-2-classify--command-already-determined)
- [Phase 3: Generate](#phase-3-generate)
- [Phase 4: Verify](#phase-4-verify)
- [Edge Cases](#edge-cases)
- [Integration with parent](#integration-with-parent-command-reviewer)

```
User's goal → [Clarify] → [Classify] → [Generate] → [Verify] → Output
                  ↑                                       |
                  └─── AskQuestion tool if unclear ────────┘
```

## Phase 1: Clarify Intent

Three things must be clear before generating. If any is missing AND cannot be safely defaulted, use the AskUserQuestion tool.

### 1.1 What — The Goal

What does the user want to automate? If the description is too vague to write a meaningful prompt, ask.

```
Sufficient:
  "Run lint and tests before commit, then generate a conventional commit message"
  → Clear procedure, clear success criteria

Insufficient:
  "Make something for code stuff"
  → Ask: "What task do you want to automate? For example: code review,
    commit workflow, deployment, documentation generation..."
```

### 1.2 Where — The Scope

| Profile | User signals | Location |
|---------|-------------|----------|
| 📁 Project | "for this repo only", "just this project" | `.claude/` in project |
| 🏠 Personal | "on my machine", "across all projects" | `~/.claude/` |
| 👥 Team | "for the team", "shared with others" | Team repo `.claude/` |
| 🌐 Public | "open-source", "marketplace" | Published externally |

**Default: 🏠 Personal** when unspecified. This is the most common case and strict enough to generalize. State the assumption so the user can correct it.

### 1.3 How Strict — Freedom Level

| Fragile (low freedom) | Flexible (high freedom) |
|----------------------|----------------------|
| DB migration, deploy, file deletion | Code review, docs, refactoring |

If the user describes a destructive operation, default to low freedom with exact steps. If creative/exploratory, default to high freedom with guidelines.

### When to Use AskUserQuestion Tool

**Ask when:**
- Goal is ambiguous — cannot produce a reasonable first draft
- Scope matters AND is ambiguous — project vs personal changes the output significantly
- Conflicting signals — "simple deploy script" (simple + destructive = unclear freedom)

**Do NOT ask when:**
- Goal is clear enough for a reasonable draft. Asking unnecessarily slows the workflow and signals lack of competence. A draft with stated assumptions is faster and more useful than multiple Q&A rounds.
- Scope can be safely defaulted to 🏠 Personal
- Freedom level is obvious from the task type

**Ask at most 2 questions at a time.** Batch related questions. Never interrogate.

## Phase 2: Classify — Command (Already Determined)

Since you're in command-reviewer/references/writer.md, the classification is already done: **COMMAND**.

If the user needs a skill instead, refer them to skill-reviewer.

## Phase 3: Generate

### 3.1 Load Quality Criteria

Apply the CF/CP checklist (already in context). Filter by scope column (📁+, 🏠+, 👥+, 🌐).

Do NOT duplicate the checks here — the parent SKILL.md is the single source of truth.

### 3.2 Scope-Aware Generation

Two key adjustments beyond the scope column filter:

| Scope | Key adjustment |
|-------|---------------|
| 📁 Project | Overfitting is intentional. Hardcoded project paths are fine. Skip generalization checks. |
| 🏠 Personal | Must work across projects. Description needs trigger conditions. No hardcoded paths. |
| 👥 Team | Add: third-person voice, argument hints, concrete examples for onboarding. |
| 🌐 Public | All checks mandatory. Reference files need TOC. Dependencies explicit. |

### 3.3 Output Structure

**Command:**
```
.claude/commands/[verb-noun].md
```
Single file: YAML frontmatter + prompt body. Typical 20-150 lines.

### 3.4 Structure Template (Optional)

For 👥 Team / 🌐 Public commands, offer the full structure template. For 📁 Project / 🏠 Personal, ask if they want the full structure or minimal version.

**Ask the user:**
> "This command is for [inferred scope]. Would you like the full structured format (recommended for team sharing) or a minimal version?"

**Full structure (👥+ recommended):**
```markdown
---
name: verb-noun
description: One-line description of what this command does
argument-hint: "<required-arg> [optional-arg]"
allowed-tools: Bash(specific:*), Read
model: sonnet
---

# [Descriptive Command Title]

[1-2 sentence summary of the command's purpose]

## When to Use This Command

- [Scenario 1]
- [Scenario 2]

## What This Command Does

1. **[Action 1]**: [Brief description]
2. **[Action 2]**: [Brief description]

## How to Use

/command-name <required-arg> [optional-arg]

| Argument | Description | Format | Required |
|----------|-------------|--------|----------|
| required-arg | What this is | format | **Yes** |
| optional-arg | What this is | format | No |

## Example

**Input**: `/command-name value1 value2`

**Output**: [Expected output sample]

## Tips

- **[Topic 1]**: [Important note]

---

## Execution Flow

### Step 1: [Action]
[Implementation details for Claude]
```

**Minimal structure (📁/🏠 acceptable):**
```markdown
---
name: verb-noun
description: One-line description
allowed-tools: [tools]
---

# [Title]

[What to do and how]

## Example
[Input/output if helpful]
```

## Phase 4: Verify

### 4.1 Self-Check

Self-review against the same CF/CP checklist. For each check applicable to the scope:
- ✅ Pass → move on
- ❌ Fail → fix immediately
- ⚠️ Uncertain → re-read the WHY column, then decide

### 4.2 Common Cascading Issues

Fixes can introduce new problems. Watch for these:

| Fix | May cause | Resolution |
|-----|-----------|------------|
| Adding WHY explanations (CP1) | Verbosity (CP2) | Be concise |
| Scoping allowed-tools (CP10) | Inline commands break (CF10) | Verify each inline command has matching scope |
| Expanding description (CF3) | Description too long | Keep under 200 chars |

Re-check after fixes to catch cascades.

### 4.3 Present Output

When presenting the generated file(s), include:

1. **What** — file path and name
2. **Why this format** — command (not skill) reasoning
3. **Scope applied** — which profile (📁/🏠/👥/🌐) and key implications
4. **Key design decisions** — freedom level, tool scoping, notable trade-offs
5. **Assumptions** — anything inferred rather than stated; flag what the user should verify

Keep the explanation concise — the user can read the file.

## Edge Cases

### Improve existing command

If the user provides an existing command, switch to improvement mode:
1. Run the parent reviewer (Mode 1: Self-Review) on the existing file
2. Identify issues
3. Rewrite with fixes applied
4. Show what changed and why

Respect the user's existing structure and decisions. Don't rewrite from scratch unless the user asks.

### User is unsure what to build

Help them discover candidates:
1. Ask about their most repetitive tasks
2. Ask about their most error-prone tasks
3. Suggest 2-3 concrete command candidates
4. Let them pick, then proceed to Phase 1

### Ecosystem considerations (👥 Team / 🌐 Public)

- Does CLAUDE.md need updating to reference the new command?
- Do existing commands overlap?
- Should the new command complement existing ones?

## Integration with parent command-reviewer

This writer is invoked by command-reviewer Mode 3 (Create). The parent handles:
- User-facing invocation
- Routing to this writer
- Final validation

This writer focuses on:
- Clarifying intent (Phase 1)
- Generating the command (Phase 3)
- Self-verification (Phase 4)
