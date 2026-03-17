# Skill & Command Reviewer

English | [한국어](README.ko.md)

High-quality reviewer skills for [Claude Code](https://code.claude.com) that evaluate and improve skills and custom commands against Anthropic's official best practices.

## What's Included

### 📋 [skill-reviewer](skills/skill-reviewer)

Evaluates Claude Code skills on **structural correctness** (F1-F12) and **principle quality** (P1-P12).

**Use when:**
- Creating a new skill
- Reviewing existing skills before publishing
- Contributing improvements to skill repositories
- Validating third-party skills before installation

**Modes:**
- Mode 1: Self-Review — validate your own skill
- Mode 2: External Review — evaluate someone else's skill
- Mode 3: Auto-PR — fork, improve, and submit PRs
- Mode 4: Create — generate new skills following best practices

### ⚙️ [command-reviewer](skills/command-reviewer)

Evaluates Claude Code custom slash commands on **structural correctness** (CF1-CF10) and **principle quality** (CP1-CP12).

**Use when:**
- Creating custom slash commands
- Reviewing `.claude/commands/*.md` files
- Deciding between command vs skill format
- Improving command security and reliability

**Modes:**
- Mode 1: Self-Review — validate your command
- Mode 2: External Review — evaluate someone else's command
- Mode 3: Create — generate new commands following best practices
- Mode 4: Auto-PR — contribute improvements to command repos

## Installation

### Option 1: Install from Smithery

Install directly from [Smithery](https://smithery.ai):

- [skill-reviewer](https://smithery.ai/skills/nyjin/skill-reviewer)
- [command-reviewer](https://smithery.ai/skills/nyjin/command-reviewer)

### Option 2: Install Both Skills

```bash
# Clone this repository
git clone https://github.com/nyjin/skill-reviewer.git

# Copy to your Claude skills directory
cp -r skill-reviewer/skills/* ~/.claude/skills/
```

### Option 3: Install Individual Skills

```bash
# Install only skill-reviewer
cp -r skill-reviewer/skills/skill-reviewer ~/.claude/skills/

# Or install only command-reviewer
cp -r skill-reviewer/skills/command-reviewer ~/.claude/skills/
```

## Usage

### Reviewing Skills

```
# Self-review your skill
/skill-reviewer

# Review a specific skill
/skill-reviewer Review the skill at ~/.claude/skills/my-skill

# Create a new skill
/skill-reviewer Create a skill for PDF processing
```

### Reviewing Commands

```
# Self-review your command
/command-reviewer

# Review a specific command
/command-reviewer Review the command at .claude/commands/deploy.md

# Create a new command
/command-reviewer Create a command for git commits
```

## Quality Checks

### Skills (F1-F12 + P1-P12)

**Form (Structural):**
- F1-F4: Frontmatter (name, description, triggers)
- F5-F11: Body structure (line count, paths, references)
- F12: Script error handling

**Principle (Quality):**
- P1: WHY explanations
- P2: Token efficiency
- P3: Freedom calibration
- P4: Consistent terminology
- P5: Concrete examples
- P6: Feedback loops
- P7: Generalization
- P8-P12: Metadata quality

### Commands (CF1-CF10 + CP1-CP12)

**Form (Structural):**
- CF1-CF6: Frontmatter (filename, YAML, description, allowed-tools)
- CF7-CF10: Security and portability

**Principle (Quality):**
- CP1: WHY explanations
- CP2: Token efficiency
- CP3: Freedom calibration
- CP4: Inline context (`!` and `@`)
- CP5: Output format
- CP6: Feedback loops
- CP7: Generalization
- CP8: Error handling
- CP9: Argument design
- CP10: Security (least privilege)
- CP11-CP12: Existence justification

## Scope Profiles

Both reviewers adapt checks based on scope:

- 📁 **Project**: Single repo, overfitting acceptable
- 🏠 **Personal**: Your own use, must generalize
- 👥 **Team**: Shared within org, + documentation
- 🌐 **Public**: Open-source, all checks mandatory

## Contributing

Improvements welcome! Both reviewer skills follow their own best practices. When contributing:

1. Run the reviewers on your changes
2. Follow F/P or CF/CP checks
3. Use the PR templates in `references/pull-request-guide.md`

## License

MIT License - see [LICENSE](LICENSE)

## References

- [Anthropic Agent Skills Best Practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- [Claude Code Documentation](https://code.claude.com/docs)
