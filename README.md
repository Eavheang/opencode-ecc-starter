# OpenCode ECC Starter

A ready-to-use [OpenCode](https://opencode.ai) configuration with [ECC](https://github.com/nicobailon/EverythingClaudeCode) (Everything Claude Code) integration. Includes custom agents, skills, hooks, and a cost-optimized model setup.

## Quick Start

```bash
# 1. Install OpenCode
npm install -g opencode-ai

# 2. Copy this config to your global OpenCode config
cp -r ~/.config/opencode ~/.config/opencode-backup   # backup first
cp -r <this-repo>/* ~/.config/opencode/

# 3. Start OpenCode in your project
cd /path/to/your/project
opencode
```

---

## Agent Stack

| Agent | Trigger | Model | Cost | Purpose |
|-------|---------|-------|------|---------|
| **Build** | `Tab` | Qwen3.7 Plus | $0.40/1M | Writing code, implementing features |
| **Plan** | `Tab` | DeepSeek V4 Pro | $1.74/1M | Planning, analysis, strategy |
| **Chat** | `Tab` | DeepSeek V4 Flash | $0.14/1M | Q&A, explanations, no changes |
| **@explore** | `@explore` | MiMo-V2.5 | $0.14/1M | Fast codebase exploration |
| **@docs-lookup** | `@docs-lookup` | DeepSeek V4 Flash | $0.14/1M | Library docs, API references |
| **@architect** | `@architect` | DeepSeek V4 Pro | $1.74/1M | System design, architecture |
| **@code-reviewer** | `@code-reviewer` | DeepSeek V4 Flash | $0.14/1M | Code quality review |
| **@security-reviewer** | `@security-reviewer` | Qwen3.7 Plus | $0.40/1M | Security audits |
| **@tdd-guide** | `@tdd-guide` | Qwen3.7 Plus | $0.40/1M | Test-driven development |
| **@build-error-resolver** | `@build-error-resolver` | Qwen3.7 Plus | $0.40/1M | Build/type error fixes |
| **@e2e-runner** | `@e2e-runner` | Qwen3.7 Plus | $0.40/1M | Playwright E2E tests |
| **@refactor-cleaner** | `@refactor-cleaner` | MiMo-V2.5 | $0.14/1M | Dead code cleanup |
| **@doc-updater** | `@doc-updater` | MiMo-V2.5 | $0.14/1M | Documentation updates |
| **@database-reviewer** | `@database-reviewer` | Qwen3.7 Plus | $0.40/1M | SQL/schema review |

**Cost tiers:**
- **Flash** ($0.14/1M) — cheap exploration, read-only tasks
- **MiMo-V2.5** ($0.14/1M) — same cost, better quality than Flash
- **Qwen3.7 Plus** ($0.40/1M) — implementation, code generation
- **DeepSeek V4 Pro** ($1.74/1M) — synthesis, architecture decisions

---

## Workflow

### 1. Start a session

Open opencode in your project directory. The system loads automatically:

- `AGENTS.md` — project context
- `LEARNED.md` — accumulated patterns from past sessions
- `ECC_AGENTS.md` — ECC principles
- All skills in the `instructions` array

### 2. Plan

Switch to Plan mode (`Tab`) and describe the task:

```
I need to add a new row type to the layout builder that supports 3-column layouts.
- A new constant for the 3-column row
- Update the builder to handle 3-column insertions
- Update the drop zone logic to accept 3-column rows
```

Plan delegates research to `@explore` (cheap) and architecture to `@architect` (when needed), then synthesizes a strategy.

### 3. Review the plan

Before implementing, review:
- Did the plan ask clarifying questions?
- Were sub-agents used for research?
- Are there missing steps or risks?

### 4. Implement

Switch back to Build (`Tab`) and tell it to proceed:

```
Good, go ahead with the plan.
```

### 5. Review code

```
@code-reviewer review the changes I just made
```

Address CRITICAL and HIGH issues immediately.

### 6. Fix errors

```
@build-error-resolver I'm getting this error:
[paste error]
```

### 7. Security check (if needed)

```
@security-reviewer check the changes for security issues
```

### 8. Write tests (if needed)

```
@tdd-guide write tests for the new 3-column row feature
```

### 9. End of session

```
/learn
```

This captures reusable patterns and appends them to `LEARNED.md` for next time.

---

## Quick Reference

| Task | Command |
|------|---------|
| "What does this file do?" | `Tab` → Chat, ask |
| "Find all API routes" | `@explore find all API route files` |
| "How do I use this library?" | `@docs-lookup how to use dnd-kit sortable` |
| "Plan this feature" | `Tab` → Plan, describe task |
| "Design the database" | `@architect design schema for user auth` |
| "Write the code" | `Tab` → Build, describe task |
| "Review my code" | `@code-reviewer review changes` |
| "Security check" | `@security-reviewer check for vulnerabilities` |
| "Fix this error" | `@build-error-resolver [error]` |
| "Clean up this code" | `@refactor-cleaner simplify this component` |
| "Write tests" | `@tdd-guide write tests for this feature` |
| "Update docs" | `@doc-updater update README with new feature` |
| "Learn from this session" | `/learn` |

---

## Token Saving Tips

1. **Don't read files in Build mode** — use `@explore` instead (isolated context)
2. **Don't plan in Build mode** — `Tab` to Plan first
3. **Use `/compact`** if context gets long (`Shift+Tab`)
4. **Remove unused skills** from `opencode.jsonc` instructions
5. **Sub-agents are cheap** — they run in isolated child contexts
6. **Chat mode for questions** — cheapest model, no tools executed

---

## Loaded Skills

| Skill | Purpose |
|-------|---------|
| `coding-standards` | Naming, readability, immutability rules |
| `frontend-patterns` | React, Next.js, state management |
| `backend-patterns` | API design, Node.js, Express |
| `security-review` | Auth, input validation, secrets |
| `tdd-workflow` | Test-driven development, 80%+ coverage |
| `e2e-testing` | Playwright testing patterns |
| `verification-loop` | Quality checks before commits |
| `api-design` | REST API design patterns |
| `strategic-compact` | Context compaction strategy |

Skills load automatically. Knowing what they cover helps you understand why the AI makes certain suggestions.

---

## Project Structure

```
~/.config/opencode/
├── opencode.jsonc          # Main config (agents, models, instructions)
├── ECC_AGENTS.md           # ECC core principles
├── plugins/
│   ├── index.ts            # Plugin entry point
│   └── ecc-hooks.ts        # Hook implementations
├── instructions/
│   ├── INSTRUCTIONS.md     # Core rules
│   └── LEARNED.md          # Accumulated patterns
├── prompts/agents/         # System prompts for subagents
└── skills/                 # Active skill guides
```

---

## Customization

### Switch models

Edit `opencode.jsonc` to change agent models:

```jsonc
{
  "agent": {
    "build": {
      "model": "opencode-go/qwen3.7-plus"  // change to any Go model
    }
  }
}
```

### Add/remove skills

Remove skills from the `instructions` array in `opencode.jsonc` to reduce context:

```jsonc
{
  "instructions": [
    "ECC_AGENTS.md",
    "instructions/INSTRUCTIONS.md",
    "skills/frontend-patterns/SKILL.md"   // remove unused skills
  ]
}
```

### Set hook profile

```powershell
# Windows
$env:ECC_HOOK_PROFILE = "strict"

# macOS/Linux
export ECC_HOOK_PROFILE=strict
```

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| `"plugin"` not recognized | Update OpenCode to v1.6+ |
| Skills not loading | Verify the `instructions` path points to a valid `SKILL.md` |
| ECC hooks not firing | Check `ECC_HOOK_PROFILE` is set |
| Model not found | Run `/models` to verify. Go models use `opencode-go/` |
| Tab doesn't cycle agents | Ensure at least 2 primary agents have `"mode": "primary"` |
