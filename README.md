# Agent OS Skills Test Project

This is a test project to verify whether Claude Code subagents properly use Skills when the `Skill` tool is added to their tools list.

## Background

In the original Agent OS, subagents have a limited set of tools defined in their frontmatter (e.g., `tools: Write, Read, Bash, WebFetch, Playwright`). Based on analysis of Claude Code behavior, **subagents only receive skill headers when the `Skill` tool is explicitly listed in their tools**. Without it, subagents cannot access any skills.

A patched version of Agent OS adds `Skill` to the tools list of relevant subagents, allowing them to:
- Receive skill headers describing available skills
- Decide whether to load and use specific skills based on context
- Access user's custom skills

## Test Results

### Key Findings

1. **Subagents CAN use Skills** - when `Skill` tool is in their tools list, subagents can request and use skills
2. **Permission requests work** - subagent permission requests "bubble up" to the main process, allowing user confirmation
3. **Skill description matters** - skills with realistic, relevant descriptions (like "JSDoc documentation standard") are more likely to be triggered than artificial test markers
4. **"PROACTIVE" keyword helps** - but only when combined with a relevant description

### Test Skill

A custom test skill is installed at `.claude/skills/skill-test-marker/SKILL.md` that requires JSDoc comments with `@skill-verified` tag for all JavaScript functions.

### Verification

After running `/implement-tasks`, check for the marker:
```bash
grep "@skill-verified" src/calculator.js
```

- **Output found** = Skills are working in subagents
- **No output** = Skills are NOT being used by subagents

## Current State

**This project is installed with a PATCHED version of Agent OS** that includes the `Skill` tool in subagents.

You can verify this by checking:
```bash
grep "Skill" .claude/agents/agent-os/implementer.md
```

## How to Run the Test

1. Open this project in Claude Code:
   ```bash
   cd /path/to/agent-os-test-skills
   claude
   ```

2. Run the implement-tasks command:
   ```
   /implement-tasks
   ```

3. When asked about the spec, specify:
   ```
   agent-os/specs/2025-01-01-test-feature
   ```

4. **Important**: When prompted for skill permission, approve it

5. After completion, verify the result:
   ```bash
   grep "@skill-verified" src/calculator.js
   ```

## Expected Results

### With PATCHED Agent OS (Skill tool in subagents):
✅ The created `src/calculator.js` should contain `@skill-verified` tags in JSDoc comments

### With ORIGINAL Agent OS (no Skill tool in subagents):
❌ The created `src/calculator.js` will NOT contain `@skill-verified` tags

---

## Testing with Original Agent OS

If you want to verify that the original (unpatched) Agent OS does NOT use skills, follow these steps:

### Step 1: Install Original Agent OS

```bash
# Backup your current ~/agent-os if needed
mv ~/agent-os ~/agent-os-patched-backup

# Install fresh original Agent OS
curl -sSL https://raw.githubusercontent.com/buildermethods/agent-os/main/scripts/base-install.sh | bash
```

### Step 2: Reset This Test Project

```bash
cd /path/to/agent-os-test-skills

# Remove current Agent OS installation from project
rm -rf agent-os
rm -rf .claude/agents
rm -rf .claude/commands

# Keep the test skill!
# .claude/skills/skill-test-marker should remain

# Remove any generated code
rm -f src/calculator.js

# Reinstall Agent OS with skills enabled
~/agent-os/scripts/project-install.sh --standards-as-claude-code-skills true --use-claude-code-subagents true
```

### Step 3: Verify Original Agent OS Has No Skill Tool

```bash
grep "Skill" .claude/agents/agent-os/implementer.md
# Should return NO results or only show Playwright tools
```

### Step 4: Re-run the Test

```bash
claude
# Then run: /implement-tasks
# Specify: agent-os/specs/2025-01-01-test-feature
```

### Step 5: Verify Result

```bash
grep "@skill-verified" src/calculator.js
# Should return NO results with original Agent OS
```

---

## Restoring Patched Agent OS

To restore the patched version:

```bash
# If you backed up
rm -rf ~/agent-os
mv ~/agent-os-patched-backup ~/agent-os

# Then reinstall in project
cd /path/to/agent-os-test-skills
rm -rf agent-os .claude/agents .claude/commands
rm -f src/calculator.js
~/agent-os/scripts/project-install.sh --standards-as-claude-code-skills true --use-claude-code-subagents true
```

---

## Patched Files

The following Agent OS subagent files were modified to include `Skill` in their tools:

| File | Original tools | Patched tools |
|------|---------------|---------------|
| `implementer.md` | Write, Read, Bash, WebFetch, Playwright | + Skill |
| `spec-writer.md` | Write, Read, Bash, WebFetch | + Skill |
| `spec-verifier.md` | Write, Read, Bash, WebFetch | + Skill |
| `spec-shaper.md` | Write, Read, Bash, WebFetch | + Skill |
| `tasks-list-creator.md` | Write, Read, Bash, WebFetch | + Skill |

## Writing Effective Skills for Subagents

Based on testing, skills that work well with subagents should:

1. **Have realistic descriptions** - describe actual coding standards, not test markers
2. **Use "PROACTIVE" keyword** - signals that skill should be loaded automatically
3. **Be relevant to the task** - skills about JavaScript documentation work when writing JavaScript code
4. **Provide clear requirements** - specific tags or patterns that can be verified

Example of effective skill description:
```
PROACTIVE - Use when writing JavaScript/TypeScript code. Requires JSDoc comments with @skill-verified tag for all functions. Essential for code documentation and IDE support.
```

## Links

- [Agent OS Repository](https://github.com/buildermethods/agent-os)

## License

This test project is provided for demonstration purposes.
