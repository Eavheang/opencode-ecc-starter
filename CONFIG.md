# ECC + OpenCode: Setup Guide

This guide documents the **current as-built configuration** for the lean, token-optimized OpenCode setup at `D:\Self-study\opencode-ecc-starter`.

It is built around the [Ponytail](https://www.npmjs.com/package/@dietrichgebert/ponytail) behavior mode, permission-gated skills, and an empty `instructions` array — the opposite of the old ECC-heavy setup this repo originally shipped.

---

## Current Configuration Summary

| Layer | Location | What It Contains |
|-------|----------|------------------|
| Global config | `~/.config/opencode/opencode.jsonc` | Plugin, MCP, permissions — minimal, no instructions |
| Agents | `~/.config/opencode/agents/*.md` | 10 agent definitions, auto-loaded by OpenCode |
| Agent prompts | `~/.config/opencode/prompts/agents/*.txt` | System prompts for 6 subagents |
| Skills | `~/.config/opencode/skills/*/SKILL.md` | 7 skill guides, loaded on demand |
| Package manifest | `~/.config/opencode/package.json` | Ponytail plugin + @opencode-ai/plugin SDK |
| Project docs | `D:\Self-study\opencode-ecc-starter\` | `README.md` (quick start), this file (reference) |

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
  "default_agent": "build",
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

### Key points of this config

- **Plugin:** `@dietrichgebert/ponytail` (v4.8.3) — provides the lazy-senior-dev mode without injecting a large system prompt.
- **LSP:** enabled.
- **MCP:** two remote servers — Context7 (library docs) and Exa (web search). All MCP calls require confirmation (`mcp_*`: ask).
- **Instructions:** `[]` — no files injected into every session. The legacy `ECC_AGENTS.md` is on disk but ignored. Skills are loaded on demand instead.
- **Skills:** default-deny. Only the 7 explicitly allowed skills can be invoked.
- **No `agent` key in config** — agents are loaded from `~/.config/opencode/agents/*.md`.

### Token footprint

This config adds near-zero tokens to a session's system prompt:

- The config itself is parsed but its body isn't injected as instructions.
- The Ponytail plugin's behavior mode is implemented in code, not prompt text.
- Skills stay on disk until invoked.

By contrast, the old ECC setup injected `ECC_AGENTS.md` + `INSTRUCTIONS.md` + `LEARNED.md` + 9 `SKILL.md` files into every session — easily 15–25K tokens of system prompt.

---

## Agent Stack

Agents are defined in `~/.config/opencode/agents/<name>.md` and auto-loaded by OpenCode. Each file has YAML frontmatter (model, permissions) plus an optional system prompt body.

### Primary agents (cycle with `Tab`)

| Agent | Model | Temp | Edit | Bash | Purpose |
|-------|-------|------|------|------|---------|
| **build** | `opencode-go/minimax-m3` | 0.2 | allow | allow | Coordinator — delegates to subagents, never writes code itself |
| **plan** | `opencode-go/deepseek-v4-pro` | 0.1 | deny | deny | Pure thinker — asks clarifying questions, synthesizes plan, no file writes |
| **chat** | `opencode-go/deepseek-v4-flash` | 0.5 | deny | ask | Context-aware Q&A — delegates exploration, no edits |

### Subagents (invoke with `@name`)

| Agent | Model | Read | Write | Edit | Bash | Purpose |
|-------|-------|------|-------|------|------|---------|
| **@general** | `opencode-go/minimax-m3` | ✅ | ✅ | ✅ | ✅ | General implementation — writes code |
| **@explore** | `opencode-go/mimo-v2.5` | ✅ | ❌ | ❌ | ask | Fast codebase exploration |
| **@docs-lookup** | `opencode-go/deepseek-v4-flash` | ✅ | ❌ | ❌ | ❌ | Library docs via Context7 MCP |
| **@code-reviewer** | `opencode-go/deepseek-v4-flash` | ✅ | ❌ | ❌ | ✅ | Code quality review |
| **@tdd-guide** | `opencode-go/minimax-m3` | ✅ | ✅ | ✅ | ✅ | Test-driven development, 80%+ coverage |
| **@build-error-resolver** | `opencode-go/minimax-m3` | ✅ | ✅ | ✅ | ✅ | Build/type error resolution |
| **@security-reviewer** | `opencode-go/minimax-m3` | ✅ | ✅ | ✅ | ✅ | Security audits |

### Cost tiers

| Tier | Models | Use for |
|------|--------|---------|
| **Cheap** (~$0.14/1M) | `mimo-v2.5`, `deepseek-v4-flash` | Exploration, docs lookup, code review |
| **Promo** | `minimax-m3` | Implementation, TDD, security review |
| **Premium** ($1.74/1M) | `deepseek-v4-pro` | Synthesis, architecture, complex planning |
| **Minimax** | `minimax-m3` | Multi-role subagents (build, general, tdd-guide, build-error-resolver, security-reviewer) |

The coordinator in `build.md` enforces this: **"Use the cheapest model that can handle each subagent task."**

---

## Skills (permission-gated)

Skills live in `~/.config/opencode/skills/<name>/SKILL.md`. They are **denied by default** and only loaded when explicitly invoked with the `skill` tool.

| Skill | Status | Purpose |
|-------|--------|---------|
| `tdd-workflow` | allow | TDD methodology, 80%+ coverage |
| `security-review` | allow | Security checklist, threat models, secrets handling |
| `coding-standards` | allow | Naming, readability, immutability |
| `error-handling` | allow | Typed errors, retries, circuit breakers |
| `strategic-compact` | allow | Context compaction strategy |
| `verification-loop` | allow | Pre-commit quality checks |
| `continuous-learning-v2` | allow | Instinct-based learning via hooks |

### Adding a skill

1. Create `~/.config/opencode/skills/<name>/SKILL.md`
2. Add `"<name>": "allow"` to `permission.skill` in `opencode.jsonc`
3. Restart OpenCode

Skills in the directory that aren't in the allow-list are ignored — useful for keeping experimental skills on disk without paying for them every session.

---

## MCP Servers

| Server | URL | Purpose | Permission |
|--------|-----|---------|------------|
| **context7** | `https://mcp.context7.com/mcp` | Library/framework documentation lookup | `mcp_*`: ask |
| **exa** | `https://mcp.exa.ai/mcp` | Web search | `mcp_*`: ask |

Both are remote MCP servers. The `ask` permission means OpenCode will prompt before invoking any MCP tool — you see the tool call before it runs.

---

## Directory Layout

```
~/.config/opencode/
├── opencode.jsonc              # Main config — minimal, permission-gated
├── package.json                # @dietrichgebert/ponytail + @opencode-ai/plugin
├── package-lock.json
├── .gitignore
├── agents/                     # 10 agent definitions
│   ├── build.md                # Default orchestrator
│   ├── plan.md                 # Pure thinker
│   ├── chat.md                 # Q&A
│   ├── explore.md              # Fast exploration
│   ├── general.md              # Implementation
│   ├── tdd-guide.md
│   ├── build-error-resolver.md
│   ├── security-reviewer.md
│   ├── code-reviewer.md
│   └── docs-lookup.md
├── prompts/agents/             # System prompts for 6 subagents
│   ├── general.txt
│   ├── tdd-guide.txt
│   ├── build-error-resolver.txt
│   ├── security-reviewer.txt
│   ├── code-reviewer.txt
│   └── docs-lookup.txt
├── skills/                     # 7 skill guides
│   ├── coding-standards/
│   ├── continuous-learning-v2/
│   ├── error-handling/
│   ├── security-review/
│   ├── strategic-compact/
│   ├── tdd-workflow/
│   └── verification-loop/
├── learned.md                 # Patterns persisted by continuous-learning-v2
└── plugins/                   # (legacy ECC plugin code — empty on disk)
```

---

## Daily Workflow

1. **Start** in a project directory. OpenCode auto-loads `~/.config/opencode/agents/*.md` and the permission-gated skills. Nothing else is injected.
2. **Plan** — `Tab` → `plan`. The planner delegates research to `@explore` and docs to `@docs-lookup`, then synthesizes.
3. **Build** — `Tab` → `build`. The coordinator delegates implementation to `@general`, tests to `@tdd-guide`, review to `@code-reviewer`.
4. **Review** — `@code-reviewer review the changes I just made`.
5. **Fix** — `@build-error-resolver I'm getting this error: [paste]`.
6. **Secure** — `@security-reviewer check the auth/API changes`.
7. **Test** — `@tdd-guide write tests for this feature`.
8. **Compact** — `Shift+Tab` or `/compact` when context gets long.

---

## Verification

To verify the setup:

```bash
# Check installed packages
cd ~/.config/opencode && npm list

# Start OpenCode
opencode
```

Inside OpenCode:

```
/models                    # confirm all models are available
Tab                        # cycle build → plan → chat
@explore                   # test subagent autocomplete
@docs-lookup how to use zod    # test MCP-backed docs lookup
```

For project health checks:

```bash
npm run build
npm run lint
npx tsc --noEmit
```

---

## Token-Saving Tips

1. **Don't add to `instructions`** — every file in the array is injected into every session. Use skills instead.
2. **Skills default to deny** — they only load when invoked. Don't allow a skill you don't use.
3. **Don't read files in Build mode** — delegate to `@explore` (isolated context).
4. **Use `chat` for questions** — cheapest model, no tools executed.
5. **Switch to `plan` for complex tasks** — Pro is more expensive per token but the no-edit constraint prevents wasted work.
6. **Use `/compact`** (`Shift+Tab`) when context gets long.

---

## Migration Notes

This config was migrated from the old ECC setup. The original `opencode.jsonc` had:

- `plugin: ["ecc-universal"]` → now `["@dietrichgebert/ponytail"]`
- `instructions: [12 files]` → now `[]`
- Many agent definitions inline in the config → now externalized to `agents/*.md`
- `permission.skill` didn't exist → now the skill-loading mechanism

The `plugins/` and `instructions/` directories are empty on disk — leftover from the ECC migration, can be deleted for a clean tree.

---

## Known Gaps

1. **Legacy `plugins/` and `instructions/` directories** — empty on disk, leftover from the ECC migration. Safe to delete.
2. **No project-level `AGENTS.md`** — for a specific project, create `AGENTS.md` in the project root to document its stack, conventions, and key files. To inject it, add it to the `instructions` array in a project-level `opencode.jsonc`.

---

## Quick Start on a New Machine

```bash
# 1. Install OpenCode
npm install -g opencode-ai

# 2. Restore the config (replace path with your backup or repo)
cp -r ~/.config/opencode ~/.config/opencode-backup
# Then copy the config from this repo or your backup

# 3. Install dependencies
cd ~/.config/opencode
npm install

# 4. Start OpenCode
opencode
# Use /connect to set the API key
# Use /models to verify models
```

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| Skill won't load | Verify the skill is in `permission.skill` with `"allow"` |
| Agent model not found | Run `/models`. Go models use `opencode-go/` prefix |
| Tab doesn't cycle agents | Ensure `build.md`, `plan.md`, `chat.md` exist in `~/.config/opencode/agents/` |
| MCP not responding | Check the URL in `opencode.jsonc` and your network access |
| Plugin not loading | `cd ~/.config/opencode && npm install` to refresh `node_modules` |
| Skill loads but produces nothing | Check the `SKILL.md` path exists and is readable |
