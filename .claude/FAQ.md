# Claude Code FAQ

## Setup Questions

### Q: How do I install Claude Code?

Follow the official installation guide at: https://docs.anthropic.com/claude/docs/claude-code

### Q: What if I don't have `.claude/settings.local.json`?

Copy the example file:
```bash
cp .claude/settings.local.json.example .claude/settings.local.json
```

This file is git-ignored and stores your personal preferences.

### Q: Why are some files in `.claude/` not in my repo?

The `.gitignore` has `.claude` excluded. However, we explicitly commit most `.claude/` files to share team configuration. Only `settings.local.json` is truly git-ignored.

---

## Permission Questions

### Q: Why am I getting "Permission denied" errors?

Claude Code asks for permission before potentially dangerous operations. You have three options:

1. **Allow once**: Grant permission for this operation
2. **Always allow**: Add to your `.claude/settings.local.json`:
   ```json
   {
     "permissions": {
       "allow": ["Bash(git commit*)"]
     }
   }
   ```
3. **Deny**: Skip the operation

### Q: How do I allow git commits without asking?

Add to your `.claude/settings.local.json`:
```json
{
  "permissions": {
    "allow": [
      "Bash(git commit*)",
      "Bash(git add*)"
    ]
  }
}
```

### Q: What's the difference between `settings.json` and `settings.local.json`?

- **`.claude/settings.json`**: Team-wide permissions (committed to repo)
- **`.claude/settings.local.json`**: Your personal overrides (git-ignored)

Local settings override team settings.

### Q: Can I be less restrictive than team settings?

Yes, your local settings can be more permissive:
```json
{
  "permissions": {
    "allow": [
      "Bash(pnpm install*)",
      "WebFetch(domain:*)"
    ]
  }
}
```

### Q: How do I deny a command allowed by team settings?

Add to your `.claude/settings.local.json`:
```json
{
  "permissions": {
    "deny": ["Bash(pnpm clean*)"]
  }
}
```

## Model Questions

### Q: Which model should I use?

Default is Sonnet 4.5 (good balance of speed and quality).

**Use Opus** for:
- Complex architectural decisions
- Comprehensive security reviews
- Multi-file refactoring

**Use Haiku** for:
- Quick, simple tasks
- Fast iterations
- Simple commands

Configure in `.claude/settings.local.json`:
```json
{
  "defaultModel": "claude-opus-4-5-20251101"
}
```

### Q: Can I override model per command?

Some commands specify their preferred model in frontmatter:
```yaml
---
Model: claude-sonnet-4.5-20250929
---
```

You can override in your local settings if needed.

### Q: What's the thinking budget?

Controls how much Claude "thinks" before responding:
- `low`: Fast, simple responses
- `medium`: Balanced (default)
- `high`: Deep analysis, complex problems

Configure in `.claude/settings.local.json`:
```json
{
  "thinking": {
    "enabled": true,
    "budget": "high"
  }
}
```

## Skill Questions

### Q: Can I disable skills?

Skills are baked into the setup, but you can:
1. Override specific behaviors in commands
2. Provide explicit instructions to Claude
3. Modify skill files (advanced)

### Q: Do skills slow down Claude?

No, skills are just context. They help Claude make better decisions without slowing responses.

---

## Agent Questions

### Q: When should I use agents vs regular Claude?

Use agents for:
- **Focused tasks**: test-fixer for failing tests
- **Comprehensive work**: test-writer for full test coverage
- **Thorough analysis**: code-reviewer for detailed reviews

Use regular Claude for:
- General questions
- Quick tasks
- Mixed workflows

### Q: Can agents work together?

No, agents run independently. But you can run them sequentially:
1. test-writer creates tests
2. test-fixer fixes any issues
3. code-reviewer reviews the result

### Q: Are agent results always correct?

Agents are specialized and focused, so they're generally more reliable for their specific tasks. But always review their output.

---