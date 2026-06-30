# bigin-skills

**BigIn's collection of Claude Code skills**
_Bộ skill Claude Code của BigIn_

Skills for standardized, AI-assisted development across BigIn's stacks.

---

## Skills

| Skill                  | Purpose                                                                                                  |
| ---------------------- | -------------------------------------------------------------------------------------------------------- |
| **bigin-harness-setup** | Scaffolds an AI workflow harness into a repo — `CLAUDE.md`, scoped rules, and enforcement gates. Profiles: `nuxt`, `go`, `nodejs`. |
| **session-handoff**     | Saves session state (tasks, decisions, uncommitted changes) to `SESSION.md` and restores it on resume.   |

---

## bigin-harness-setup

Sets up a consistent "harness level" on any repo so team members of mixed skill levels produce consistent, maintainable output.

### Principles

- **Guidance defines intent; gates enforce it mechanically.** Anything left to judgment varies by skill level — so the value is in the gates, not more docs.
- **Single source of truth.** Reference shared rules, never duplicate them.
- **No overhead.** Lean, scannable markdown — a rule nobody reads is worse than no rule.
- **Additive-first cross-repo contract.** `openapi.yaml` is the contract between frontend and backend. Backend leads with backward-compatible changes; a breaking change requires a version bump. Frontend generates types from `openapi.yaml` — never hardcoded.

### Profiles

| Profile  | Stack                                                                          |
| -------- | ------------------------------------------------------------------------------ |
| `nuxt`   | Nuxt 4 (SPA), Pinia + Pinia Colada, VueUse, Nuxt UI, Zod, Vitest, Cloudflare Pages |
| `go`     | Go REST API (Gin)                                                              |
| `nodejs` | Node.js TypeScript REST API                                                    |

### What gets generated

```
your-repo/
├── CLAUDE.md                      ← thin contract: stack, commands, hard rules
├── AI_TASK_GUIDE.md               ← per-task workflow + spec gate
├── AI_REVIEW_CHECKLIST.md         ← definition of done (profile commands filled in)
├── .claude/
│   ├── rules/
│   │   ├── conventions.md         ← naming, patterns (profile-specific)
│   │   ├── security.md            ← shared security rules
│   │   └── architecture.md        ← shared base + profile addendum
│   ├── guards/
│   │   └── bash-guard.py          ← blocks --no-verify and force-push to main
│   ├── settings.json              ← pre-approved commands + hook wiring
│   └── agents/
│       └── code-reviewer.md       ← optional, read-only (opt-in)
├── scripts/
│   └── pre-commit.sh              ← lint + typecheck + test for the profile
└── README.md                      ← AI Onboarding section appended
```

### Usage

Trigger in Claude Code with:

```
Set up a harness
Add AI rules to this repo
Thiết lập harness
```

The skill detects the stack profile (or asks), confirms before overwriting anything, and prints onboarding next steps. Re-running on an already-set-up repo is safe (idempotent).

### Enforcement (the load-bearing part)

- **`scripts/pre-commit.sh`** — runs lint + typecheck + tests; fails closed.
- **`.claude/guards/bash-guard.py`** — a `PreToolUse` hook that blocks the agent from weakening its own gates (`--no-verify`, `git commit -n`, force-push to main). `--force-with-lease` on a feature branch is allowed.
- **`.claude/settings.json`** — pre-approves safe profile commands to reduce prompt friction.

---

## Installation / Cài đặt

### Via Marketplace

```
/plugin marketplace add tammai/bigin-skills
/plugin install bigin-skills@bigin
```

### Direct (single skill)

```bash
cp -r skills/bigin-harness-setup ~/.claude/skills/bigin-harness-setup
```

---

## Plugin Structure

```
bigin-skills/
├── .claude-plugin/
│   ├── plugin.json                ← plugin metadata (name, version, author)
│   └── marketplace.json           ← marketplace registry entry
├── skills/
│   ├── bigin-harness-setup/       ← harness scaffolder
│   │   ├── SKILL.md               ← 8-phase workflow
│   │   └── references/
│   │       ├── profile-nuxt.md
│   │       ├── profile-go.md
│   │       ├── profile-nodejs.md
│   │       ├── files-shared.md    ← security, architecture, task guide, review checklist, code-reviewer
│   │       └── hook-guard.md      ← bash-guard.py + pre-commit scripts per profile
│   └── session-handoff/           ← session state persistence
│       └── SKILL.md
├── CHANGELOG.md
└── README.md
```

---

## License

MIT
