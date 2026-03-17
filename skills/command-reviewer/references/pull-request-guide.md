# Pull Request Guide for Commands

> Command PRs modify **slash command workflows**, not regular code.
> Help reviewers understand how your changes improve command reliability and security.

## Table of Contents

1. [What Makes Command PRs Different](#what-makes-command-prs-different)
2. [Template](#template)
3. [Quick Checklist](#quick-checklist)
4. [Writing Guidelines](#writing-guidelines)
5. [Scope Considerations](#scope-considerations)
6. [Tone & Manner](#tone--manner)
7. [Template Examples](#template-examples)

---

## What Makes Command PRs Different

Commands are procedural shortcuts for Claude Code. Different validation rules apply:

| Aspect | Regular Code PR | Command PR |
|--------|----------------|------------|
| **Content** | Source code | Markdown with frontmatter |
| **UI changes** | Screenshots required | No UI - skip screenshots |
| **Testing** | Unit/integration tests | Manual invocation + arg testing + CF/CP checks |
| **Quality metric** | Code correctness | Workflow reliability + security scoping |
| **Review focus** | Logic + performance | `allowed-tools` security + inline context + freedom calibration |

**Key principle**: Explain **why** the change improves command reliability or security, not just **what** changed.

---

## Template

### Core Components

#### **Context & Intent**

Explain **why** this change is needed in 1-2 sentences.

```markdown
❌ "Improve commit command"
✅ "Current commit command uses unrestricted Bash access. Scoped to
    git-only operations to follow CP10 (least privilege)."
```

#### **Type of Change**

Check one or more:

```markdown
- [ ] ✨ New command
- [ ] 🔧 Improve allowed-tools (security scoping)
- [ ] 📝 Update prompt logic (workflow improvement)
- [ ] 📚 Add examples or documentation
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
✅ "feat(deploy): add pre-deployment validation steps"
✅ "fix(commit): scope allowed-tools to git operations only"
✅ "docs(review): add argument usage examples"
❌ "commit command improvements"
```

#### **Testing & Validation**

Show how you verified quality:

```markdown
- [ ] Manual invocation tests (command works as expected)
- [ ] Argument testing (tried various $ARGUMENTS patterns)
- [ ] CF1-CF10 checks (structural)
- [ ] CP1-CP12 checks (quality)
- [ ] Edge case testing (no staged changes, invalid args, etc.)
- [ ] Command length ≤200 lines
```

**Be specific:**
```markdown
❌ "Testing complete"
✅ "Tested 5 scenarios locally:
    - /commit without args → ✅ generates message from git diff
    - /commit 'custom message' → ✅ uses provided message
    - /commit with no staged changes → ✅ shows helpful error
    - CF1-CF10: all pass
    - CP1-CP12: CP10 improved → scoped Bash to git-only"
```

---

## Quick Checklist

Verify before submitting:

**PR Completeness:**
- [ ] Explained **why** the change is needed
- [ ] Selected Type of Change checkboxes
- [ ] Testing & Validation is specific, not vague
- [ ] Included Before/After comparison (speeds review by 2x)

**Command Quality (CF/CP Checks):**
- [ ] Ran CF/CP checks yourself
- [ ] Command length ≤200 lines (20-150 typical)
- [ ] `allowed-tools` scoped to minimum necessary operations
- [ ] `argument-hint` present when using $ARGUMENTS
- [ ] Inline context (`!`, `@`) used for dynamic data
- [ ] No hardcoded secrets or absolute paths
- [ ] Error handling for edge cases (no files, invalid args)
- [ ] Freedom calibrated (destructive ops = exact steps)
- [ ] Scope-appropriate checks (📁 Project allows specific repo patterns, 🌐 Public requires generalization)

---

## Writing Guidelines

### Keep It Specific

Vague descriptions waste reviewer time. Be concrete:

```markdown
❌ "Improved command security"
✅ "Scoped allowed-tools from 'Bash' to 'Bash(git add:*), Bash(git commit:*)'
    following CP10 least privilege principle"

❌ "Fixed some issues"
✅ "Added error handling for 'no staged changes' scenario (CP8)"
```

### Use Metrics

Show measurable improvement when possible:

```markdown
- Security: allowed-tools unrestricted → scoped to 2 git operations
- Length: 180 lines → 95 lines (token efficiency)
- Edge cases: 0 handled → 3 handled (no files, empty message, merge conflict)
- CF/CP checks: 16/22 → 22/22
```

### Use HTML Comments for Template Hints

Include writing tips in the template that don't show up in the final PR:

```html
<!--
💡 Writing tips:
- allowed-tools changes: explain security improvement
- prompt logic changes: explain what scenarios now work better
- Before/After comparisons make reviews 2x faster
-->
```

---

## Scope Considerations

Commands have different requirements based on their scope:

| Scope | What It Means | CF/CP Check Flexibility |
|-------|---------------|------------------------|
| 📁 **Project** | One repo, specific workflow | Overfitting (CP7) acceptable - can reference specific paths/scripts |
| 🏠 **Personal** | Your projects, must generalize | Most checks apply |
| 👥 **Team** | Shared within org | + naming clarity, documentation |
| 🌐 **Public** | Published as plugin | All CF/CP checks mandatory |

**When to mention scope:**

If your PR involves trade-offs, explain the scope context:

```markdown
✅ "CP7 overfitting concern exists, but this is 📁 Project scope -
    command calls npm scripts specific to this repo's package.json"

✅ "CP10 security: used Bash(npm run:*) instead of restricting to specific
    scripts since this is 🏠 Personal scope across multiple projects"
```

---

## Tone & Manner

### 1. **Be a Helpful Guide: Share Intent**

Reviewers don't know your command's workflow. Provide context.

**Explain the "why":**
```markdown
❌ "Improve deploy command"
✅ "Current deploy command allows any Bash operation. Scoped to npm scripts
    only and added build→test→deploy validation sequence to prevent
    failed deployments"
```

**Show Before/After:**
```markdown
## Before
allowed-tools: Bash

## After
allowed-tools: Bash(npm run:*), Bash(git tag:*)

## Why
CP10 (least privilege) - unrestricted Bash allows rm -rf or other
destructive operations. Scoping to npm + git limits blast radius.
```

### 2. **Be Humble Yet Confident**

Explain your reasoning while staying open to feedback:

```markdown
✅ "From a CP3 (freedom calibration) perspective, destructive operations
    need exact steps. Open to suggestions for better safeguards."

❌ "This is the only secure way to do deployments"
```

### 3. **Self-Review Shows Respect**

Save reviewer time by checking your own work first.

**Run CF/CP checks yourself:**
```markdown
## Self-Review
CF1-CF10: ✅ all pass
CP1-CP12: ⚠️ CP10 needs improvement
  - L3: allowed-tools: Bash → unrestricted access
  - → Changed to Bash(git add:*), Bash(git commit:*)
  - Security improvement: unlimited → 2 git operations only
```

**Add self-comments:**
Add comments to your own PR immediately after submission:
```markdown
💭 "CP7 overfitting warning: hardcoded 'npm test' but this is 📁 Project
    scope so acceptable"
⚠️ "No automated tests - validated with manual invocation only"
🔍 "CP3 freedom mismatch concern: destructive deploy but high freedom.
    Should add exact step sequence."
```

### 4. **Feedback = Growth Opportunity**

PRs improve command quality through collaboration:

**Frame feedback constructively:**
- "This security scoping is wrong" ❌
- "Scoping allowed-tools to specific operations would improve security" ✅

**Use gentle emoji:**
- ✅ Agree, completed
- 💡 Suggestion
- 🤔 Question
- ⚠️ Attention needed
- 🔒 Security concern

---

## Template Examples

### Example 1: Improve Security (allowed-tools scoping)

```markdown
## Context & Intent
Current /commit command uses unrestricted Bash access (CP10 violation).
Scoping to git-only operations to follow least privilege principle.

## Type of Change
- [x] 🔧 Improve allowed-tools (security scoping)

## Related Issues
Fixes #123

## Changes
- Frontmatter: `allowed-tools: Bash` → `Bash(git add:*), Bash(git commit:*), Bash(git status:*)`
- No prompt logic changes - only security scoping

## Testing & Validation
- [x] Manual invocation - 5 scenarios
  - /commit → ✅ works (git operations allowed)
  - Tried `rm test.txt` via inline ! → ✅ blocked (not in allowed-tools)
  - /commit "message" → ✅ works
- [x] CF1-CF10: all pass
- [x] CP1-CP12: CP10 now passes (was failing before)

## Before/After

### Before
```yaml
allowed-tools: Bash
```

### After
```yaml
allowed-tools: Bash(git add:*), Bash(git commit:*), Bash(git status:*)
```

## Why
CP10 (security: least privilege) - unrestricted Bash allows destructive
operations like rm -rf. Scoped to only the git operations this command
actually needs. Blast radius reduced from "any shell command" to "3 git operations".
```

### Example 2: Add Error Handling

```markdown
## Context & Intent
/commit command fails silently when no files are staged. Added CP8 error
handling to provide helpful feedback instead of confusing behavior.

## Type of Change
- [x] 🐛 Fix bug

## Related Issues
Related to #456

## Changes
- Added `!git status --short` check before generating commit message
- Early exit with helpful message if no staged changes detected

## Testing & Validation
- [x] Edge case testing
  - No staged files → ✅ shows "No staged changes. Use git add first."
  - Staged files → ✅ generates commit message normally
  - Untracked files only → ✅ shows staged changes message
- [x] CF1-CF10: all pass
- [x] CP1-CP12: CP8 improved (error handling added)

## Example

### Before (confusing behavior)
```bash
User: /commit
Claude: [generates commit message for empty changeset]
User: git commit → "nothing to commit"
```

### After (helpful error)
```bash
User: /commit
Claude: "No staged changes detected. Use 'git add <files>' first."
```

## Why
CP8 (error handling) - commands should handle edge cases gracefully.
Silent failure on empty changeset wastes user time. Explicit error
message guides user to correct action.
```

---

**Core principle**: Command PRs are about **workflow reliability** and **security scoping**.
Clearly communicate why the change is needed and how it improves quality.
