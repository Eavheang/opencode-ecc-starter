# Daily Workflow Guide — ECC + OpenCode

How to use this setup for daily coding tasks, from start to finish.

---

## Your Agent Stack

| Agent | Mode | Model | When to Use |
|-------|------|-------|-------------|
| **Build** | Tab | Qwen3.7 Plus | Writing code, implementing features, fixing bugs |
| **Plan** | Tab | DeepSeek V4 Pro | Planning features, analyzing code, creating strategies |
| **Chat** | Tab | DeepSeek V4 Flash | Quick questions, reading files, explanations |
| **Explore** | `@explore` | MiMo-V2.5 | Searching codebase, finding files, code patterns |
| **Docs-lookup** | `@docs-lookup` | DeepSeek V4 Flash | Looking up library docs, API references |
| **Architect** | `@architect` | DeepSeek V4 Pro | System design, architecture decisions |
| **Code-reviewer** | `@code-reviewer` | DeepSeek V4 Flash | Reviewing code after implementation |
| **Security-reviewer** | `@security-reviewer` | Qwen3.7 Plus | Security audits before commits |
| **Build-error-resolver** | `@build-error-resolver` | Qwen3.7 Plus | Fixing build/type errors |
| **Refactor-cleaner** | `@refactor-cleaner` | MiMo-V2.5 | Cleaning dead code, simplifying |
| **TDD-guide** | `@tdd-guide` | Qwen3.7 Plus | Writing tests, TDD workflow |
| **Doc-updater** | `@doc-updater` | MiMo-V2.5 | Updating documentation |

---

## The Workflow (Step by Step)

### Step 1: Start Session

Open opencode in your project directory. The system automatically loads:
- `AGENTS.md` — project context
- `LEARNED.md` — your accumulated patterns
- `ECC_AGENTS.md` — ECC principles
- All skills in `"instructions"` array

Check if there are learned patterns to apply:
```
What patterns do I have in LEARNED.md?
```

---

### Step 2: Receive Task & Plan

Before writing any code, switch to Plan mode:

```
Tab  →  Plan mode (DeepSeek V4 Pro)
```

**Planning Mode Rules:**
- Always ask clarifying questions before planning
- Never assume design, tech stack, or features
- Use deep-dive sub-agents to assist with research
- Use deep-dive sub-agents to review the different aspects of your plan before presenting to the user

Describe the task:
```
I need to add a new row type to the layout builder that supports 3-column layouts.
Currently rows only support 2 columns. Here's what I need:
- A new constant for the 3-column row
- Update the builder to handle 3-column insertions
- Update the drop zone logic to accept 3-column rows
```

**How Plan handles research (cost-efficient):**

```
You → Plan (DeepSeek V4 Pro)
           ├── delegates research to @explore (Flash, cheap)
           ├── delegates architecture to @architect (Pro, only when needed)
           └── synthesizes final plan (Pro)
```

- `@explore` (Flash — $0.14/1M) reads files, finds patterns, gathers context — 60% of token work at 1/12th the cost
- `@architect` (Pro — $1.74/1M) handles actual system design decisions only when needed
- Plan agent (Pro) takes all outputs and produces the cohesive strategy

The Plan agent will analyze your codebase and create a strategy without making changes.

---

### Step 3: Review Plan

Before the Plan agent presents the strategy, it should:

1. **Ask clarifying questions** if anything is ambiguous — don't assume design decisions
2. **Delegate research** to `@explore` to gather codebase context (cheap, fast)
3. **Delegate review** to `@architect` or `@code-reviewer` to validate different aspects of the plan
4. **Synthesize** all inputs into a final cohesive strategy

**Your review checklist:**

- Does the plan ask clarifying questions (or did it assume too much)?
- Were deep-dive sub-agents used for research?
- Were different aspects reviewed by relevant specialists?
- Does it make sense?
- Are there missing steps?
- Any risks?

Give feedback:
```
Good plan, but also handle the case where someone drags
a 2-column component into a 3-column row's drop zone.
```

---

### Step 4: Implement

Switch back to Build mode:

```
Tab  →  Build mode (Qwen3.7 Plus)
```

Tell it to execute:
```
Good, go ahead with the plan.
```

The Build agent will implement the changes using Qwen3.7 Plus.

---

### Step 5: Review Code

After implementation, launch the code reviewer:

```
@code-reviewer review the changes I just made
```

The reviewer runs in a **child session** — it won't pollute your main context. It returns a summary of issues found.

Address CRITICAL and HIGH issues immediately. MEDIUM/LOW can wait.

---

### Step 6: Fix Errors

If the build fails:

```
@build-error-resolver I'm getting this error:
[paste error]
```

Or switch to Build and ask directly:
```
Tab → Build
The build is failing with [error]. Fix it.
```

---

### Step 7: Security Check (If Needed)

If your changes touch authentication, API routes, user input, or sensitive data:

```
@security-reviewer check the changes I just made for security issues
```

---

### Step 8: Write Tests (If Needed)

For new features or bug fixes:

```
@tdd-guide write tests for the new 3-column row feature
```

Or in Build mode:
```
Tab → Build
Write tests for the 3-column row support in util/builder.ts.
Aim for 80%+ coverage.
```

---

### Step 9: End of Session

Before closing, run `/learn` to capture what you discovered:

```
/learn
```

The AI will:
1. Scan the session for reusable patterns
2. Append them to `LEARNED.md`
3. They'll be loaded automatically next session

**What to learn:**
- Error fixes that weren't obvious
- Codebase quirks you discovered
- Workflow improvements
- Project-specific patterns

**What NOT to learn:**
- Typos or simple syntax errors
- One-time issues (API outages, etc.)
- Trivial fixes

---

## Quick Reference: When to Use What

| Situation | Agent | How |
|-----------|-------|-----|
| "What does this file do?" | Chat | Tab → Chat, ask |
| "Find all API routes" | Explore | `@explore find all API route files` |
| "How do I use this library?" | Docs-lookup | `@docs-lookup how to use dnd-kit sortable` |
| "Plan this feature" | Plan | Tab → Plan, describe task — Plan delegates to `@explore` for research |
| "Design the database" | Architect | `@architect design schema for user auth` |
| "Write the code" | Build | Tab → Build, describe task |
| "Review my code" | Code-reviewer | `@code-reviewer review changes` |
| "Security check" | Security-reviewer | `@security-reviewer check for vulnerabilities` |
| "Fix this error" | Build-error-resolver | `@build-error-resolver [error]` |
| "Clean up this code" | Refactor-cleaner | `@refactor-cleaner simplify this component` |
| "Write tests" | TDD-guide | `@tdd-guide write tests for this feature` |
| "Update docs" | Doc-updater | `@doc-updater update README with new feature` |
| "Learn from this session" | — | `/learn` |

---

## Token Saving Tips

1. **Don't read files in Build mode** — use `@explore` instead (child context)
2. **Don't plan in Build mode** — Tab to Plan first
3. **Plan research goes to Flash** — `@explore` does the heavy lifting, Pro only synthesizes
4. **Use `/compact`** if context gets long (Shift+Tab)
5. **Remove unused skills** from `opencode.jsonc` instructions
6. **Subagents are cheap** — they run in isolated child contexts
7. **Chat mode for questions** — uses cheapest model, no tools executed

---

## ECC Skills Reference

These skills are loaded and guide the AI's behavior:

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

You don't need to reference these explicitly — the AI loads them automatically. But knowing what they cover helps you understand why the AI makes certain suggestions.

---

## Project-Specific Notes (dnd)

This is a **Next.js 16 + React 19 + dnd-kit** drag-and-drop layout builder.

### Key files:
- `app/page.tsx` — Entry point, renders Example
- `components/Example.tsx` — Main layout builder (DndContext)
- `components/Row.tsx`, `Column.tsx`, `Component.tsx` — Layout components
- `components/Dropzone.tsx`, `Trash.tsx` — Drop targets
- `constant/index.ts` — Layout constants and initial data
- `util/builder.ts` — Layout mutation functions
- `util/dnd.ts` — Drop zone rules
- `types/dnd.ts` — TypeScript types

### Commands:
- `npm run dev` — Dev server
- `npm run build` — Production build
- `npm run lint` — ESLint
- `npx tsc --noEmit` — Type check (no script exists, run manually)

### Gotchas:
- No test setup yet — add tests to `util/builder.ts` and `util/dnd.ts` first
- `tsconfig.json` targets ES2017 — consider ES2020+
- Dark mode CSS vars defined but unused by components
- `@dnd-kit/react` is installed but never imported
#   o p e n c o d e - e c c - s t a r t e r  
 