# ECC + OpenCode: Setup Guide

This guide documents the **current as-built configuration** for using **Everything Claude Code (ECC)** with **OpenCode**.

It covers the global OpenCode config, the agents and skills currently loaded, the ECC plugin hooks, and how to use this setup in a project.

---

## Current Configuration Summary

| Layer | Location | What It Contains |
|-------|----------|------------------|
| Global config | `~/.config/opencode/opencode.jsonc` | Agents, models, instructions, plugin registration |
| ECC agent rules | `~/.config/opencode/ECC_AGENTS.md` | Core ECC principles loaded every session |
| Core instructions | `~/.config/opencode/instructions/INSTRUCTIONS.md` | Security, coding style, TDD, workflow rules |
| Learned patterns | `~/.config/opencode/instructions/LEARNED.md` | Accumulated session patterns (`/learn`) |
| Active skills | `~/.config/opencode/skills/*` | Domain-specific skill guides |
| Agent prompts | `~/.config/opencode/prompts/agents/*.txt` | System prompts for subagents |
| ECC plugin | `~/.config/opencode/plugins/` | OpenCode plugin entry point and hooks |
| Project docs | `D:\Self-study\opencode-setup\` | `INSTRUCTION.md` (daily workflow), this file |

---

## Global Config (`~/.config/opencode/opencode.jsonc`)

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": ["ecc-universal"],
  "instructions": [
    "ECC_AGENTS.md",
    "instructions/INSTRUCTIONS.md",
    "instructions/LEARNED.md",
    "skills/tdd-workflow/SKILL.md",
    "skills/security-review/SKILL.md",
    "skills/coding-standards/SKILL.md",
    "skills/frontend-patterns/SKILL.md",
    "skills/backend-patterns/SKILL.md",
    "skills/e2e-testing/SKILL.md",
    "skills/verification-loop/SKILL.md",
    "skills/api-design/SKILL.md",
    "skills/strategic-compact/SKILL.md"
  ],
  "agent": {
    // PRIMARY AGENTS (switch with Tab)
    "build": {
      "description": "Primary coding agent for development work",
      "mode": "primary",
      "model": "opencode-go/qwen3.7-plus",
      "temperature": 0.2
    },
    "plan": {
      "description": "Expert planning specialist for complex features and refactoring",
      "mode": "primary",
      "model": "opencode-go/deepseek-v4-pro",
      "temperature": 0.1,
      "permission": { "edit": "ask", "bash": "ask" }
    },
    "chat": {
      "description": "Ask questions, read files, get explanations — no changes",
      "mode": "primary",
      "model": "opencode-go/deepseek-v4-flash",
      "temperature": 0.5,
      "color": "info",
      "permission": { "edit": "deny", "bash": "deny", "task": "deny" }
    },

    // SUBAGENTS (invoke with @name)
    "explore": {
      "description": "Fast agent specialized for exploring codebases",
      "mode": "subagent",
      "model": "opencode-go/deepseek-v4-flash",
      "permission": { "edit": "deny", "bash": "ask" }
    },
    "architect": {
      "description": "Software architecture specialist for system design and scalability",
      "mode": "subagent",
      "prompt": "{file:prompts/agents/architect.txt}",
      "model": "opencode-go/deepseek-v4-pro",
      "permission": { "edit": "deny", "bash": "deny" }
    },
    "code-reviewer": {
      "description": "Expert code review for quality, security, and maintainability",
      "mode": "subagent",
      "prompt": "{file:prompts/agents/code-reviewer.txt}",
      "tools": { "read": true, "bash": true, "write": false, "edit": false }
    },
    "security-reviewer": {
      "description": "Security vulnerability detection and remediation",
      "mode": "subagent",
      "prompt": "{file:prompts/agents/security-reviewer.txt}",
      "tools": { "read": true, "bash": true, "write": true, "edit": true }
    },
    "tdd-guide": {
      "description": "Test-Driven Development specialist (80%+ coverage)",
      "mode": "subagent",
      "prompt": "{file:prompts/agents/tdd-guide.txt}",
      "tools": { "read": true, "write": true, "edit": true, "bash": true }
    },
    "build-error-resolver": {
      "description": "Build and TypeScript error resolution specialist",
      "mode": "subagent",
      "prompt": "{file:prompts/agents/build-error-resolver.txt}",
      "tools": { "read": true, "write": true, "edit": true, "bash": true }
    },
    "e2e-runner": {
      "description": "End-to-end testing specialist using Playwright",
      "mode": "subagent",
      "prompt": "{file:prompts/agents/e2e-runner.txt}",
      "tools": { "read": true, "write": true, "edit": true, "bash": true }
    },
    "doc-updater": {
      "description": "Documentation and codemap specialist",
      "mode": "subagent",
      "prompt": "{file:prompts/agents/doc-updater.txt}",
      "tools": { "read": true, "write": true, "edit": true, "bash": true }
    },
    "refactor-cleaner": {
      "description": "Dead code cleanup and consolidation specialist",
      "mode": "subagent",
      "prompt": "{file:prompts/agents/refactor-cleaner.txt}",
      "tools": { "read": true, "write": true, "edit": true, "bash": true }
    },
    "docs-lookup": {
      "description": "Documentation specialist using Context7 MCP",
      "mode": "subagent",
      "prompt": "{file:prompts/agents/docs-lookup.txt}",
      "model": "opencode-go/deepseek-v4-flash",
      "permission": { "edit": "deny", "bash": "deny" }
    },
    "database-reviewer": {
      "description": "Database specialist for query optimization and schema design",
      "mode": "subagent",
      "prompt": "{file:prompts/agents/database-reviewer.txt}",
      "tools": { "read": true, "write": true, "edit": true, "bash": true }
    }
  },
  "permission": {
    "mcp_*": "ask"
  }
}
```

### Key points of this config

- **3 primary agents** cycled with `Tab`: `build`, `plan`, `chat`.
- **11 subagents** invoked with `@name`.
- `chat` is read-only: it cannot edit files, run commands, or spawn tasks.
- `plan` must ask before editing or running bash commands.
- Subagents that should not touch code (`code-reviewer`, `architect`, `docs-lookup`) have write/edit denied.
- `security-reviewer`, `build-error-resolver`, `tdd-guide`, `e2e-runner`, `doc-updater`, `refactor-cleaner`, and `database-reviewer` have full tooling when explicitly invoked.

---

## Agent Stack

| Agent | Mode | Model | Input Cost (approx.) | Purpose |
|-------|------|-------|---------------------|---------|
| **build** | Tab | `opencode-go/qwen3.7-plus` | $0.40/1M | Main coding work |
| **plan** | Tab | `opencode-go/deepseek-v4-pro` | $1.74 / 1M | Complex planning, strategy |
| **chat** | Tab | `opencode-go/deepseek-v4-flash` | $0.14 / 1M | Q&A, explanations, no changes |
| **@explore** | Subagent | `opencode-go/mimo-v2.5` | $0.14 / 1M | Cheap codebase exploration |
| **@docs-lookup** | Subagent | `opencode-go/deepseek-v4-flash` | $0.14 / 1M | Library docs lookup |
| **@architect** | Subagent | `opencode-go/deepseek-v4-pro` | $1.74 / 1M | System design only when needed |
| **@code-reviewer** | Subagent | `opencode-go/deepseek-v4-flash` | $0.14 / 1M | Review code quality |
| **@security-reviewer** | Subagent | `opencode-go/qwen3.7-plus` | $0.40 / 1M | Security audits |
| **@build-error-resolver** | Subagent | `opencode-go/qwen3.7-plus` | $0.40 / 1M | Fix build/type errors |
| **@tdd-guide** | Subagent | `opencode-go/qwen3.7-plus` | $0.40 / 1M | Write tests, TDD workflow |
| **@e2e-runner** | Subagent | `opencode-go/qwen3.7-plus` | $0.40 / 1M | Playwright E2E tests |
| **@refactor-cleaner** | Subagent | `opencode-go/mimo-v2.5` | $0.14 / 1M | Dead code cleanup |
| **@doc-updater** | Subagent | `opencode-go/mimo-v2.5` | $0.14 / 1M | Update documentation |
| **@database-reviewer** | Subagent | `opencode-go/qwen3.7-plus` | $0.40 / 1M | SQL/schema review |

> **Cost-efficiency rule:** research and exploration go to MiMo-V2.5 (`@explore`, `@doc-updater`, `@refactor-cleaner`), code review goes to Flash (`@code-reviewer`), synthesis and architecture go to Pro (`plan`, `@architect`), and implementation goes to Qwen 3.7 Plus (`build`, `@tdd-guide`, `@build-error-resolver`, `@security-reviewer`, `@e2e-runner`, `@database-reviewer`).

---

## Loaded Instructions / Skills

These files are injected into every session via the `instructions` array:

| Path | Purpose |
|------|---------|
| `ECC_AGENTS.md` | ECC core principles, agent orchestration, security checks |
| `instructions/INSTRUCTIONS.md` | Response style, coding standards, TDD, git workflow |
| `instructions/LEARNED.md` | Personal patterns accumulated via `/learn` |
| `skills/coding-standards/SKILL.md` | Naming, readability, immutability rules |
| `skills/frontend-patterns/SKILL.md` | React/Next.js/state management guidance |
| `skills/backend-patterns/SKILL.md` | API/server-side patterns |
| `skills/security-review/SKILL.md` | Security checklist and patterns |
| `skills/tdd-workflow/SKILL.md` | TDD methodology, 80%+ coverage |
| `skills/e2e-testing/SKILL.md` | Playwright testing patterns |
| `skills/verification-loop/SKILL.md` | Pre-commit quality checks |
| `skills/api-design/SKILL.md` | REST API design patterns |
| `skills/strategic-compact/SKILL.md` | Context compaction strategy |

If a skill is not relevant to the current project, remove it from the `instructions` array to save context tokens.

---

## ECC Plugin & Hooks

The ECC plugin is registered in `opencode.jsonc`:

```jsonc
"plugin": ["ecc-universal"]
```

The plugin code lives in `~/.config/opencode/plugins/`:

```
plugins/
├── index.ts              # Re-exports the ECC hooks plugin
├── ecc-hooks.ts          # Hook implementations
└── lib/
    └── changed-files-store.ts
```

### Hook behavior

Hooks fire on OpenCode events. They are controlled by the `ECC_HOOK_PROFILE` environment variable:

| Profile | Behavior |
|---------|----------|
| `minimal` | Console.log audit only (silent) |
| `standard` | Console.log audit + warnings (default) |
| `strict` | Auto-format, typecheck, security reminders |

Set in PowerShell:

```powershell
$env:ECC_HOOK_PROFILE = "strict"
```

Or persist it:

```powershell
[Environment]::SetEnvironmentVariable("ECC_HOOK_PROFILE", "strict", "User")
```

Disable specific hooks:

```powershell
$env:ECC_DISABLED_HOOKS = "post:edit:format,pre:bash:git-push-reminder"
```

### What the hooks do

- **`file.edited`**: track edited files; in `strict` mode run Prettier, warn on `console.log`.
- **`tool.execute.after`**: track writes/edits; in `strict` mode run `npx tsc --noEmit`.
- **`tool.execute.before`**: remind before `git push`; warn on unnecessary doc files; flag long-running commands.
- **`session.created`**: log session start and profile.
- **`session.idle`**: audit edited files for `console.log` statements.
- **`session.deleted`**: cleanup tracked state.
- **`shell.env`**: inject `ECC_VERSION`, `PROJECT_ROOT`, detected package manager, and primary language.
- **`permission.ask`**: auto-approve read/search tools, formatters, and test commands.

---

## Learning System

The `LEARNED.md` file at `~/.config/opencode/instructions/LEARNED.md` is loaded into every session.

### How it works

```
Session starts → AI reads LEARNED.md → applies past patterns
       ↓
User says /learn → AI appends new patterns
       ↓
Next session → AI reads updated LEARNED.md
```

### Capturing patterns

At the end of a session, run:

```
/learn
```

The AI will scan the session for reusable patterns and append them to `LEARNED.md` as Instinct entries.

Record things like:
- Error fixes that were not obvious
- Codebase quirks
- Workflow improvements
- Project-specific conventions

Do **not** record typos, one-time issues, or trivial fixes.

---

## Project-Level Setup

For a specific project, create these files in the project root:

- `AGENTS.md` — project-specific stack, conventions, key files
- `opencode.json` or `opencode.jsonc` — project-level overrides

Example minimal project config:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "instructions": [
    "AGENTS.md",
    "INSTRUCTION.md"
  ]
}
```

> **Current status for `D:\Self-study\opencode-setup`:**
> - `INSTRUCTION.md` exists (daily workflow guide).
> - `AGENTS.md` and `opencode.json` have **not** been created yet.

---

## Daily Workflow

For the day-to-day workflow, see `INSTRUCTION.md` in this directory.

Quick reference:

1. **Start** in a project directory. OpenCode loads `ECC_AGENTS.md`, `INSTRUCTIONS.md`, `LEARNED.md`, and the configured skills.
2. **Plan** — press `Tab` → `plan` for complex tasks. The planner delegates research to `@explore`.
3. **Build** — press `Tab` → `build` to implement.
4. **Review** — invoke `@code-reviewer` after changes.
5. **Fix** — invoke `@build-error-resolver` for build or type errors.
6. **Secure** — invoke `@security-reviewer` for auth/API/input changes.
7. **Test** — invoke `@tdd-guide` for new features or bug fixes.
8. **Learn** — run `/learn` before ending the session.

---

## Directory Layout

```
~/.config/opencode/
├── opencode.jsonc              # Main config
├── ECC_AGENTS.md               # ECC agent definitions
├── package.json                # Plugin dependency (@opencode-ai/plugin)
├── plugins/
│   ├── index.ts                # Plugin entry point
│   ├── ecc-hooks.ts            # Hook implementations
│   └── lib/
│       └── changed-files-store.ts
├── instructions/
│   ├── INSTRUCTIONS.md         # Core ECC rules
│   └── LEARNED.md              # Accumulated learnings
├── prompts/
│   └── agents/                 # System prompts for subagents
│       ├── architect.txt
│       ├── code-reviewer.txt
│       ├── security-reviewer.txt
│       ├── tdd-guide.txt
│       └── ... (25 total)
└── skills/                     # Active skills
    ├── coding-standards/
    ├── frontend-patterns/
    ├── backend-patterns/
    ├── security-review/
    ├── tdd-workflow/
    ├── e2e-testing/
    ├── verification-loop/
    ├── api-design/
    └── strategic-compact/

~/.claude/
└── skills/
    └── ecc/                    # Full ECC skill catalog (source)
        ├── coding-standards/
        ├── frontend-patterns/
        └── ... (80+ skills)

D:\Self-study\opencode-setup/
├── SETUP.md                    # This file
├── INSTRUCTION.md              # Daily workflow guide
└── AGENTS.md                   # (not created yet)
```

---

## Verification

To verify the setup:

```bash
# Check plugin is loaded
opencode --list-plugins

# Start OpenCode
opencode
```

Inside OpenCode:

```
/models                    # confirm models are available
Tab                        # cycle build → plan → chat
@explore                   # test subagent autocomplete
@code-reviewer             # test subagent invocation
```

For project health checks:

```bash
npm run build
npm run lint
npx tsc --noEmit
```

---

## Notes & Known Gaps

1. **Subagent model inheritance:** all subagents now have explicit `model` set. No more implicit inheritance from the Build agent.
2. **Skill pruning:** 12 instructions are loaded every session. For a pure frontend project you can drop `backend-patterns`, `api-design`, and `database-reviewer` from `opencode.jsonc` to reduce context.
3. **Project-level files:** `AGENTS.md` and `opencode.json` for `D:\Self-study\opencode-setup` are not created yet.
4. **Plugin import:** `plugins/ecc-hooks.ts` imports from `../tools/changed-files.js`, but the `tools/` directory is not present in the current layout. If hook errors appear, verify this path or recreate the `changed-files` tool.

---

## Quick Start on a New Machine

```bash
# 1. Install OpenCode
npm install -g opencode-ai

# 2. Install ECC
npm install -g ecc-universal

# 3. Copy global config
cp -r ~/.config/opencode ~/.config/opencode-backup
# Then restore the backed-up config files

# 4. Copy project files to the new project
cp SETUP.md INSTRUCTION.md AGENTS.md <new-project>/

# 5. Run OpenCode
opencode
# Use /connect to set the API key
# Use /models to verify models
```

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| `"plugin"` not recognized | Update OpenCode to v1.6+ |
| Skills not loading | Verify the `instructions` path points to a valid `SKILL.md` |
| ECC hooks not firing | Check `ECC_HOOK_PROFILE` is set and `ecc-hooks.ts` compiles |
| ECC agents not found | Ensure `ECC_AGENTS.md` is in the `instructions` array |
| `ecc` command not found | Reinstall: `npm install -g ecc-universal` |
| Model not found | Check `/models`. Go models use `opencode-go/`, Zen models use `opencode/` |
| Build agent uses wrong model | Verify the agent `"model"` matches a model from `/models` |
| `LEARNED.md` not loading | Verify its path in the `instructions` array |
| Tab doesn't cycle agents | Ensure at least 2 primary agents are configured with `"mode": "primary"` |
| Broken `changed-files` import in hooks | Verify `~/.config/opencode/tools/changed-files.js` exists |
