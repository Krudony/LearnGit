# agent.md - Codex AI Assistant Operating Guide

## 📚 Table of Contents

1. [Executive Summary](#executive-summary)
2. [Codex Quick Start](#codex-quick-start)
3. [Project Context](#project-context)
4. [Critical Safety Rules](#critical-safety-rules)
5. [Development Environment](#development-environment)
6. [Development Workflows](#development-workflows)
7. [Context & Slash Commands](#context--slash-commands)
8. [Technical Reference](#technical-reference)
9. [Development Practices](#development-practices)
10. [Lessons Learned](#lessons-learned)
11. [Troubleshooting](#troubleshooting)
12. [Appendices](#appendices)

## Executive Summary

This playbook adapts the `doc.md` guidelines for Codex agents. It defines safe, efficient workflows that work with Codex CLI conventions (PowerShell shell, `apply_patch`, plan tool, etc.) so the agent can ship real changes quickly.

### Key Responsibilities
- Implement and refactor code with clear reasoning.
- Run tests or commands when safe; summarize results.
- Keep documentation, plans, and retrospectives current.
- Follow Codex workflow constraints (no raw `cd`, specify `workdir`, etc.).
- Maintain project context and session history.

### Quick Reference
- `/ccc` – Capture context + compact conversation before planning.
- `/nnn` – Smart planning (auto-runs `/ccc` if needed) and produce detailed execution plan.
- `/gogogo` – Execute the latest plan step-by-step, updating the plan tool.
- `/rrr` – Produce a full session retrospective (what went well, risks, follow-ups).
- `lll` – List repository status (issues/PRs/commits) when needed.

## Codex Quick Start

### Prerequisites
```powershell
# Confirm required tooling inside Codex
node --version
python --version
git --version
rg --version
```

### Codex Execution Basics
1. **Shell usage**
   - Always call `shell` commands with `["powershell.exe","-Command", "..."]` and set `workdir`.
   - Avoid standalone `cd`; pass `workdir` instead.
2. **File edits**
   - Prefer `apply_patch` for single-file edits; ensure patches are ASCII unless needed otherwise.
   - For large or auto-generated changes, use purpose-built tools/commands instead of `apply_patch`.
3. **Plan tool**
   - Use `/nnn` when tasks are non-trivial (more than a single simple edit).
   - Update plan status after completing each step.
4. **Approvals**
   - Respect current `approval_policy`. In `never` mode, do not request escalation—find sandbox-safe alternatives.
5. **Validation**
   - Run relevant tests/linters when feasible; explain if something cannot be run.

### First Task Flow
1. `/ccc` (if necessary) then `/nnn` to build the plan.
2. Execute via `/gogogo`, keeping the plan tool in sync.
3. Provide results plus recommended next steps; `/rrr` on request.

## Project Context

*(Fill this section per repository.)*

### Project Overview
Summarize goals, domain, and success criteria.

### Architecture
- **Backend**: [Framework/Language/DB]
- **Frontend**: [Framework/Language/Libraries]
- **Infrastructure**: [Hosting/CI/CD]
- **Key Libraries**: [Important deps]

### Current Features
- [Feature A]
- [Feature B]
- [Feature C]

## Critical Safety Rules

### Repository Usage
- **Never create issues/PRs on upstream without user approval.**

### Command Usage
- **Never run commands with `-f/--force` unless user mandates it.**
- Provide implications of commands before running them.
- Keep commands reversible; prefer verbose flags for clarity.

### Git Operations
- No `git push --force`, `git checkout -f`, or `git clean -f`.
- Do not merge PRs or run `gh pr merge` unless explicitly told.
- Respect existing user changes; never revert them unless asked.

### File Operations
- Avoid destructive deletion (`rm -rf`). Use interactive or targeted removal if truly necessary.
- Confirm before deleting generated artifacts.

### Package Management
- Avoid blanket updates (`npm update`, etc.) without scope.
- Review lockfile diffs before turning them in.

### Codex-Specific Guardrails
- Stay aware of sandbox (`read-only`, `workspace-write`, `danger-full-access`).
- Honor network restrictions; note when connectivity was required.
- Keep transcripts concise; user controls formatting.

## Development Environment

Document runtime versions, package managers, and tooling expectations. Example checklist:

- Node/Python versions.
- Package manager (`npm`, `pnpm`, `pip`, etc.).
- linters/formatters (ESLint, Prettier, gofmt).
- Test frameworks (Jest, Pytest, etc.).
- Build/CI commands.

## Development Workflows

1. **Context Capture**
   - `/ccc` to log recent tasks, open issues, and constraints.
2. **Planning**
   - `/nnn` to break tasks into substeps, noting dependencies and validation.
3. **Execution**
   - `/gogogo` to follow the plan. Update the plan tool each time a step finishes.
4. **Retrospective**
   - `/rrr` when wrapping up or prompted to log learnings, blockers, metrics.

### Additional Patterns
- Use short, frequent checkpoints. Large tasks should be split into 1-hour slices.
- Note assumptions before coding; confirm with the user when uncertain.
- Keep diffs small and well-scoped.

## Context & Slash Commands

Codex agents must fully support slash-style triggers in addition to shorthand text:

| Command | Purpose |
| --- | --- |
| `/ccc` | Capture latest context and compact conversation logs. |
| `/nnn` | Generate or refresh a detailed implementation plan (auto-calls `/ccc` if stale). |
| `/gogogo` | Execute plan sequentially; report per-step progress and update plan tool. |
| `/rrr` | Produce a retrospective (What/Why/Next/Risks). |
| `/lll` | (Optional) Summarize repo status: branches, open PRs, important commits. |

Always confirm command completion and mention follow-up actions.

## Technical Reference

### Common Commands
```powershell
# Read files (Windows PowerShell)
Get-Content -Raw -Path path/to/file

# List files swiftly
rg --files

# Search within repo
rg "pattern" --type [ext]
```

### Editing Tips
- Use `Set-Content` or `Out-File` carefully—prefer `apply_patch`.
- Keep file encoding ASCII unless existing file dictates otherwise.
- Add concise comments only when code needs clarification.

## Development Practices

### Code Standards
- Follow project style/linting rules.
- Favor explicit types; avoid weak typing (e.g., `any`) unless justified.
- Ensure code is self-explanatory; only add comments when logic is non-obvious.

### Commit Message Template
```
[type]: [brief description]

- What: [specific changes]
- Why: [motivation]
- Impact: [affected areas]

Closes #[issue-number]
```
Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`.

### Error Handling
- Wrap risky operations in try/catch (or project equivalent).
- Emit actionable error messages.
- Provide graceful fallbacks or recovery paths in UI/backends.

## Lessons Learned

Keep this section evolving with project discoveries.

### Planning & Architecture Patterns (2025-08-26)
- Use parallel agents to analyze different subsystems.
- Avoid monolithic plans—favor incremental phases.
- Always ask “What’s the minimum viable first step?”
- 1-hour execution chunks maintain focus and momentum.

### Common Mistakes
- Overly comprehensive initial plans—split into phases.
- Trying to implement everything at once—iterate.
- Skipping AI Diary/Honest Feedback in retrospectives.
- Forgetting lockfile updates after dependency changes.
- Ignoring build warnings that foreshadow errors.

### Useful Tricks
- Parallel analysis for faster planning.
- `ccc → nnn → gogogo` workflow keeps context fresh.
- Phase markers (“Phase 1”, “Phase 2”) improve tracking.

### User Preferences
- Prefers manageable scope (“done in under 1 hour”).
- Appreciates phased approaches and visible progress.
- Likes clear, actionable feedback.
- Timezone: GMT+7 (Bangkok/Asia).

## Troubleshooting

### Build Failures
```powershell
[build-command] 2>&1 | rg -A 5 "error"
# If needing clean install:
Remove-Item -Recurse node_modules, .cache, dist, build
[package-manager] install
```

### Port Conflicts
```powershell
Get-NetTCPConnection -LocalPort [port]
Stop-Process -Id [PID]
```

Document any repo-specific debugging recipes here.

## Appendices

### A. Glossary
- **Term**: Definition.

### B. Quick Command Reference
```powershell
[run-command]          # Start dev server
[test-command]         # Run tests
gh issue create        # Create issue
gh pr create           # Create PR
```

### C. Environment Checklist
- [ ] Runtime versions match project requirements.
- [ ] Package manager installed & locked.
- [ ] GitHub CLI configured.
- [ ] Env variables set (`.env` or secrets).
- [ ] Git identity configured.

**Last Updated**: [Date]
**Version**: 1.0.0
