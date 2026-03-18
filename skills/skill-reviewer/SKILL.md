---
name: skill-reviewer
description: "Quality checker for Claude Code skills (SKILL.md) with 24-point checklist based on Anthropic's best practices. Validates structural correctness (YAML frontmatter, paths, line count) and principle quality (WHY explanations, freedom calibration, token efficiency). Use for skill review, linting, security audit, auto-PR improvements, or creating new agent skills."
---

# Skill Reviewer

Evaluates Claude Code skills on two dimensions: **structural correctness** (does it follow the format?) and **principle quality** (will it actually work well for an LLM?).

Many skills pass structural checks but fail in practice because they lack reasoning, mismatched freedom levels, or waste tokens on things Claude already knows. This reviewer catches both.

## Quick Reference

| Situation | Mode | Key Action |
|-----------|------|------------|
| Validate my skill before publishing | Self-Review | Run checklist → fix → re-validate |
| Evaluate someone else's skill | External Review | Clone → full audit → report |
| Contribute improvements to a repo | Auto-PR | Fork → additive changes → PR |

## Scope Profiles

Not every skill needs every check. Determine the scope before reviewing — it controls which checks are mandatory vs advisory.

| Profile | When | Location | Key Difference |
|---------|------|----------|---------------|
| 📁 Project | One repo only, solo or small team | `.claude/skills/` in project | Overfitting OK — meant for this project |
| 🏠 Personal | Your own use, all projects | `~/.claude/skills/` | Must generalize across projects |
| 👥 Team | Shared within company/org | Committed to team repos | + discoverability, naming, documentation |
| 🌐 Public | Open-source, marketplace, community | Published externally | All checks mandatory |

Check applicability: `📁+` = from Project up, `🏠+` = from Personal up, `👥+` = from Team up, `🌐` = Public only.

When reporting, mark advisory-only issues as 🟢 Low regardless of their usual severity. A project-specific skill that's "overfitted" (P7) is not a problem — it's the intent.

## Two-Tier Evaluation Model

### Tier 1: Form (Structural)

Objective checks that can be verified mechanically. These catch obvious problems.

| # | Check | Why It Matters | Scope |
|---|-------|----------------|-------|
| F1 | `name`: present, ≤64 chars, lowercase+hyphens | Claude uses this for slash-command routing | 📁+ |
| F2 | `description`: present, ≤1024 chars, non-empty | Claude's only basis for choosing this skill among 100+ | 📁+ |
| F3 | `description` in third-person | Injected into system prompt; first-person breaks coherence | 👥+ |
| F4 | `description` includes trigger conditions ("Use when...") | Without this, Claude under-triggers the skill | 🏠+ |
| F5 | SKILL.md body ≤500 lines | Beyond this, competes with conversation context for tokens | 📁+ |
| F6 | No Windows paths (`\`) | Breaks on Unix systems; forward slashes work everywhere | 🏠+ |
| F7 | No hardcoded absolute paths (`/Users/...`, `/home/...`) | Fails on any other machine | 🏠+ |
| F8 | No hardcoded secrets or API keys | Security risk if skill is shared | 📁+ |
| F9 | Reference depth ≤1 level from SKILL.md | Claude may `head -100` nested refs, losing information | 🏠+ |
| F10 | Reference files >100 lines have a table of contents | Claude can see scope even with partial reads | 👥+ |
| F11 | No unnecessary files (README.md, CHANGELOG, etc.) | Skills are for AI agents, not human onboarding | 🌐 |
| F12 | Scripts have explicit error handling | Bare failures force Claude to guess what went wrong | 🏠+ |

### Tier 2: Principle (Quality)

Judgment-based checks that determine whether the skill will actually perform well. These are what separate a mediocre skill from a great one.

| # | Check | Why It Matters | Scope |
|---|-------|----------------|-------|
| P1 | **WHY is explained** — rules include reasoning, not just commands | Claude can generalize from principles; bare rules break on edge cases | 🏠+ |
| P2 | **Token-efficient** — no explanations of things Claude already knows | Context window is shared; every wasted token degrades performance | 📁+ |
| P3 | **Freedom calibrated** — specificity matches task fragility | Over-constraining flexible tasks wastes effort; under-constraining fragile tasks causes errors | 📁+ |
| P4 | **Consistent terminology** — one term per concept throughout | Mixed terms ("field"/"box"/"element") confuse Claude's pattern matching | 👥+ |
| P5 | **Concrete examples** — input/output pairs, not abstract descriptions | Examples teach format and style more effectively than rules | 👥+ |
| P6 | **Feedback loops** — validate→fix→re-validate for quality-critical steps | Single-pass execution misses errors that compound downstream | 📁+ |
| P7 | **Not overfitted** — instructions generalize beyond test examples | Narrow rules work for known cases but fail on real-world variation | 🏠+ |
| P8 | **No time-sensitive info** — no dates, versions that will expire | Skills may be used months later; stale info produces wrong outputs | 👥+ |
| P9 | **Description is assertive** — slightly "pushy" to prevent under-triggering | Claude tends to skip skills when descriptions are too conservative | 🏠+ |
| P10 | **Default over options** — one recommended approach, not many choices | Multiple equal options cause decision paralysis in the agent | 👥+ |
| P11 | **Dependencies explicit** — required packages and tools are listed | Implicit assumptions fail silently in different environments | 🏠+ |
| P12 | **MCP tools fully qualified** — `ServerName:tool_name` format | Ambiguous tool names cause "tool not found" errors | 🏠+ |

Full checklist with scoring: `references/evaluation-checklist.md`

## Mode 1: Self-Review

Validate your own skill before publishing. Run all checks, fix issues, then re-run until clean. Present results using the Report Output Format below.

```
Self-Review Loop:
1. Run Tier 1 checks (structural)
   → Fix any failures
   → Re-run until all pass

2. Run Tier 2 checks (principle)
   → For each issue, decide: fix, justify, or accept
   → Re-run to confirm fixes didn't introduce new issues

3. Final validation
   → Read SKILL.md as if seeing it for the first time
   → Ask: "Would Claude know what to do with this?"

4. Output results using Report Output Format
   → Layer 1 (Overall Assessment) + Layer 2 (Section Summary) + Layer 3 (Issue Details)
```

The Tier 1 and Tier 2 tables above ARE the checklist. Read each row, verify, move on. Do not duplicate them here.

## Mode 2: External Review

Evaluate someone else's skill. The goal is understanding the author's intent before judging quality.

```
External Review Workflow:
1. [ ] Read ALL files before forming opinions
2. [ ] Identify the author's intent and target use case
3. [ ] Run Tier 1 checks (structural)
4. [ ] Run Tier 2 checks (principle)
5. [ ] Output results using Report Output Format:
       Layer 1: Overall Assessment (scores + severity counts)
       Layer 2: Section Summary (category status table)
       Layer 3: Issue Details (sorted 🔴→🟡→🟠→🟢)
```

### Report Output Format

This report applies three design principles. The template below is a reference, not a rigid format — adapt freely as long as these principles are preserved.

- **Pyramid Principle** (Minto) — Layer 1 delivers the verdict first. A busy user can stop here.
- **Progressive Disclosure** — Layer 2 shows which areas are clean (skip) vs problematic (drill down to Layer 3).
- **Severity-Ordered Triage** — Layer 3 sorts 🔴→🟡→🟠→🟢. Each issue includes Location, Problem, Impact, and Fix — enough to act immediately.

**Layer 1 → Layer 2 → Layer 3** (verdict → section scan → actionable details)

```markdown
# Skill Review: [skill-name]

## 1. Overall Assessment

| Tier | Score | Verdict |
|------|-------|---------|
| Form (Structural) | 9/12 | Good — minor issues |
| Principle (Quality) | 6/12 | Needs work |
| **Overall** | **15/24** | **Several principle issues to address** |

Issues found: 🔴 1 Critical · 🟡 3 High · 🟠 2 Medium · 🟢 1 Low

Strengths:
- [Always acknowledge what the skill does well before listing problems]
- [This sets a constructive tone and respects the author's effort]

## 2. Section Summary

Quick scan — which areas need attention and which are clean.

| Category | Status | Issues |
|----------|--------|--------|
| Frontmatter (F1-F4) | ⚠️ 1 issue | F4 missing triggers |
| Body & Structure (F5-F11) | ✅ Clean | — |
| Scripts (F12) | ⚠️ 1 issue | F12 bare except |
| WHY & Reasoning (P1) | ❌ 2 issues | P1 bare ALWAYS/NEVER |
| Efficiency (P2, P10) | ⚠️ 1 issue | P2 verbose |
| Calibration (P3, P7) | ❌ 1 issue | P3 freedom mismatch |
| Examples & Loops (P5, P6) | ✅ Clean | — |
| Metadata Quality (P9, P11, P12) | ⚠️ 1 issue | P11 deps missing |

## 3. Issue Details

Sorted by severity. Fix 🔴 Critical first — these can prevent the skill from working at all.

### 🔴 Critical

**P3: Freedom mismatch on database migration**
- Location: Line 45-52
- Problem: "Migrate however you see fit" for a destructive DB operation
- Impact: Claude may run unverified migration, causing data loss
- Fix:
  ```markdown
  # Before
  Migrate the database as needed.

  # After
  Run exactly this migration script — it includes backup and rollback:
    python scripts/migrate.py --verify --backup-first
  ```

### 🟡 High

**F4: Missing trigger conditions in description**
- Location: YAML frontmatter
- Problem: Description says "Processes PDFs" — no context for when to activate
- Impact: Claude will under-trigger this skill for related requests
- Fix:
  ```yaml
  # Before
  description: Processes PDF files.

  # After
  description: Extracts text and tables from PDF files, fills forms,
    merges documents. Use when working with PDFs or when the user
    mentions forms, document extraction, or PDF manipulation.
  ```

**P1: Bare ALWAYS/NEVER without reasoning (2 occurrences)**
- Location: Lines 78, 92
- Problem: "ALWAYS validate" / "NEVER skip backup" — no WHY
- Impact: Claude follows the rule but can't handle edge cases
- Fix: Add one sentence of reasoning after each rule

### 🟠 Medium

**P2: Verbose explanation of PDF basics**
- Location: Lines 15-22
- Problem: 150 tokens explaining what a PDF is — Claude knows this
- Impact: Wastes context window, pushes out conversation history
- Fix: Replace with direct code example (~50 tokens)

### 🟢 Low

**P11: Dependencies not listed**
- Location: (missing section)
- Problem: Requires pdfplumber but doesn't state this
- Impact: May fail on first run in a clean environment
- Fix: Add dependencies section with install command

## 4. Structure Recommendations (Optional)

💡 The following structure improvements are optional but recommended for 👥 Team / 🌐 Public skills:

| Recommendation | Current | Benefit if Applied |
|----------------|---------|-------------------|
| "When to Use" section | ❌ Missing | Improves Claude's activation accuracy |
| "What This Skill Does" section | ❌ Missing | Quick understanding for team members |
| "How to Use" with examples | ⚠️ Partial | Enables immediate usage |
| Concrete input/output examples | ❌ Missing | Most impactful for Claude's output quality |

**Note**: These are suggestions, not requirements. The P1-P12 checks already cover the essentials for Claude's performance. Structure recommendations primarily benefit human readers and team onboarding.

Apply based on your scope:
- 📁 Project / 🏠 Personal: Optional — minimal structure is acceptable
- 👥 Team / 🌐 Public: Recommended — improves discoverability and onboarding
```



## Mode 3: Auto-PR

Fork, improve, and submit a PR. Changes must be additive — removing content risks breaking existing users' workflows and disrespects the author's design decisions.

```
Auto-PR Workflow:
1. [ ] Fork repository
2. [ ] Create feature branch (improve/skill-name)
3. [ ] Run full review (Modes 1+2) to identify issues
4. [ ] Apply improvements — additive only
5. [ ] Run self-review on modified skill
6. [ ] Verify nothing was removed (respect check)
7. [ ] Create PR following references/pull-request-guide.md
```

**PR Guidelines**: See `references/pull-request-guide.md` for skill-specific PR best practices, including template, tone, and validation checklist.

### Additive Principle

Existing content is preserved because the author made deliberate choices, and other users may depend on the current behavior. Breaking their workflows is worse than leaving imperfections.

```
❌ "Removed metadata.json (non-standard)"
✅ "Added marketplace.json alongside metadata.json"

❌ "Rewrote README in English"
✅ "Added README.en.md (original Chinese README preserved)"

❌ "Fixed the incorrect description"
✅ "Improved description with trigger conditions for better discoverability"
```

## Mode 4: Create New Skill

Create a skill from scratch following all F1-F12 and P1-P12 best practices.

**When to use**: User asks to "create a skill", "write a skill", "build a skill", or "scaffold a skill"

**Process**: Delegate to `references/writer.md` which contains the full creation workflow:

```
1. Read references/writer.md
2. Follow Phase 1: Clarify Intent (goal, scope, freedom)
3. Follow Phase 3: Generate (apply F1-F12 + P1-P12)
4. Follow Phase 4: Verify (self-review against this SKILL.md)
5. Present the generated skill
```

The writer references this SKILL.md as the single source of truth for quality criteria. This ensures created skills automatically pass review.

**Key principle**: The writer doesn't duplicate the checklist — it reads this SKILL.md fresh each time. Any updates to F/P checks automatically apply to new skills.

**User signals for Mode 4**:
- "Create a skill for [capability]"
- "Write a skill that [provides X]"
- "I need a skill to [augment Y]"
- "Make me a skill for [context]"

If the user needs a command instead of a skill, refer them to command-reviewer.



## Common Issues & Fixes

### P1 Violation: Rules Without Reasoning

The most common and impactful problem. Claude can follow rules, but understanding WHY lets it handle cases the rule didn't anticipate.

```markdown
# ❌ Bare rule
ALWAYS validate XML after editing.

# ✅ Rule with reasoning
Validate XML after every edit. Invalid XML makes Word unable to open
the document, which means the user loses their work with no recovery path.
```

### P2 Violation: Explaining What Claude Knows

Every token that explains common knowledge pushes out conversation context.

```markdown
# ❌ Verbose (~150 tokens explaining PDFs)
PDF (Portable Document Format) is a common file format that contains
text, images, and other content. To extract text from a PDF, you'll
need a library...

# ✅ Concise (~50 tokens)
Extract text with pdfplumber:
  import pdfplumber
  with pdfplumber.open("file.pdf") as pdf:
      text = pdf.pages[0].extract_text()
```

### P3 Violation: Mismatched Freedom

```markdown
# ❌ Low freedom for a flexible task (code review)
Run exactly: python scripts/review.py --strict --all-files
Do not modify the command.

# ✅ High freedom for a flexible task
Review the code for:
- Potential bugs or edge cases
- Readability and maintainability
- Adherence to project conventions

# ❌ High freedom for a fragile task (DB migration)
You can migrate the database however you prefer.

# ✅ Low freedom for a fragile task
Run exactly this script — it includes backup and verification:
  python scripts/migrate.py --verify --backup
```

### F4 Violation: Missing Trigger Conditions

```yaml
# ❌ Claude doesn't know when to activate this
description: Processes PDF files.

# ✅ Clear activation context
description: Extracts text and tables from PDF files, fills forms,
  merges documents. Use when working with PDF files or when the user
  mentions PDFs, forms, or document extraction.
```

### P7 Violation: Overfitted Instructions

```markdown
# ❌ Narrow — only works for the test case
When the user asks about Q4 revenue, query the finance_q4_2024 table
and format as a bar chart with blue colors.

# ✅ General — works across situations
Query the appropriate table based on the user's time period.
Format visualizations to match the data type:
- Time series → line chart
- Comparisons → bar chart
- Proportions → pie chart
```

### P10 Violation: Too Many Options

```markdown
# ❌ Decision paralysis
You can use pypdf, pdfplumber, PyMuPDF, pdf2image, or camelot...

# ✅ Default with escape hatch
Use pdfplumber for text extraction.
For scanned PDFs requiring OCR, use pdf2image + pytesseract instead.
```

## References

- `references/evaluation-checklist.md` — Full checklist with scoring rubric and severity guide
- `references/pull-request-guide.md` — Skill-specific PR best practices for Mode 3 (Auto-PR)
- `references/writer.md` — Skill creation workflow for Mode 4 (Create)
- Anthropic official: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices
