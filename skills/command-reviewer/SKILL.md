---
name: command-reviewer
description: "Quality checker for Claude Code custom slash commands (.claude/commands/*.md) with 22-point checklist. Validates structural correctness (YAML frontmatter, allowed-tools) and principle quality (security scoping, inline context, freedom calibration). Use for command linting, review, security audit, creation, or command-vs-skill decisions."
---

# Command Reviewer

Evaluates Claude Code custom slash commands on two dimensions: **structural correctness** (is the command file well-formed?) and **principle quality** (will it work reliably and securely?).

Custom slash commands are markdown files in `.claude/commands/` (project) or `~/.claude/commands/` (personal). The filename becomes the command name: `commit.md` → `/commit`. They differ from skills in important ways, and this reviewer applies evaluation criteria specific to commands.

## Quick Reference

| Situation | Mode | Key Action |
|-----------|------|------------|
| Validate my command before sharing | Self-Review | Checklist → fix → re-validate |
| Evaluate someone else's command | External Review | Read → full audit → report |
| Decide: command or skill? | Decision Guide | See "Command vs Skill" section |

## Command vs Skill: When to Use What

Before reviewing, determine if a command is the right format. Some commands should be skills, and vice versa.

| Signal | → Command | → Skill |
|--------|-----------|---------|
| Who triggers it? | User types `/name` | Claude decides automatically |
| How complex? | Single-file prompt | Multi-file with references |
| How often? | Frequently repeated workflow | Situational capability |
| Side effects? | Yes (git, deploy, send) | Usually read/generate |
| Arguments? | Needs `$ARGUMENTS` | Context-driven |

```
Use a COMMAND when:
- You trigger it manually with /name
- It wraps a specific, repeatable procedure (commit, deploy, review)
- It needs allowed-tools for shell operations
- It's short enough for a single markdown file

Use a SKILL when:
- Claude should activate it automatically based on context
- It needs reference files, scripts, or templates
- It's knowledge/guidance rather than a procedure
- It exceeds ~200 lines (needs progressive disclosure)
```

If the reviewed command should actually be a skill, note this as a 🟡 High issue in the report.

## Scope Profiles

Not every command needs every check. Determine the scope before reviewing — it controls which checks are mandatory vs advisory.

| Profile | When | Location | Key Difference |
|---------|------|----------|---------------|
| 📁 Project | One repo only, personal project | `.claude/commands/` solo project | Overfitting OK — meant for this project |
| 🏠 Personal | Your own use, all projects | `~/.claude/commands/` | Must generalize across projects |
| 👥 Team | Shared within company/org | `.claude/commands/` in team repo | + discoverability, naming, onboarding |
| 🌐 Public | Published as plugin or shared widely | Published externally | All checks mandatory |

Check applicability: `📁+` = from Project up, `🏠+` = from Personal up, `👥+` = from Team up, `🌐` = Public only.

When reporting, mark advisory-only issues as 🟢 Low regardless of their usual severity. A project-specific command with hardcoded `npm test` (CP7) is not overfitted — it's the intent.

## Two-Tier Evaluation Model

### Tier 1: Form (Structural)

Objective checks for command file correctness.

| # | Check | Why It Matters | Scope |
|---|-------|----------------|-------|
| CF1 | Filename: lowercase, kebab-case, verb-first | Filename IS the command name; `commit.md` > `my-helper.md` | 👥+ |
| CF2 | Frontmatter: valid YAML syntax between `---` markers | Malformed YAML silently disables all frontmatter options | 📁+ |
| CF3 | `description` present | Appears in `/help` and autocomplete; missing = invisible command | 🏠+ |
| CF4 | `allowed-tools` appropriate | Too broad = security risk; missing = approval prompts every time | 📁+ |
| CF5 | `argument-hint` present when `$ARGUMENTS`/`$1` used | Without hint, user doesn't know what to pass | 👥+ |
| CF6 | `model` selection matches task complexity | Haiku for simple tasks saves tokens; Opus for complex reasoning | 🏠+ |
| CF7 | No hardcoded secrets or API keys | Commands may be committed to shared repos | 📁+ |
| CF8 | No hardcoded absolute paths | Fails on other machines; use relative paths or `$ARGUMENTS` | 🏠+ |
| CF9 | Reasonable length (20-150 lines typical) | <10 lines = just type it; >200 lines = should be a skill | 🏠+ |
| CF10 | `!` bash commands have matching `allowed-tools` | `\!`git diff`` without `Bash(git diff:*)` fails silently | 📁+ |

### Tier 2: Principle (Quality)

Judgment-based checks that determine real-world effectiveness.

| # | Check | Why It Matters | Scope |
|---|-------|----------------|-------|
| CP1 | **WHY explained** — steps include reasoning | Claude executes more reliably when it understands the intent | 🏠+ |
| CP2 | **Token-efficient** — no verbose explanations | Command prompt adds to every invocation's token cost | 📁+ |
| CP3 | **Freedom calibrated** — specificity matches task fragility | Destructive operations (git push, deploy) need exact steps | 📁+ |
| CP4 | **Inline context effective** — `!` and `@` used for dynamic data | Static prompts miss current state; `\!`git diff`` provides live context | 🏠+ |
| CP5 | **Output format specified** — expected result format is clear | Without format guidance, output varies unpredictably | 🏠+ |
| CP6 | **Feedback loops** — validate→fix→retry for critical steps | Commits without test pass, deploys without build check | 📁+ |
| CP7 | **Not overfitted** — works across projects/scenarios | Command hardcoded to one repo structure fails elsewhere | 🏠+ |
| CP8 | **Error handling** — edge cases and failures addressed | "What if no staged changes?" "What if branch already exists?" | 🏠+ |
| CP9 | **Arguments well-designed** — meaningful `$1`/`$2`, defaults for optional | Poor arg design leads to confusing invocations | 👥+ |
| CP10 | **Security: least privilege** — `allowed-tools` scoped minimally | `Bash` allows everything; `Bash(git commit:*)` allows only commits | 📁+ |
| CP11 | **Justifies existence** — not trivially typed | A 2-word prompt doesn't need a command file | 🏠+ |
| CP12 | **Complements CLAUDE.md** — no duplication with project context | Commands are procedures; CLAUDE.md is context. Don't mix. | 📁+ |

Full checklist with scoring: `references/evaluation-checklist.md`

## Mode 1: Self-Review

Validate your command before sharing with the team. Present results using the Report Output Format below.

```
Self-Review Loop:
1. Run Tier 1 checks (structural)
   → Fix any failures
   → Re-run until all pass

2. Run Tier 2 checks (principle)
   → For each issue, decide: fix, justify, or accept
   → Re-run to confirm fixes didn't introduce new issues

3. Command vs Skill check
   → Is this really a command, or should it be a skill?

4. Output results using Report Output Format
   → Layer 1 (Overall Assessment) + Layer 2 (Section Summary) + Layer 3 (Issue Details)

5. If preparing a PR, follow references/pull-request-guide.md
   → Command-specific PR template and best practices
```

The Tier 1 and Tier 2 tables above ARE the checklist. Read each row, verify, move on. Do not duplicate them here.

## Mode 2: External Review

Evaluate someone else's command. Understand the workflow it encodes before judging.

```
External Review Workflow:
1. [ ] Read the command file completely
2. [ ] Identify the workflow it automates
3. [ ] Run Tier 1 checks (structural)
4. [ ] Run Tier 2 checks (principle)
5. [ ] Output results using Report Output Format:
       Layer 1: Overall Assessment (scores + severity counts)
       Layer 2: Section Summary (category status table)
       Layer 3: Issue Details (sorted 🔴→🟡→🟠→🟢)
```

### Report Output Format

Apply three design principles:

- **Pyramid Principle** — Layer 1 (verdict) → Layer 2 (section scan) → Layer 3 (details)
- **Progressive Disclosure** — Only expand ⚠️/❌ sections
- **Severity-Ordered Triage** — 🔴→🟡→🟠→🟢 with Location, Problem, Impact, Fix

**Minimal Example:**

```markdown
# Command Review: /commit

## 1. Overall Assessment
| Tier | Score | Verdict |
|------|-------|---------|
| Form (Structural) | 8/10 | Good |
| Principle (Quality) | 7/12 | Needs work |
| **Overall** | **15/22** | **Security tightening needed** |

Issues: 🔴 0 · 🟡 2 · 🟠 1 · 🟢 1

Command type: ✅ Correctly a command

Strengths:
- Uses inline context (`!git diff`) effectively
- Good argument design with defaults

## 2. Section Summary
| Category | Status | Issues |
|----------|--------|--------|
| Frontmatter (CF3-CF6) | ⚠️ 1 | CF4 over-broad tools |
| Safety (CP6-CP10) | ❌ 1 | CP10 Bash too broad |

## 3. Issue Details

### 🟡 High

**CP10: allowed-tools too broad**
- Location: Frontmatter
- Problem: `allowed-tools: Bash` grants unrestricted access
- Impact: Can execute any shell command without approval
- Fix:
  ```yaml
  # Before
  allowed-tools: Bash

  # After
  allowed-tools: Bash(git add:*), Bash(git commit:*)
  ```

**CF4: allowed-tools over-broad**
- Same as CP10 — security and structural overlap
- Fix: Scope to specific git operations

### 🟠 Medium

**CP5: No output format**
- Location: Body
- Problem: Doesn't specify commit message format
- Impact: Output varies (conventional vs freeform)
- Fix: Add format example (e.g., "Use conventional commits format")
```

## Mode 3: Create New Command

Create a command from scratch following all CF1-CF10 and CP1-CP12 best practices.

**When to use**: User asks to "create a command", "write a command", "build a command", or "scaffold a command"

**Process**: Delegate to `references/writer.md` which contains the full creation workflow:

```
1. Read references/writer.md
2. Follow Phase 1: Clarify Intent (goal, scope, freedom)
3. Follow Phase 3: Generate (apply CF1-CF10 + CP1-CP12)
4. Follow Phase 4: Verify (self-review against this SKILL.md)
5. Present the generated command
6. If submitting to a repo, follow references/pull-request-guide.md
```

The writer references this SKILL.md as the single source of truth for quality criteria. This ensures created commands automatically pass review.

**Key principle**: The writer doesn't duplicate the checklist — it reads this SKILL.md fresh each time. Any updates to CF/CP checks automatically apply to new commands.

**PR Guidelines**: See `references/pull-request-guide.md` for command-specific PR best practices.

**User signals for Mode 3**:
- "Create a command for [task]"
- "Write a command that [does X]"
- "I need a command to [automate Y]"
- "Make me a command for [workflow]"

If the user needs a skill instead of a command, refer them to skill-reviewer.

## Mode 4: Auto-PR (Optional)

Fork an external command repository, improve it, and submit a pull request.

**When to use**: User wants to contribute improvements to someone else's command repository

**Process**:

```
Auto-PR Workflow:
1. [ ] Fork repository
2. [ ] Create feature branch (improve/command-name)
3. [ ] Run Mode 2 (External Review) to identify issues
4. [ ] Apply improvements following CF/CP checks
5. [ ] Run Mode 1 (Self-Review) on modified command
6. [ ] Create PR following references/pull-request-guide.md
```

**Key principle**: Focus on security (CP10), error handling (CP8), and inline context (CP4) improvements. These have the highest impact on command reliability.

**PR Guidelines**: See `references/pull-request-guide.md` for command-specific PR template, including:
- CF/CP check results
- Security improvements (allowed-tools scoping)
- Before/After examples
- Testing with various arguments

**User signals for Mode 4**:
- "Contribute this improvement to the original repo"
- "Submit a PR with these fixes"
- "Create a pull request for this command"

If the command is fundamentally flawed (should be a skill, violates core principles), recommend creating a new command instead of PR.

## Common Issues & Fixes

### CP10 Violation: Over-Broad allowed-tools

The most common security issue. `Bash` without scoping grants full shell access.

```yaml
# ❌ Unrestricted — can run ANY shell command
allowed-tools: Bash

# ✅ Scoped — only git operations
allowed-tools: Bash(git add:*), Bash(git commit:*), Bash(git status:*)

# ✅ Scoped — read-only operations
allowed-tools: Read, Grep, Glob
```

Claude grants these tools without per-use approval when scoped. Broad scoping means Claude can `rm -rf` without asking. Always scope to the minimum operations the command actually needs.

### CF10 Violation: Bash Commands Without Matching allowed-tools

```markdown
# ❌ Uses git diff inline but doesn't declare permission
---
description: Review changes
---
Review these changes:
\!`git diff HEAD`

# ✅ Declares the tool it uses
---
description: Review changes
allowed-tools: Bash(git diff:*)
---
Review these changes:
\!`git diff HEAD`
```

Without matching allowed-tools, the inline `!` command either fails silently or prompts for approval every time, defeating the purpose of a command.

### CP4 Violation: Missing Inline Context

```markdown
# ❌ Static — doesn't know current state
---
description: Create a commit
---
Create a good commit message for the changes.

# ✅ Dynamic — includes current state
---
description: Create a commit
allowed-tools: Bash(git diff:*), Bash(git status:*)
---
## Current State
\!`git status`

## Changes
\!`git diff --cached`

Create a conventional commit message based on the staged changes above.
If $ARGUMENTS is provided, use it as the commit message.
```

Commands without inline context force Claude to independently gather information, wasting tokens and adding approval prompts. `!` and `@` provide context directly in the prompt.

### CP3 Violation: Mismatched Freedom

```markdown
# ❌ High freedom for destructive operation
---
description: Deploy to production
allowed-tools: Bash
---
Deploy the application to production.

# ✅ Low freedom with explicit steps and safeguards
---
description: Deploy to production
disable-model-invocation: true
allowed-tools: Bash(npm run:*), Bash(git tag:*)
---
Deploy to production following these exact steps:
1. Run `npm run build` — abort if build fails
2. Run `npm run test` — abort if any test fails
3. Tag the release: `git tag v$1`
4. Run `npm run deploy:production`
5. Verify deployment: `curl -s https://api.example.com/health`

Do NOT skip steps. Do NOT proceed if any step fails.
Destructive operations require this exact sequence because
a failed deploy with no rollback plan causes downtime.
```

### CF9 Violation: Wrong Format

```markdown
# ❌ Too short — just type it
---
description: Run tests
---
Run the tests.

# Better: just type "run the tests" in Claude Code

# ❌ Too long — should be a skill
---
description: Full code review process
---
[250+ lines of detailed instructions, checklists,
 reference material, output templates...]

# Better: convert to a skill with SKILL.md + references/
```

Commands should encode workflows that are **frequently repeated** and **complex enough to benefit from structure** but **simple enough for a single file**. The sweet spot is 20-150 lines.

## References

- `references/evaluation-checklist.md` — Full checklist with scoring rubric and severity guide
- `references/pull-request-guide.md` — Command-specific PR best practices
- `references/writer.md` — Command creation workflow for Mode 3 (Create)
- Claude Code docs: https://code.claude.com/docs/en/slash-commands
