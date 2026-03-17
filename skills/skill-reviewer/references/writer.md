# Skill Writer

Creates skills following the F1-F12 and P1-P12 checklist (already loaded from the parent SKILL.md).

This document is internal to skill-reviewer. Users should invoke skill-reviewer directly, which will delegate to this writer when creating new skills.

## Table of Contents

- [Phase 1: Clarify Intent](#phase-1-clarify-intent)
- [Phase 2: Classify — Skill](#phase-2-classify--skill-already-determined)
- [Phase 3: Generate](#phase-3-generate)
- [Phase 4: Verify](#phase-4-verify)
- [Edge Cases](#edge-cases)
- [Integration with parent](#integration-with-parent-skill-reviewer)

```
User's goal → [Clarify] → [Classify] → [Generate] → [Verify] → Output
                  ↑                                       |
                  └─── AskQuestion tool if unclear ────────┘
```

## Phase 1: Clarify Intent

Three things must be clear before generating. If any is missing AND cannot be safely defaulted, use the AskUserQuestion tool.

### 1.1 What — The Goal

What capability does the user want to add? If the description is too vague to write a meaningful skill, ask.

```
Sufficient:
  "A skill that helps process PDF files - extract text, fill forms, merge documents"
  → Clear capability, clear use cases

Insufficient:
  "Make something for documents"
  → Ask: "What document operations do you need? For example: PDF extraction,
    Word editing, spreadsheet analysis, format conversion..."
```

### 1.2 Where — The Scope

| Profile | User signals | Location |
|---------|-------------|----------|
| 📁 Project | "for this repo only", "just this project" | `.claude/skills/` in project |
| 🏠 Personal | "on my machine", "across all projects" | `~/.claude/skills/` |
| 👥 Team | "for the team", "shared with others" | Team repo `.claude/skills/` |
| 🌐 Public | "open-source", "marketplace" | Published externally |

**Default: 🏠 Personal** when unspecified. This is the most common case and strict enough to generalize. State the assumption so the user can correct it.

### 1.3 How Strict — Freedom Level

| Fragile (low freedom) | Flexible (high freedom) |
|----------------------|----------------------|
| DB migration, XML editing, file deletion | Code review, docs, data analysis |

If the user describes a destructive operation, default to low freedom with exact steps. If creative/exploratory, default to high freedom with guidelines.

### When to Use AskUserQuestion Tool

**Ask when:**
- Goal is ambiguous — cannot produce a reasonable first draft
- Scope matters AND is ambiguous — project vs personal changes the output significantly
- Conflicting signals — "simple migration skill" (simple + destructive = unclear freedom)

**Do NOT ask when:**
- Goal is clear enough for a reasonable draft. Asking unnecessarily slows the workflow and signals lack of competence. A draft with stated assumptions is faster and more useful than multiple Q&A rounds.
- Scope can be safely defaulted to 🏠 Personal
- Freedom level is obvious from the task type

**Ask at most 2 questions at a time.** Batch related questions. Never interrogate.

## Phase 2: Classify — Skill (Already Determined)

Since you're in skill-reviewer/references/writer.md, the classification is already done: **SKILL**.

If the user needs a command instead, refer them to command-reviewer.

## Phase 3: Generate

### 3.1 Load Quality Criteria

Apply the F/P checklist (already in context). Filter by scope column (📁+, 🏠+, 👥+, 🌐).

Do NOT duplicate the checks here — the parent SKILL.md is the single source of truth.

### 3.2 Scope-Aware Generation

Two key adjustments beyond the scope column filter:

| Scope | Key adjustment |
|-------|---------------|
| 📁 Project | Overfitting is intentional. Hardcoded project paths are fine. Skip generalization checks. |
| 🏠 Personal | Must work across projects. Description needs trigger conditions. No hardcoded paths. |
| 👥 Team | Add: third-person voice, concrete examples for onboarding, TOC for long references. |
| 🌐 Public | All checks mandatory. Reference files need TOC. Dependencies explicit. |

### 3.3 Output Structure

**Skill:**
```
.claude/skills/[skill-name]/
├── SKILL.md                    (≤500 lines)
└── references/                 (optional, for overflow)
```

### 3.4 Structure Template (Optional)

For 👥 Team / 🌐 Public skills, offer the full structure template. For 📁 Project / 🏠 Personal, ask preference.

**Ask the user:**
> "This skill is for [inferred scope]. Would you like the full structured format (recommended for team sharing) or a minimal version?"

**Full structure (👥+ recommended):**
```markdown
---
name: skill-name
description: "What this skill does. Use when [trigger conditions]."
---

# [Descriptive Skill Title]

[1-2 sentence summary]

## When to Use This Skill

- [Scenario 1]
- [Scenario 2]

## What This Skill Does

1. **[Capability 1]**: [Description]
2. **[Capability 2]**: [Description]

## How to Use

[Usage patterns and examples]

## Example

**User**: "[Sample request]"

**Output**:
[Sample output]

## [Domain-specific sections as needed]

## References

- `references/[file].md` — [Description]
```

**Minimal structure (📁/🏠 acceptable):**
```markdown
---
name: skill-name
description: "What this skill does. Use when [triggers]."
---

# [Title]

[Core instructions]

## Example
[If helpful]
```

## Phase 4: Verify

### 4.1 Self-Check

Self-review against the same F/P checklist. For each check applicable to the scope:
- ✅ Pass → move on
- ❌ Fail → fix immediately
- ⚠️ Uncertain → re-read the WHY column, then decide

### 4.2 Common Cascading Issues

Fixes can introduce new problems. Watch for these:

| Fix | May cause | Resolution |
|-----|-----------|------------|
| Adding WHY explanations (P1) | Verbosity (P2) | Be concise |
| Adding trigger conditions (F4) | Description too long | Keep under 1024 chars |
| Adding examples (P5) | Exceeds 500 lines (F5) | Move to references/ |

Re-check after fixes to catch cascades.

### 4.3 Present Output

When presenting the generated file(s), include:

1. **What** — file path and name
2. **Why this format** — skill (not command) reasoning
3. **Scope applied** — which profile (📁/🏠/👥/🌐) and key implications
4. **Key design decisions** — freedom level, reference structure, notable trade-offs
5. **Assumptions** — anything inferred rather than stated; flag what the user should verify

Keep the explanation concise — the user can read the file.

## Edge Cases

### Improve existing skill

If the user provides an existing skill, switch to improvement mode:
1. Run the parent reviewer (Mode 1: Self-Review) on the existing file
2. Identify issues
3. Rewrite with fixes applied
4. Show what changed and why

Respect the user's existing structure and decisions. Don't rewrite from scratch unless the user asks.

### User is unsure what to build

Help them discover candidates:
1. Ask about their most repetitive tasks
2. Ask about contexts where they need specialized knowledge
3. Suggest 2-3 concrete skill candidates
4. Let them pick, then proceed to Phase 1

### Ecosystem considerations (👥 Team / 🌐 Public)

- Does CLAUDE.md need updating to reference the new skill?
- Do existing skills overlap?
- Should the new skill complement existing ones?

## Integration with parent skill-reviewer

This writer is invoked by skill-reviewer Mode 4 (Create). The parent handles:
- User-facing invocation
- Routing to this writer
- Final validation

This writer focuses on:
- Clarifying intent (Phase 1)
- Generating the skill (Phase 3)
- Self-verification (Phase 4)
