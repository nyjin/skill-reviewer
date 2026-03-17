# Evaluation Checklist

Complete checklist for evaluating Claude Code skills. Two tiers: structural correctness and principle quality.

## Table of Contents

- [Scoring](#scoring)
- [Tier 1: Form (Structural Checks)](#tier-1-form-structural-checks)
- [Tier 2: Principle (Quality Checks)](#tier-2-principle-quality-checks)
- [Summary Scorecard](#summary-scorecard)

## Scoring

Each check is scored:
- ✅ **Pass** — meets the standard
- ⚠️ **Warn** — minor issue, skill still functional
- ❌ **Fail** — significant issue affecting skill quality

### Severity Guide

| Severity | Impact | Examples |
|----------|--------|----------|
| 🔴 Critical | Skill may not trigger or will malfunction | Missing description, >500 lines, broken references |
| 🟡 High | Skill works but produces inconsistent results | No WHY, mismatched freedom, overfitted rules |
| 🟠 Medium | Skill works but wastes tokens or confuses | Verbose explanations, inconsistent terms, too many options |
| 🟢 Low | Minor polish, no functional impact | Naming convention, TOC missing in short files |

---

## Tier 1: Form (Structural Checks)

Objective, mechanically verifiable. Run these first — if structural checks fail, principle checks are premature.

### F1: Name Field Valid 🔴
```
Checks:
- [ ] Present in YAML frontmatter
- [ ] ≤64 characters
- [ ] Lowercase letters, numbers, hyphens only
- [ ] No reserved words ("anthropic", "claude")
- [ ] Descriptive (not "helper", "utils", "tools")

Recommended: Gerund form (processing-pdfs, analyzing-spreadsheets)
```

### F2: Description Field Valid 🔴
```
Checks:
- [ ] Present and non-empty
- [ ] ≤1024 characters
- [ ] No XML tags

Why critical: This is Claude's ONLY basis for choosing this skill
from potentially 100+ available skills. Everything else loads after.
```

### F3: Third-Person Voice 🟡
```
Checks:
- [ ] No "I can help you..."
- [ ] No "You can use this to..."
- [ ] No "Browse YouTube..." (imperative directed at user)
- [ ] Uses "Processes...", "Extracts...", "Generates..."

Why: Description is injected into the system prompt.
First/second person creates incoherent voice shifts.
```

### F4: Trigger Conditions 🔴
```
Checks:
- [ ] Includes "Use when..." clause
- [ ] Mentions specific file types, keywords, or scenarios
- [ ] Assertive enough to prevent under-triggering

Test: Would Claude activate this skill for a vaguely related request?
If not, the triggers are too narrow.

❌ "Processes PDF files."
✅ "Extracts text and tables from PDF files, fills forms, merges
    documents. Use when working with PDF files or when the user
    mentions PDFs, forms, or document extraction."
```

### F5: Body Length 🔴
```
Checks:
- [ ] SKILL.md body ≤500 lines
- [ ] If approaching limit, content split into references/

Why: Once loaded, every line competes with conversation history
and other context for the finite context window.
```

### F6: No Windows Paths 🟢
```
Checks:
- [ ] No backslash paths (scripts\helper.py)
- [ ] All paths use forward slashes (scripts/helper.py)

Why: Forward slashes work on all platforms.
Backslashes break on Unix/Mac.
```

### F7: No Hardcoded Absolute Paths 🟡
```
Checks:
- [ ] No /Users/username/...
- [ ] No /home/username/...
- [ ] No C:\Users\...
- [ ] All paths are relative

Why: Absolute paths fail on any machine except the author's.
```

### F8: No Hardcoded Secrets 🔴
```
Checks:
- [ ] No API keys
- [ ] No tokens or passwords
- [ ] No personal/company-specific identifiers

Why: Skills may be shared publicly. Embedded secrets
get leaked into repositories and context windows.
```

### F9: Reference Depth ≤1 Level 🟡
```
Checks:
- [ ] SKILL.md → reference file (OK)
- [ ] SKILL.md → reference → another reference (NOT OK)
- [ ] All reference files linked directly from SKILL.md

Why: Claude may use `head -100` for nested references,
resulting in incomplete information and wrong decisions.

❌ SKILL.md → advanced.md → details.md → actual info
✅ SKILL.md → advanced.md (direct)
   SKILL.md → details.md (direct)
```

### F10: Long References Have TOC 🟢
```
Checks:
- [ ] Reference files >100 lines include a table of contents
- [ ] TOC lists all major sections

Why: Even with partial reads, Claude can see the full scope
and navigate to the relevant section.
```

### F11: No Unnecessary Files 🟠
```
Checks:
- [ ] No README.md (skills are for AI, not human onboarding)
- [ ] No INSTALLATION_GUIDE.md
- [ ] No CHANGELOG.md
- [ ] No LICENSE files (unless required by license terms)
- [ ] Only SKILL.md, scripts/, references/, assets/

Why: Extra files waste discovery time and may confuse
Claude about what to read.
```

### F12: Script Error Handling 🟡
```
Checks:
- [ ] No bare except / catch-all
- [ ] Specific exception types
- [ ] Helpful error messages (not just "Error occurred")
- [ ] Recovery paths where possible
- [ ] No "magic numbers" without explanation

Why: When scripts fail with unclear errors, Claude guesses
at the fix — often incorrectly. Specific errors enable
specific recovery.

❌ except: print("Error")
✅ except FileNotFoundError:
       print(f"{path} not found. Creating with defaults...")
```

---

## Tier 2: Principle (Quality Checks)

Requires reading comprehension and judgment. These determine whether the skill will perform well in practice.

### P1: WHY Is Explained 🟡
```
Checks:
- [ ] Rules include reasoning, not just commands
- [ ] No bare ALWAYS/NEVER in caps without justification
- [ ] Reader can understand the intent behind each instruction
- [ ] Edge cases are handleable because the principle is clear

Why this check exists: LLMs with reasoning about WHY can
generalize to situations the rule didn't anticipate.
Rules alone break on the first unexpected input.

❌ "ALWAYS validate XML after editing."
✅ "Validate XML after every edit. Invalid XML makes Word
    unable to open the document, causing unrecoverable data loss."
```

### P2: Token-Efficient 🟡
```
Checks:
- [ ] No explaining what PDFs are, how libraries work, etc.
- [ ] No restating common programming concepts
- [ ] Code examples preferred over verbose prose
- [ ] Each paragraph justifies its token cost

Test question: "Does Claude really need this explanation?"
If Claude already knows it, remove it.

❌ "PDF (Portable Document Format) is a common file format
    that contains text, images, and other content..." (~150 tokens)
✅ "Extract text with pdfplumber:" + code block (~50 tokens)
```

### P3: Freedom Calibrated 🟡
```
Checks:
- [ ] Fragile tasks (migrations, XML edits) → exact scripts, low freedom
- [ ] Flexible tasks (code review, writing) → direction only, high freedom
- [ ] Middle ground (report generation) → template + parameters

Assessment questions:
- "If Claude deviates, will something break?" → Low freedom
- "Are multiple approaches equally valid?" → High freedom
- "Is there a preferred pattern with room for adaptation?" → Medium

❌ Low freedom for code review: "Run exactly: python review.py --strict"
❌ High freedom for DB migration: "Migrate however you see fit"
```

### P4: Consistent Terminology 🟠
```
Checks:
- [ ] Same concept uses same word throughout
- [ ] No switching between synonyms for the same thing
- [ ] Glossary provided if domain terms are ambiguous

Common violations:
- "API endpoint" / "URL" / "API route" / "path"
- "field" / "box" / "element" / "control"
- "extract" / "pull" / "get" / "retrieve"
```

### P5: Concrete Examples 🟡
```
Checks:
- [ ] Input/output pairs for key operations
- [ ] Before/after for transformations
- [ ] Real-looking data, not placeholder abstractions

Why: Examples teach format, style, and expected behavior
more precisely than descriptive prose.

❌ "Generate appropriate commit messages"
✅ Example 1:
   Input: Added user authentication with JWT tokens
   Output: feat(auth): implement JWT-based authentication
```

### P6: Feedback Loops 🟡
```
Checks:
- [ ] Quality-critical steps have validate→fix→revalidate pattern
- [ ] Validation tool/method is specified
- [ ] "Only proceed when validation passes" is explicit
- [ ] Error messages guide the fix

Why: Single-pass execution accumulates errors.
Validation loops catch problems before they compound.

✅ Pattern:
   1. Make edits
   2. Run validator
   3. If fails → fix → go to step 2
   4. Only proceed when validator passes
```

### P7: Not Overfitted 🟡
```
Checks:
- [ ] Instructions work for inputs beyond the test set
- [ ] No hardcoded values specific to one example
- [ ] Patterns described, not just instances
- [ ] Skill would work for a user with different data

Test: Imagine 10 different users with 10 different inputs.
Would the skill guide Claude correctly for all of them?

❌ "When user asks about Q4 revenue, query finance_q4_2024"
✅ "Query the table matching the user's time period and metric"
```

### P8: No Time-Sensitive Info 🟠
```
Checks:
- [ ] No "before August 2025, use the old API"
- [ ] No specific version numbers that will change
- [ ] No "currently" without a mechanism to verify

If historical context is needed, use "Current/Legacy" pattern:
## Current method
Use v2 endpoint.
## Legacy (deprecated 2025-08)
<details>v1 endpoint was...</details>
```

### P9: Description Assertive Enough 🟡
```
Checks:
- [ ] Covers informal phrasings users might say
- [ ] Includes edge-case triggers
- [ ] Slightly "pushy" — Claude tends to under-trigger

Claude currently under-triggers skills. A conservative description
makes this worse. Better to over-trigger slightly and let the skill
gracefully handle irrelevant activations.

❌ "Builds dashboards."
✅ "Builds dashboards for data visualization. Use when the user
    mentions dashboards, charts, metrics display, data visualization,
    or wants to show any kind of data visually."
```

### P10: Default Over Options 🟠
```
Checks:
- [ ] One recommended approach, not a list of alternatives
- [ ] Escape hatch for the non-default case
- [ ] No "you can use X or Y or Z or..."

Why: Multiple equal options cause decision paralysis.
One clear default with an exception path is fastest.

❌ "Use pypdf, pdfplumber, PyMuPDF, or camelot..."
✅ "Use pdfplumber for text extraction.
    For scanned PDFs needing OCR, use pdf2image + pytesseract instead."
```

### P11: Dependencies Explicit 🟠
```
Checks:
- [ ] Required packages listed with install commands
- [ ] System tools mentioned (pandoc, LibreOffice, etc.)
- [ ] No assumption that tools are pre-installed

Why: Implicit assumptions fail silently. Claude may try to use
a tool that isn't installed and waste tokens debugging.
```

### P12: MCP Tools Fully Qualified 🟢
```
Checks:
- [ ] MCP tools use ServerName:tool_name format
- [ ] No bare tool names without server prefix

Why: Bare names cause "tool not found" errors when multiple
MCP servers are available.

❌ "Use the bigquery_schema tool"
✅ "Use the BigQuery:bigquery_schema tool"
```

---

## Summary Scorecard

```
Skill: ___________________
Date:  ___________________

TIER 1: FORM (Structural)
F1  name valid          [ ] Pass  [ ] Fail
F2  description valid   [ ] Pass  [ ] Fail
F3  third-person        [ ] Pass  [ ] Fail
F4  trigger conditions  [ ] Pass  [ ] Fail
F5  ≤500 lines          [ ] Pass  [ ] Fail
F6  no Windows paths    [ ] Pass  [ ] Fail
F7  no absolute paths   [ ] Pass  [ ] Fail
F8  no secrets          [ ] Pass  [ ] Fail
F9  ref depth ≤1        [ ] Pass  [ ] Fail
F10 long refs have TOC  [ ] Pass  [ ] Fail
F11 no unnecessary files[ ] Pass  [ ] Fail
F12 script error handling[ ] Pass [ ] Fail

Form Score: ___/12

TIER 2: PRINCIPLE (Quality)
P1  WHY explained       [ ] Pass  [ ] Warn  [ ] Fail
P2  token-efficient     [ ] Pass  [ ] Warn  [ ] Fail
P3  freedom calibrated  [ ] Pass  [ ] Warn  [ ] Fail
P4  consistent terms    [ ] Pass  [ ] Warn  [ ] Fail
P5  concrete examples   [ ] Pass  [ ] Warn  [ ] Fail
P6  feedback loops      [ ] Pass  [ ] Warn  [ ] Fail
P7  not overfitted      [ ] Pass  [ ] Warn  [ ] Fail
P8  no time-sensitive   [ ] Pass  [ ] Warn  [ ] Fail
P9  description assertive[ ] Pass [ ] Warn  [ ] Fail
P10 default over options[ ] Pass  [ ] Warn  [ ] Fail
P11 dependencies explicit[ ] Pass [ ] Warn  [ ] Fail
P12 MCP tools qualified [ ] Pass  [ ] Warn  [ ] Fail

Principle Score: ___/12

OVERALL: Form ___/12 + Principle ___/12
```

### Interpretation

| Score | Assessment |
|-------|-----------|
| 22-24 | Excellent — ready for production |
| 18-21 | Good — minor improvements recommended |
| 12-17 | Needs work — several issues to address |
| <12   | Significant rewrite needed |

Critical failures (any 🔴 item failing) should be fixed regardless of total score.
