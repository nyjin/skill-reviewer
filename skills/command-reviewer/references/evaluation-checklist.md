# Command Evaluation Checklist

Complete checklist for evaluating Claude Code custom slash commands. Two tiers: structural correctness and principle quality.

## Table of Contents

- [Scoring](#scoring)
- [Tier 1: Form (Structural Checks)](#tier-1-form-structural-checks)
- [Tier 2: Principle (Quality Checks)](#tier-2-principle-quality-checks)
- [Summary Scorecard](#summary-scorecard)

## Scoring

Each check is scored:
- ✅ **Pass** — meets the standard
- ⚠️ **Warn** — minor issue, command still functional
- ❌ **Fail** — significant issue affecting command quality or security

### Severity Guide

| Severity | Impact | Examples |
|----------|--------|----------|
| 🔴 Critical | Command fails or creates security risk | Malformed YAML, `Bash` without scoping on destructive ops |
| 🟡 High | Command works but behaves unpredictably | No output format, mismatched freedom, should be a skill |
| 🟠 Medium | Command works but wastes tokens or confuses | Verbose prompt, missing model selection, missing description |
| 🟢 Low | Minor polish, no functional impact | Naming convention, argument-hint missing |

---

## Tier 1: Form (Structural Checks)

### CF1: Filename Valid 🟢
```
Checks:
- [ ] Lowercase only
- [ ] Kebab-case (hyphens, not underscores)
- [ ] Verb-first when possible (commit.md, review-code.md, deploy-staging.md)
- [ ] Descriptive (not "helper.md", "do-stuff.md")
- [ ] No spaces or special characters

Why: The filename IS the command name. Users type /commit,
not /my-commit-helper-v2. Verb-first aids discoverability.

✅ commit.md, review-pr.md, fix-issue.md, deploy-staging.md
❌ helper.md, MyCommand.md, do_stuff.md, v2-thing.md
```

### CF2: Frontmatter Valid YAML 🔴
```
Checks:
- [ ] Opens and closes with --- on separate lines
- [ ] Valid YAML syntax (proper key: value pairs)
- [ ] No tabs (YAML requires spaces)
- [ ] String values with special chars are quoted

Why critical: Malformed YAML silently disables ALL frontmatter.
The command still runs but without description, allowed-tools,
model, etc. — it appears broken with no obvious error.

✅ ---
   description: Create a commit
   allowed-tools: Bash(git add:*)
   ---

❌ ---
   description: Create a commit  # ok
   allowed-tools:Bash(git add:*) # missing space after colon
   ---
```

### CF3: Description Present 🟠
```
Checks:
- [ ] description field exists in frontmatter
- [ ] Non-empty string
- [ ] Concise (one sentence)
- [ ] Describes what the command DOES, not how to use it

Why: Description appears in /help listing and autocomplete.
Without it, the command is invisible unless you remember
the exact name.

✅ description: Create a git commit with conventional message
❌ description: Use this command to create commits by running git add and then...
❌ (no description field at all)
```

### CF4: allowed-tools Appropriate 🔴
```
Checks:
- [ ] Present when command uses any tools
- [ ] Scoped to minimum necessary operations
- [ ] No bare `Bash` for destructive operations
- [ ] Matches the actual tools used in the prompt
- [ ] Read-only commands use Read, Grep, Glob (not Bash)

Why critical: allowed-tools grants permission WITHOUT per-use
approval. `Bash` alone means Claude can execute ANY shell command
including rm, curl to external servers, etc.

Scoping guide:
- Git operations → Bash(git add:*), Bash(git commit:*)
- Build/test → Bash(npm run:*), Bash(pytest:*)
- Read-only → Read, Grep, Glob
- Full shell (rare) → Bash (only for trusted, non-destructive commands)

✅ allowed-tools: Bash(git diff:*), Bash(git add:*), Bash(git commit:*)
❌ allowed-tools: Bash  (for a commit command — way too broad)
❌ (no allowed-tools, but prompt uses !`git diff`)
```

### CF5: argument-hint Present When Needed 🟢
```
Checks:
- [ ] If $ARGUMENTS or $1/$2 is used → argument-hint exists
- [ ] Hint describes what to pass
- [ ] Uses brackets for clarity: [message], [issue-number]
- [ ] Optional args indicated: [message?] or [optional: branch-name]

Why: Without argument-hint, the user has no guidance on what
to type after the command name. They have to read the source file.

✅ ---
   argument-hint: [commit-message]
   description: Create a commit
   ---
   Commit with message: $ARGUMENTS

❌ ---
   description: Fix an issue
   ---
   Fix issue #$1 with priority $2
   (user doesn't know to type "/fix-issue 123 high")
```

### CF6: Model Selection Appropriate 🟠
```
Checks:
- [ ] Simple tasks (commit, format, lint) → model: haiku
- [ ] Complex tasks (review, debug, refactor) → model: sonnet or opus
- [ ] If omitted, task complexity justifies default model cost

Why: Model defaults to conversation model (often Opus).
Simple git operations don't need Opus-level reasoning.
Explicit model: haiku runs faster and costs less.

Task → Model mapping:
- Formatting, committing, simple transforms → haiku
- Code review, debugging, refactoring → sonnet
- Architecture decisions, complex reasoning → opus
- Depends on context → omit (inherit conversation model)
```

### CF7: No Hardcoded Secrets 🔴
```
Checks:
- [ ] No API keys or tokens
- [ ] No passwords or credentials
- [ ] No personal identifiers
- [ ] Secrets referenced via environment variables if needed

Why: Commands in .claude/commands/ are typically committed
to git and shared with the team. Secrets in command files
get pushed to repositories.

❌ curl -H "Authorization: Bearer sk-abc123..."
✅ curl -H "Authorization: Bearer $API_TOKEN"
```

### CF8: No Hardcoded Absolute Paths 🟡
```
Checks:
- [ ] No /Users/username/...
- [ ] No /home/username/...
- [ ] No C:\Users\...
- [ ] Paths are relative or use arguments

Why: Project commands are shared via git. Absolute paths
fail on every other developer's machine.

❌ Read the config at /Users/john/projects/myapp/config.json
✅ Read the config at config.json
✅ Read the config at $1
```

### CF9: Reasonable Length 🟡
```
Checks:
- [ ] ≥20 lines (enough to justify a command file)
- [ ] ≤150 lines (single-file scope)
- [ ] If >200 lines → should probably be a skill

Boundary cases:
- <10 lines: "just type it" — a command file adds indirection
  without adding value
- 10-20 lines: borderline — acceptable if frequently repeated
- 20-150 lines: sweet spot for commands
- 150-200 lines: getting long — consider splitting
- >200 lines: convert to skill with SKILL.md + references/

Why: Commands are single-file prompt templates. If the prompt
needs reference files, progressive disclosure, or bundled
scripts, the skill format is more appropriate.
```

### CF10: Bash Commands Match allowed-tools 🟡
```
Checks:
- [ ] Every !`command` in body has matching allowed-tools entry
- [ ] Every @file reference has Read in allowed-tools
- [ ] No tools granted that aren't used (unnecessary permissions)

Why: Mismatched tools cause two problems:
1. !`command` without permission → fails or prompts every time
2. Granted but unused tools → unnecessary security exposure

✅ ---
   allowed-tools: Bash(git diff:*), Bash(git status:*)
   ---
   !`git status`
   !`git diff --cached`

❌ ---
   allowed-tools: Read
   ---
   !`git diff HEAD`  ← needs Bash(git diff:*), not Read
```

---

## Tier 2: Principle (Quality Checks)

### CP1: WHY Explained 🟡
```
Checks:
- [ ] Steps include reasoning, not just commands
- [ ] Claude understands intent behind each action
- [ ] Edge cases handleable because principle is clear

Why: When Claude understands WHY a step exists, it can
adapt when unexpected situations arise. Without WHY,
it follows steps mechanically even when they don't apply.

❌ 1. Run tests
   2. If pass, commit
   3. Push to origin

✅ 1. Run tests — abort if failing, broken code must not be committed
   2. If pass, create commit with conventional message
   3. Push to origin — verifies remote is accessible first
```

### CP2: Token-Efficient 🟡
```
Checks:
- [ ] No explaining git, npm, or other tools Claude knows
- [ ] No restating project context available in CLAUDE.md
- [ ] Direct instructions preferred over verbose explanations
- [ ] Each line justifies its token cost

Why: Command prompt is prepended to EVERY invocation.
100 tokens of wasted explanation × 50 daily invocations
= 5,000 wasted tokens per day.

❌ "Git is a version control system that tracks changes
    to your files. To commit changes, you first need to
    stage them using git add..."

✅ "Stage and commit changes with a conventional message."
```

### CP3: Freedom Calibrated 🟡
```
Checks:
- [ ] Destructive ops (push, deploy, delete) → exact steps, no deviation
- [ ] Creative ops (review, write) → direction only
- [ ] Mixed ops → structured with escape hatches

Assessment:
- "If Claude deviates, will something break?" → Low freedom
- "Are multiple approaches equally valid?" → High freedom

❌ High freedom for deploy: "Deploy however you see fit"
✅ Low freedom for deploy: exact step sequence with abort conditions
❌ Low freedom for code review: "Run exactly: python review.py --strict"
✅ High freedom for code review: "Review for bugs, readability, and conventions"
```

### CP4: Inline Context Effective 🟡
```
Checks:
- [ ] !`command` used for dynamic state (git diff, status, etc.)
- [ ] @file used for relevant project files
- [ ] Context is relevant — not dumping irrelevant data
- [ ] Context appears BEFORE the instructions that use it

Why: Static prompts lack current state. If a commit command
doesn't include the actual diff, Claude has to separately
run git diff — adding a tool call, approval prompt, and tokens.

✅ Pattern: context first, then instructions
   ## Current State
   !`git status`
   ## Changes
   !`git diff --cached`
   ## Task
   Create a conventional commit message based on the above.
```

### CP5: Output Format Specified 🟡
```
Checks:
- [ ] Expected output format or structure is described
- [ ] Examples of good output provided
- [ ] If multiple formats possible, one default is chosen

Why: Without format specification, Claude's output varies
between invocations. A commit command might produce
"feat: add auth" one time and "Added authentication
system with JWT tokens" another time.

❌ "Create a good commit message"
✅ "Create a commit message following Conventional Commits:
    <type>(<scope>): <description>
    Example: feat(auth): implement JWT-based authentication"
```

### CP6: Feedback Loops 🟡
```
Checks:
- [ ] Quality-critical steps have validate→fix→retry
- [ ] Build/test results checked before proceeding
- [ ] "Abort if X fails" conditions are explicit

Why: "Run tests then commit" without a failure check
means Claude commits even when tests fail. Explicit
abort conditions prevent cascading mistakes.

✅ 1. Run `npm test`
   2. If any test fails → STOP, do not proceed
   3. Only if all tests pass → create commit
```

### CP7: Not Overfitted 🟡
```
Checks:
- [ ] Works across different projects using this command
- [ ] No hardcoded project-specific values
- [ ] Detects framework/tooling dynamically when possible

Why: Project commands in .claude/commands/ are shared via git.
If a /commit command hardcodes "npm test", it breaks in
Python projects. Generic detection is more robust.

❌ "Run `npm test` to check"
✅ "Detect test framework and run appropriate test command"
   (for personal commands; project commands can be specific)

Note: Project-specific commands (.claude/commands/) CAN be
specific to that project. Personal commands (~/.claude/commands/)
should generalize across projects.
```

### CP8: Error Handling 🟠
```
Checks:
- [ ] Common edge cases addressed ("What if no staged changes?")
- [ ] Failure modes documented ("If build fails, abort")
- [ ] Recovery path provided when possible

Why: Commands run in varied states. A commit command invoked
with no staged changes should handle it gracefully rather
than creating an empty commit or failing cryptically.

✅ "If no files are staged, ask the user what to stage
    before proceeding."
```

### CP9: Arguments Well-Designed 🟠
```
Checks:
- [ ] $ARGUMENTS used for free-form input
- [ ] $1, $2 used for structured positional args
- [ ] Optional args have defaults or conditional logic
- [ ] Argument meaning is clear from argument-hint

Why: Poor argument design makes commands confusing to use.
If /fix-issue expects $1=issue-number and $2=priority,
the user needs to know this without reading the source.

✅ ---
   argument-hint: <issue-number> [priority=medium]
   ---
   Fix issue #$1.
   Priority: ${2:-medium}

❌ ---
   (no argument-hint)
   ---
   Process $1 with $2 using $3
```

### CP10: Security — Least Privilege 🔴
```
Checks:
- [ ] allowed-tools scoped to specific commands
- [ ] No bare `Bash` for commands with side effects
- [ ] Write operations explicitly listed, not blanket-granted
- [ ] disable-model-invocation: true for destructive commands

Why critical: allowed-tools bypass the approval system.
Overly broad permissions mean one prompt injection or
misunderstanding can execute destructive operations.

Privilege escalation pattern to watch for:
❌ allowed-tools: Bash                    ← can do ANYTHING
❌ allowed-tools: Bash, Read, Write, Edit ← can modify any file
✅ allowed-tools: Bash(git commit:*)      ← only git commits
✅ allowed-tools: Read, Grep             ← read-only
```

### CP11: Justifies Existence 🟠
```
Checks:
- [ ] Prompt is complex enough to benefit from a file
- [ ] Used frequently (at least weekly)
- [ ] Would be tedious to type repeatedly

Threshold questions:
- "Would I type this exact prompt often?" → Yes = command
- "Is the prompt ≤10 words?" → just type it
- "Do I need tool permissions or inline context?" → command

❌ ---
   description: Run tests
   ---
   Run the tests.
   (Just type "run the tests" in Claude Code)

✅ ---
   description: Create conventional commit with context
   allowed-tools: Bash(git diff:*), Bash(git status:*)
   ---
   [30 lines of structured commit workflow]
```

### CP12: Complements CLAUDE.md 🟠
```
Checks:
- [ ] Command doesn't restate CLAUDE.md project rules
- [ ] CLAUDE.md provides context, command provides procedure
- [ ] No conflicting instructions between the two

Why: CLAUDE.md is always loaded. If the command restates
"use Conventional Commits" that's already in CLAUDE.md,
those tokens are pure waste on every invocation.

CLAUDE.md → "This project uses Conventional Commits"
Command → "Create a commit message" (format already known)
```

---

## Summary Scorecard

```
Command: /___________________
File:    ___________________
Date:    ___________________

TIER 1: FORM (Structural)
CF1  filename valid       [ ] Pass  [ ] Fail
CF2  frontmatter valid    [ ] Pass  [ ] Fail
CF3  description present  [ ] Pass  [ ] Fail
CF4  allowed-tools proper [ ] Pass  [ ] Fail
CF5  argument-hint        [ ] Pass  [ ] Fail
CF6  model appropriate    [ ] Pass  [ ] Fail
CF7  no secrets           [ ] Pass  [ ] Fail
CF8  no absolute paths    [ ] Pass  [ ] Fail
CF9  reasonable length    [ ] Pass  [ ] Fail
CF10 bash matches tools   [ ] Pass  [ ] Fail

Form Score: ___/10

TIER 2: PRINCIPLE (Quality)
CP1  WHY explained        [ ] Pass  [ ] Warn  [ ] Fail
CP2  token-efficient      [ ] Pass  [ ] Warn  [ ] Fail
CP3  freedom calibrated   [ ] Pass  [ ] Warn  [ ] Fail
CP4  inline context       [ ] Pass  [ ] Warn  [ ] Fail
CP5  output format        [ ] Pass  [ ] Warn  [ ] Fail
CP6  feedback loops       [ ] Pass  [ ] Warn  [ ] Fail
CP7  not overfitted       [ ] Pass  [ ] Warn  [ ] Fail
CP8  error handling       [ ] Pass  [ ] Warn  [ ] Fail
CP9  arguments designed   [ ] Pass  [ ] Warn  [ ] Fail
CP10 least privilege      [ ] Pass  [ ] Warn  [ ] Fail
CP11 justifies existence  [ ] Pass  [ ] Warn  [ ] Fail
CP12 complements CLAUDE.md[ ] Pass  [ ] Warn  [ ] Fail

Principle Score: ___/12

OVERALL: Form ___/10 + Principle ___/12

COMMAND TYPE: [ ] Correct as command  [ ] Should be a skill
```

### Interpretation

| Score | Assessment |
|-------|-----------|
| 20-22 | Excellent — production-ready |
| 16-19 | Good — minor improvements recommended |
| 11-15 | Needs work — several issues to address |
| <11   | Significant revision needed |

Any 🔴 Critical failure should be fixed regardless of total score.
