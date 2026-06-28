# OpenCode ECC Starter

A lean [OpenCode](https://opencode.ai) configuration optimized for **token efficiency**. Uses the [Ponytail](https://www.npmjs.com/package/@dietrichgebert/ponytail) behavior mode, permission-gated skills, and minimal default context.

## Philosophy

This is a **tokenmaxxer** setup — minimize context overhead, load only what's needed:

- **Empty `instructions` array** by default. No massive system prompt files injected every session.
- **Permission-gated skills** — only load `SKILL.md` content when the skill is actually invoked.
- **No hardcoded agent prompts in config** — agent definitions live in `~/.config/opencode/agents/*.md` (auto-loaded by OpenCode).
- **MCP only when needed** — Context7 for library docs, Exa for web search.
- **Ponytail mode** for lazy-senior-dev guardrails without per-file system prompt bloat.

---

## Quick Start

```bash
# 1. Install OpenCode
npm install -g opencode-ai

# 2. Install dependencies
cd ~/.config/opencode
npm install

# 3. Start OpenCode in your project
cd /path/to/your/project
opencode
```

---

## Global Config (`~/.config/opencode/opencode.jsonc`)

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "lsp": true,
  "mcp": {
    "context7": {
      "type": "remote",
      "url": "https://mcp.context7.com/mcp",
      "enabled": true
    },
    "exa": {
      "type": "remote",
      "url": "https://mcp.exa.ai/mcp",
      "enabled": true
    }
  },
  "plugin": ["@dietrichgebert/ponytail"],
  "default_agent": "harness",
  "instructions": [],
  "permission": {
    "mcp_*": "ask",
    "skill": {
      "*": "deny",
      "tdd-workflow": "allow",
      "security-review": "allow",
      "coding-standards": "allow",
      "error-handling": "allow",
      "strategic-compact": "allow",
      "verification-loop": "allow",
      "continuous-learning-v2": "allow"
    }
  }
}
```

### Key points

- **Plugin:** `@dietrichgebert/ponytail` (v4.8.3) — adds lazy-senior-dev mode without instruction-file overhead.
- **Default agent:** `harness` — orchestrator that auto-plans, delegates in parallel, and verifies.
- **LSP:** enabled.
- **MCP:** Context7 (docs) and Exa (web search) — both `ask` per call.
- **Instructions:** empty. No `ECC_AGENTS.md`, no `INSTRUCTIONS.md`, no skills auto-loaded.
- **Skills:** denied by default. Only the 7 listed skills are allowed when explicitly invoked.

---

## Agent Stack

Agents are defined in `~/.config/opencode/agents/*.md` and auto-loaded by OpenCode.

### Primary agents (cycle with `Tab`)

| Agent | Model | Cost | Purpose |
|-------|-------|------|---------|
| **harness** | `opencode-go/kimi-k2.6` | — | **Default orchestrator** — auto-plans, delegates in parallel, verifies independently |
| **build** | `opencode-go/qwen3.7-plus` | $0.40/1M | Coordinator — delegates implementation to subagents, never writes code directly |
| **plan** | `opencode-go/deepseek-v4-pro` | $1.74/1M | Pure thinker — delegates research, synthesizes plan. Edit/deny, no file writes |
| **chat** | `opencode-go/deepseek-v4-flash` | $0.14/1M | Context-aware Q&A. Edit denied |

### Subagents (invoke with `@name`)

| Agent | Model | Cost | Tools | Purpose |
|-------|-------|------|-------|---------|
| **@general** | `opencode-go/minimax-m3` | — | read/write/edit/bash | General implementation — writes code |
| **@explore** | `opencode-go/mimo-v2.5` | $0.14/1M | read, bash (ask) | Fast codebase exploration |
| **@docs-lookup** | `opencode-go/deepseek-v4-flash` | $0.14/1M | read only | Library docs via Context7 MCP |
| **@code-reviewer** | `opencode-go/deepseek-v4-flash` | $0.14/1M | read, bash | Code quality review (no edits) |
| **@tdd-guide** | `opencode-go/qwen3.7-plus` | $0.40/1M | full | Test-driven development, 80%+ coverage |
| **@build-error-resolver** | `opencode-go/minimax-m3` | — | full | Build/type error fixes |
| **@security-reviewer** | `opencode-go/minimax-m3` | — | full | Security audits |

### Cost tiers

- **Flash / MiMo** (~$0.14/1M) — exploration, read-only tasks, reviews
- **Qwen 3.7 Plus** ($0.40/1M) — implementation, TDD, security
- **DeepSeek V4 Pro** ($1.74/1M) — synthesis, architecture, complex planning

---

## Skills (permission-gated)

Skills live in `~/.config/opencode/skills/*/SKILL.md` and are only loaded when invoked. Default permission: **deny**.

| Skill | Permission | Purpose |
|-------|------------|---------|
| `tdd-workflow` | allow | TDD methodology, 80%+ coverage |
| `security-review` | allow | Security checklist and patterns |
| `coding-standards` | allow | Naming, readability, immutability |
| `error-handling` | allow | Typed errors, retries, circuit breakers |
| `strategic-compact` | allow | Context compaction strategy |
| `verification-loop` | allow | Pre-commit quality checks |
| `continuous-learning-v2` | allow | Instinct-based learning via hooks |

To add a new skill:

1. Create `~/.config/opencode/skills/<name>/SKILL.md`
2. Add `"<name>": "allow"` to the `permission.skill` object in `opencode.jsonc`

---

## Workflow

### 1. Plan a task

```
Tab → plan
"I need to add a 3-column row type to the layout builder."
```

Plan delegates research to `@explore` and docs lookup to `@docs-lookup`, then synthesizes a plan. It cannot edit files.

### 2. Implement

```
Tab → harness
"Go ahead with the plan."
```

Harness orchestrates — it auto-plans, delegates to `@general` for code, `@tdd-guide` for tests, `@code-reviewer` for review, and verifies results. It does not write code itself. For simpler tasks you can also use `Tab → build`.

### 3. Review and fix

```
@code-reviewer review the changes
@build-error-resolver I'm getting this error: [paste]
@security-reviewer check the auth changes
```

### 4. Look up docs

```
@docs-lookup how to use dnd-kit sortable
```

### 5. Web search (Exa MCP)

Available to any agent — ask before using.

---

## Directory Layout

```
~/.config/opencode/
├── opencode.jsonc           # Main config — empty instructions, permission-gated skills
├── package.json             # @dietrichgebert/ponytail + @opencode-ai/plugin
├── agents/                  # Agent definitions (auto-loaded)
│   ├── harness.md           # Default orchestrator
│   ├── build.md             # Coordinator
│   ├── plan.md
│   ├── chat.md
│   ├── explore.md
│   ├── general.md
│   ├── tdd-guide.md
│   ├── build-error-resolver.md
│   ├── security-reviewer.md
│   ├── code-reviewer.md
│   └── docs-lookup.md
├── prompts/agents/          # System prompts for subagents that need them
│   ├── general.txt
│   ├── tdd-guide.txt
│   ├── build-error-resolver.txt
│   ├── security-reviewer.txt
│   ├── code-reviewer.txt
│   └── docs-lookup.txt
├── skills/                  # Skill guides (permission-gated)
│   ├── coding-standards/
│   ├── continuous-learning-v2/
│   ├── error-handling/
│   ├── security-review/
│   ├── strategic-compact/
│   ├── tdd-workflow/
│   └── verification-loop/
├── learned.md               # Patterns persisted by continuous-learning-v2
└── plugins/                 # (legacy ECC plugin code — no longer loaded)
```

> **Note:** `ECC_AGENTS.md` and `instructions/INSTRUCTIONS.md` are still on disk from the old setup but are **not loaded** because `instructions` is empty. They are safe to delete.

---

## Token-Saving Tips

1. **Skills default to deny** — they only load when you invoke them. Don't add a skill unless you need it.
2. **No instructions in config** — adding files to `instructions` injects them every session. Use skills instead.
3. **Use `@explore` for file reads** in Harness/Build mode — subagents have isolated contexts.
4. **Use `chat` for questions** — cheapest model, no tools executed.
5. **Switch to `plan` for complex tasks** — Pro is more expensive per token but the no-edit constraint prevents wasted work.
6. **Use `/compact`** (`Shift+Tab`) when context gets long.

---

## Customization

### Add/remove skills

Edit `opencode.jsonc` `permission.skill` — set `"<name>": "allow"` or `"deny"`.

### Change agent models

Edit the corresponding file in `~/.config/opencode/agents/<name>.md`:

```yaml
---
model: opencode-go/qwen3.7-plus   # change to any model from /models
---
```

### Switch default agent

Edit `opencode.jsonc` and change `"default_agent"` to `"build"`, `"plan"`, or `"chat"`.

### Switch plugin

Edit `opencode.jsonc` `plugin` array. Ponytail can be removed or replaced — no other config changes required.

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| Skill won't load | Check `permission.skill.<name>` is `"allow"` |
| Agent model not found | Run `/models` in OpenCode. Go models use `opencode-go/` prefix |
| Tab doesn't cycle agents | Ensure `harness.md`, `build.md`, `plan.md`, `chat.md` exist in `~/.config/opencode/agents/` |
| MCP not responding | Check the URL in `opencode.jsonc` and your network access |
| Plugin not loading | `cd ~/.config/opencode && npm install` to refresh `node_modules` |
