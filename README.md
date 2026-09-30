# Simple Skills

A small set of essential agent skills, kept simple on purpose.

Each skill is short, concise, and does one job. No long checklists, no over-engineering. If a skill can be shorter, it should be.

## Vision

- Essential only: provide the skills people actually need, nothing more.
- Simple over complete: a skill can run to thousands of lines, but that defeats the purpose. Short instructions are easier to read, maintain, and follow.
- Minimal formatting: plain headings, short lists, and small tables. Bold and decoration only when they help the agent get a better result. This keeps every skill easy to read for humans, and clear structure is what the agent relies on anyway.

## Skills

| Skill | Purpose | Manual only |
|-------|---------|:-----------:|
| `simple-check` | Pre-ship evaluation of code changes | No |
| `simple-debug` | Find serious bugs before ship | No |
| `simple-code-review` | Review quality, security, and maintainability | Yes |
| `simple-refactor` | Minimal refactor, remove redundancy (DRY) | No |
| `simple-restructure` | Group files into domain folders, no logic changes | Yes |
| `simple-commit` | Conventional commit with a secrets and junk-file check | No |
| `simple-text-cleanup` | Clean up text formatting and punctuation | No |
| `simple-orchestrator` | Multi-agent workflow: plan, dispatch workers, review | Yes |
| `simple-cloudflare` | Guided Cloudflare setup: DNS, SSL, caching, headers, email anti-spoofing | No |
| `simple-supabase` | Guided Supabase setup: schema, RLS, keys, auth | No |
| `simple-launch` | Pre-launch checklist: secrets sweep, metadata, SEO, performance | Yes |
| `simple-release` | Version bump, changelog, and git tag | Yes |
| `simple-readme` | Write or update a developer-focused README | No |
| `simple-chinese` | Concise, natural written Chinese for docs and UI copy | No |
| `simple-cantonese` | Natural Hong Kong spoken Cantonese copy | No |

### Why some skills are manual only

Manual only skills set `disable-model-invocation: true`, so the agent never runs them on its own. You call them by name, such as `/simple-orchestrator`.

| Skill | Why |
|-------|-----|
| `simple-orchestrator` | Spawns several agents, so it costs more |
| `simple-restructure` | Moves many files |
| `simple-release` | Bumps the version and creates tags |
| `simple-launch` | Scans the whole repo and the build output |
| `simple-code-review` | A broad review that overlaps built-in review commands, so you pick the moment |

All other skills load automatically when the task matches. Other skills and agents rely on this: the reviewer runs `simple-check`, and the language skills run `simple-text-cleanup`. To change a skill, add or remove `disable-model-invocation: true` in its `SKILL.md` frontmatter.

### No AI attribution

`simple-commit` and `simple-release` never add `Co-Authored-By`, "Generated with", or other AI attribution to commits, even when your agent's default says to. This keeps the history clean. If you want the attribution, remove that rule from each skill.

## Requirements

- An agent that supports the `SKILL.md` format: **Claude Code**, **Cursor**, or **Codex**
- **Node.js 18+**, only for the `npx` install

## Quick Setup

### 1. Install with npx

```bash
npx skills add denniemok/simple-skills
```

Pick your agent when prompted. To install a single skill:

```bash
npx skills add denniemok/simple-skills --skill simple-check
```

### 2. Install manually

Clone the repo and copy the skill folders you want into your agent's skills directory.

```bash
git clone https://github.com/denniemok/simple-skills.git
```

| Agent | Global (all projects) | Project only |
|-------|-----------------------|--------------|
| Claude Code | `~/.claude/skills/` | `.claude/skills/` |
| Cursor | `~/.cursor/skills/` | `.cursor/skills/` |
| Codex | `~/.codex/skills/` | `.agents/skills/` |

Example for Claude Code:

```bash
cp -r simple-skills/simple-check ~/.claude/skills/
```

Each skill is a folder containing a `SKILL.md`, so copy the whole folder.

### 3. Install subagents (simple-orchestrator only)

Installing the skill does not install its subagents. To use the orchestrator's multi-agent workflow, install the subagents yourself: copy the agent files to your agent's agents directory.

| Agent | Files to copy | Global | Project only |
|-------|---------------|--------|--------------|
| Claude Code | `simple-orchestrator/agents/*.md` | `~/.claude/agents/` | `.claude/agents/` |
| Cursor | `simple-orchestrator/agents/*.md` | `~/.cursor/agents/` | `.cursor/agents/` |
| Codex | `simple-orchestrator/agents/codex/*.toml` | `~/.codex/agents/` | `.codex/agents/` |

Without them the orchestrator cannot spawn its explorer, worker, and reviewer, and will ask you to install them first. The agent files use your current model. Set a model per agent in its frontmatter (Claude Code, Cursor) or with `model` in the TOML (Codex) if you want.

Codex limits how many subagents run at once. To allow parallel workers, set it in `.codex/config.toml`:

```toml
[agents]
max_concurrent_threads_per_session = 4
```

### 4. Use

- Manual skills: type the skill name as a command, such as `/simple-check`
- Auto skills: the agent picks them up when the task matches

## Structure

```
simple-skills/
├── simple-check/                  # Pre-ship evaluation
├── simple-debug/                  # Serious bug hunt
├── simple-code-review/            # Code review
├── simple-refactor/               # Minimal DRY refactor
├── simple-restructure/            # Domain-based file layout
├── simple-commit/                 # Commit message and git flow
├── simple-text-cleanup/           # Text formatting cleanup
├── simple-orchestrator/           # Multi-agent workflow
│   ├── SKILL.md
│   └── agents/                    # Explorer, worker, and reviewer subagents
│       └── codex/                 # Same subagents as .toml for Codex
├── simple-cloudflare/             # Cloudflare setup guide
├── simple-supabase/               # Supabase setup guide
├── simple-launch/                 # Pre-launch checklist
├── simple-release/                # Release and changelog
├── simple-readme/                 # README writer
│   ├── SKILL.md
│   └── template.md                # README skeleton
├── simple-chinese/                # Written Chinese
├── simple-cantonese/              # Spoken Cantonese
├── LICENSE
└── README.md
```

Every skill folder holds a `SKILL.md` (frontmatter plus instructions).
