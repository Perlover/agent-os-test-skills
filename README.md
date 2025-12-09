# Agent OS Skills Test Project

This is a test project to verify whether Claude Code subagents properly use Skills when the `Skill` tool is added to their tools list.

## Author's Note

By chance, I discovered that a well-known tool called [Agent OS](https://github.com/buildermethods/agent-os) for Claude Code does not work as intended. Recently, I noticed that it generates code and often ignores the Skills I wrote for my project. Not long ago, I started wondering: *what if subagents do not use Skills by default?*

I began researching this topic, and after a thorough investigation, I found that subagents work with Skills only under certain conditions. Specifically, when the YAML frontmatter of a subagent lists in the `tools:` section either an asterisk (`*`) or explicitly includes `Skill` among the listed tools. In that case, the subagent receives a special structure loaded by the main process, containing headers that specify the skill name and its description.

Once this structure is provided, the subagent decides on its own whether it needs to use the Skill. If it decides it does, the user will be prompted for permission to read the Skill file. And if the code is run with the `--dangerously-skip-permissions` option, then no prompt will appear — the Skill will be used automatically.

To back up my claims, I performed specific testing. I wrote code to verify the behavior I am describing. For this purpose, I created this project and conducted independent testing, confirming my conclusions. **You are welcome to verify them yourself if you wish.**

## Background

In the original Agent OS, subagents have a limited set of tools defined in their frontmatter (e.g., `tools: Write, Read, Bash, WebFetch, Playwright`). Based on analysis of Claude Code behavior, **subagents only receive skill headers when the `Skill` tool is explicitly listed in their tools**. Without it, subagents cannot access any skills.

A patched version of Agent OS adds `Skill` to the tools list of relevant subagents, allowing them to:
- Receive skill headers describing available skills
- Decide whether to load and use specific skills based on context
- Access user's custom skills

## Repository Branches

This repository has two branches for testing:

| Branch | Description |
|--------|-------------|
| `master` | Patched Agent OS with `Skill` tool in subagents |
| `original-agent-os` | Original Agent OS without `Skill` tool (demonstrates the bug) |

## Test Results

### Key Findings

1. **Subagents CAN use Skills** - but ONLY when `Skill` tool is in their tools list
2. **Original Agent OS has a bug** - subagents cannot access any skills, including user's custom coding standards
3. **Permission requests work** - subagent permission requests "bubble up" to the main process, allowing user confirmation
4. **Skill description matters** - realistic descriptions trigger better than artificial test markers

### Test Skill

A custom test skill is installed at `.claude/skills/skill-test-marker/SKILL.md` that requires JSDoc comments with `@skill-verified` tag for all JavaScript functions.

### Verification

After running `/implement-tasks`, check for the marker:
```bash
grep "@skill-verified" src/calculator.js
```

- **Output found** = Skills are working in subagents
- **No output** = Skills are NOT being used by subagents

---

## Recommended Testing Procedure

The recommended way to verify this bug and fix is to start with the `original-agent-os` branch:

### Phase 1: Test with Original Agent OS (demonstrates the bug)

#### Step 1: Clone this repository with the original-agent-os branch

```bash
git clone -b original-agent-os https://github.com/perlover/agent-os-test-skills.git
cd agent-os-test-skills
```

#### Step 2: Install original Agent OS (if not already installed)

```bash
curl -sSL https://raw.githubusercontent.com/buildermethods/agent-os/main/scripts/base-install.sh | bash
```

When prompted, select option **1** to perform a fresh installation.

#### Step 3: Verify original Agent OS has no Skill tool

```bash
grep "Skill" ~/.agent-os/profiles/default/agents/implementer.md
# Should return NO results
```

#### Step 4: Run the test

```bash
cd agent-os-test-skills
claude
# Run: /implement-tasks
# Specify: agent-os/specs/2025-01-01-test-feature
```

#### Step 5: Verify result

```bash
grep "@skill-verified" src/calculator.js
# Should return NO results - this demonstrates the bug!
```

---

### Phase 2: Test with Patched Agent OS (demonstrates the fix)

#### Step 1: Remove the test repository and re-clone

```bash
cd ..
rm -rf agent-os-test-skills
git clone -b original-agent-os https://github.com/perlover/agent-os-test-skills.git
cd agent-os-test-skills
```

#### Step 2: Install patched Agent OS

```bash
curl -sSL "https://raw.githubusercontent.com/Perlover/agent-os/fix/subagent-skills-and-claude-md/scripts/base-install.sh" \
    | sed 's|buildermethods/agent-os|Perlover/agent-os|g; s|/raw/main/|/raw/fix/subagent-skills-and-claude-md/|g; s|branch="main"|branch="fix/subagent-skills-and-claude-md"|g' \
    > /tmp/install-fixed.sh && bash /tmp/install-fixed.sh
```

When prompted, select option **1** to perform a fresh installation.

#### Step 3: Verify patched Agent OS has Skill tool

```bash
grep "Skill" ~/agent-os/profiles/default/agents/implementer.md
# Should show: tools: Write, Read, Bash, WebFetch, Playwright, Skill
```

#### Step 4: Reinstall Agent OS in the test project

```bash
rm -rf agent-os .claude/agents .claude/commands
rm -f src/calculator.js

~/agent-os/scripts/project-install.sh --standards-as-claude-code-skills true --use-claude-code-subagents true
```

#### Step 5: Run the test

```bash
claude
# Run: /implement-tasks
# Specify: agent-os/specs/2025-01-01-test-feature
# When prompted for skill permission, approve it
```

#### Step 6: Verify result

```bash
grep "@skill-verified" src/calculator.js
# Should return results - the fix works!
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

- [Original Agent OS Repository](https://github.com/buildermethods/agent-os)
- [Patched Agent OS Fork](https://github.com/Perlover/agent-os/tree/fix/subagent-skills-and-claude-md)

## License

This test project is provided for demonstration purposes.
