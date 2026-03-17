# Pull Request Guide for Skills

> Skill PRs modify **instructions for LLMs**, not code.
> Help reviewers understand how your changes improve Claude's comprehension and trigger accuracy.

## Table of Contents

1. [What Makes Skill PRs Different](#what-makes-skill-prs-different)
2. [Template](#template)
3. [Quick Checklist](#quick-checklist)
4. [Writing Guidelines](#writing-guidelines)
5. [Scope Considerations](#scope-considerations)
6. [Tone & Manner](#tone--manner)
7. [Template Examples](#template-examples)
8. [For Auto-PR Contributors (Mode 3)](#for-auto-pr-contributors-mode-3)

---

## What Makes Skill PRs Different

Skills are not regular code. Different validation rules apply:

| Aspect | Regular Code PR | Skill PR |
|--------|----------------|----------|
| **Content** | Source code | YAML + Markdown instructions |
| **UI changes** | Screenshots required | No UI - skip screenshots |
| **Testing** | Unit/integration tests | Eval + trigger tests + F/P checks |
| **Quality metric** | Code correctness | LLM understanding + trigger accuracy |
| **Review focus** | Logic + performance | Clarity + generalization + token efficiency |

**Key principle**: Explain **why** the change improves Claude's performance, not just **what** changed.

---

## Template

### Core Components

#### **Context & Intent**

Explain **why** this change is needed in 1-2 sentences.

```markdown
❌ "Improve skill-reviewer"
✅ "skill-reviewer's P1 check misses WHY explanations in code comments.
    Added pattern matching for //, #, <!-- comment syntax."
```

#### **Type of Change**

Check one or more:

```markdown
- [ ] ✨ New skill
- [ ] 📝 Improve description (trigger accuracy)
- [ ] 🔧 Fix instructions (execution quality)
- [ ] 📚 Add examples
- [ ] 🐛 Fix bug
- [ ] ♻️ Refactor
```

#### **Related Issues**

Link to issues, discussions, or documentation:

```markdown
Fixes #234
Related to #456
```

#### **PR Title**

Keep under 70 characters. Use type prefix for clarity:

```markdown
✅ "feat(description): add trigger conditions for better activation"
✅ "fix(P1): detect WHY in code comments"
✅ "docs: improve instructions clarity"
❌ "skill-reviewer improvements"
```

#### **Testing & Validation**

Show how you verified quality:

```markdown
- [ ] Eval results (success rate: 8/10 → 10/10)
- [ ] F1-F12 checks (structural)
- [ ] P1-P12 checks (quality)
- [ ] Trigger tests (activates correctly)
- [ ] Manual testing (real-world scenarios)
- [ ] SKILL.md ≤500 lines
```

**Be specific:**
```markdown
❌ "Testing complete"
✅ "Tested 10 scenarios locally:
    - Trigger: 'review my skill' → ✅ activates
    - Trigger: 'make a PR' → ✅ does not activate (correct)
    - F1-F12: all pass
    - P1-P12: P2 needs work → removed 150 tokens of basic explanations"
```

---

## Quick Checklist

Verify before submitting:

**PR Completeness:**
- [ ] Explained **why** the change is needed
- [ ] Selected Type of Change checkboxes
- [ ] Testing & Validation is specific, not vague
- [ ] Included Before/After comparison (speeds review by 2x)

**Skill Quality (F/P Checks):**
- [ ] Ran F/P checks yourself
- [ ] SKILL.md ≤500 lines
- [ ] Description includes "Use when..." trigger conditions
- [ ] References use relative paths (not absolute)
- [ ] No hardcoded paths (/Users/..., C:\...)
- [ ] ALWAYS/NEVER statements include WHY reasoning
- [ ] No unnecessary files (README.md, CHANGELOG, etc.)
- [ ] Eval results attached (if available)
- [ ] Scope-appropriate checks (📁 Project allows overfitting, 🌐 Public requires all checks)

---

## Writing Guidelines

### Keep It Specific

Vague descriptions waste reviewer time. Be concrete:

```markdown
❌ "Improved skill performance"
✅ "Trigger accuracy improved from 6/10 to 9/10 by adding 'Use when...' clause"

❌ "Fixed some issues"
✅ "P1 check now detects WHY in code comments (// reason:, # because:)"
```

### Use Metrics

Show measurable improvement when possible:

```markdown
- Eval: 8/10 → 10/10 ✅
- Tokens: 500 → 350 (30% reduction)
- Trigger accuracy: 60% → 90%
- F/P checks: 18/24 → 24/24
```

### Use HTML Comments for Template Hints

Include writing tips in the template that don't show up in the final PR:

```html
<!--
💡 Writing tips:
- description changes: explain how trigger accuracy improves
- instructions changes: explain what scenarios now work better
- Before/After comparisons make reviews 2x faster
-->
```

---

## Scope Considerations

Skills have different requirements based on their scope:

| Scope | What It Means | F/P Check Flexibility |
|-------|---------------|----------------------|
| 📁 **Project** | One repo, specific use case | Overfitting (P7) acceptable |
| 🏠 **Personal** | Your projects, must generalize | Most checks apply |
| 👥 **Team** | Shared within org | + naming, documentation |
| 🌐 **Public** | Open-source, marketplace | All F/P checks mandatory |

**When to mention scope:**

If your PR involves trade-offs, explain the scope context:

```markdown
✅ "P7 overfitting concern exists, but this is 📁 Project scope -
    designed specifically for this repo's API patterns"

✅ "P2 token efficiency: kept detailed examples since this is 🏠 Personal
    scope and helps me remember context months later"
```

---

## Tone & Manner

### 1. **Be a Helpful Guide: Share Intent**

Reviewers don't know your skill's purpose. Provide context.

**Explain the "why":**
```markdown
❌ "Improve commit skill"
✅ "Current commit skill uses --amend after pre-commit hook failures,
    overwriting the previous commit. Changed to create new commits after
    hook failures instead."
```

**Show Before/After:**
```markdown
## Before
description: Processes PDF files.

## After
description: Extracts text and tables from PDF files, fills forms, merges
  documents. Use when the user mentions PDFs, forms, or document extraction.

## Why
Missing trigger conditions - won't activate even for clear requests like
"read this pdf". Adding "Use when..." improves trigger accuracy.
```

### 2. **Be Humble Yet Confident**

Explain your reasoning while staying open to feedback:

```markdown
✅ "From a P3 (freedom calibration) perspective, this approach seems
    appropriate. Open to suggestions for better methods."

❌ "This is the only correct solution"
```

### 3. **Self-Review Shows Respect**

Save reviewer time by checking your own work first.

**Run F/P checks yourself:**
```markdown
## Self-Review
F1-F12: ✅ all pass
P1-P12: ⚠️ P2 needs improvement
  - L45-60: Explains basic PDF concepts Claude already knows (150 tokens)
  - → Removed, replaced with code example (50 tokens)
  - Saved 100 tokens
```

**Add self-comments:**
Add comments to your own PR immediately after submission:
```markdown
💭 "L120's ALWAYS looks like it lacks WHY, but the reasoning is
    explained in the paragraph at L115-118"
⚠️ "Submitted without eval - validated with manual testing only"
🔍 "P7 (overfitting) concern exists but this is 📁 Project scope so acceptable"
```

### 4. **Feedback = Growth Opportunity**

PRs improve skill quality through collaboration:

**Frame feedback constructively:**
- "This part is wrong" ❌
- "This approach could improve trigger accuracy" ✅

**Use gentle emoji:**
- ✅ Agree, completed
- 💡 Suggestion
- 🤔 Question
- ⚠️ Attention needed
- 📊 Eval results

---

## Template Examples

### Example 1: Improve Description

```markdown
## Context & Intent
skill-reviewer's description lacks trigger conditions. Doesn't activate
for clear requests like "review my skill". Adding "Use when..." to improve
trigger accuracy.

## Type of Change
- [x] 📝 Improve description

## Related Issues
Fixes #123

## Changes
- Added "Use when..." trigger conditions to description
- Listed specific use cases (review, validate, audit, lint)

## Testing & Validation
- [x] Trigger tests - 10 scenarios
  - "review my skill" → ✅ activates
  - "create a skill" → ✅ delegates to skill-creator (correct)
  - "review my command" → ⚠️ activates instead of command-reviewer → fixed description
- [x] F1-F12: all pass
- [x] P1-P12: all pass

## Before/After

### Before
```yaml
description: Evaluates Claude Code skills against best practices.
```

### After
```yaml
description: Evaluates, improves, and creates Claude Code skills against
  Anthropic's official best practices. Use when the user wants to create,
  review, validate, audit, lint, or improve skills.
```

## Why
F4 (trigger conditions) check - without "Use when...", Claude doesn't know
when to activate. Listing specific verbs (review, validate, audit) improves
trigger accuracy.
```

### Example 2: Fix Instructions Bug

```markdown
## Context & Intent
P1 check misses WHY explanations in code comments. Pattern like
"// reason: ..." should be recognized as WHY. Updated detection logic.

## Type of Change
- [x] 🐛 Fix bug

## Related Issues
Related to #456

## Changes
- L234: Search 3 lines before/after ALWAYS/NEVER, not just immediately after
- Added comment patterns: `//`, `#`, `<!--`

## Testing & Validation
- [x] Eval execution
  - Before: 6/10 (false positive - missed WHY in comments)
  - After: 10/10 ✅
- [x] F1-F12: all pass
- [x] P1-P12: all pass

## Example

### Before (false positive)
```python
# ALWAYS validate input
# reason: invalid input causes downstream errors, hard to debug
def process(data):
    validate(data)
```
→ P1 fails: "WHY missing" (comment not detected)

### After (correct)
→ P1 passes: "WHY present" (detects "reason:" in comment)
```

---

## For Auto-PR Contributors (Mode 3)

When contributing to others' skills, follow the **Additive Principle**:

**Preserve existing content** - other users may depend on current behavior.

```markdown
❌ "Removed metadata.json (non-standard)"
✅ "Added marketplace.json alongside metadata.json"

❌ "Rewrote description in English"
✅ "Added description-en.yaml (original preserved)"

❌ "Fixed the incorrect validation logic"
✅ "Enhanced validation to handle edge case X (original logic preserved for compatibility)"
```

**Why additive?**
- Respects author's design decisions
- Prevents breaking existing users' workflows
- Shows collaborative spirit, not prescriptive control

See SKILL.md Mode 3 for full Additive Principle guidelines.

---

**Core principle**: Skill PRs are about **LLM understanding** and **trigger accuracy**.
Clearly communicate why the change is needed and how it improves quality.
