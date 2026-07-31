# Copilot Instructions

## Purpose of this repository

This repo is a **central hub for agentic coding** — a tool-agnostic library of agents, skills, and orchestration prompts shared across multiple AI coding hosts (GitHub Copilot, Cursor, Claude/OpenCode-style runners, etc.).

It is **not an application codebase**. There is no source code to build, no tests, and no runtime. Every file is Markdown with YAML frontmatter that an external AI host loads at runtime.

The hub is used to:

1. **Generate project ideas** from a rough brief or domain.
2. **Produce project documents and plans**, including HLD (high-level design) and LLD/SLD (low-/system-level design) specs, edge-case analysis, and ordered implementation steps — all without writing production code in this repo.
3. **Add or update features for an external project context** — when a user supplies a target codebase, the agents here research it, plan changes, and (via the `coder`/`designer` roles) implement them in that target project, not here.

When working in this repo, you are editing the *prompts and configuration* that drive those workflows — not the deliverables themselves.

## Directory layout and how each tree is consumed

Three parallel trees hold the same conceptual content for different AI hosts. Keep them aligned when content overlaps.

- `.cursor/` — Cursor host
  - `agents/{coder,designer,planner}.md` — subagent definitions with frontmatter (`name`, `model`, `description`, `is_background`). These three are the **only** delegatable roles.
  - `commands/orchestrate.md` — the top-level command that breaks a request into phases, assigns files per task, and runs agents in parallel when their file scopes don't overlap.
  - `skills/{frontend-design,web-design-guidelines,structured-autonomy-plan,structured-autonomy-generate,structured-autonomy-implement}/SKILL[S].md` — skill prompts.
- `.github/` — GitHub Copilot host. Mirrors the relevant parts of `.cursor/`:
  - `agents/{coder,designer,planner}.md` — **mirror** of `.cursor/agents/`. Skill path references inside these files point at `.github/skills/...` instead of `.cursor/skills/...`; everything else is byte-equivalent.
  - `commands/orchestrate.md` — **mirror** of `.cursor/commands/orchestrate.md`.
  - `skills/…` — **mirror** of `.cursor/skills/`.
  - `copilot-instructions.md` — this file.
- `.agents/skills/ui-ux-pro-max/` — host-agnostic skill (`SKILL.md` + empty `data/` and `scripts/` placeholders). Referenced by literal path from `.cursor/agents/designer.md`.
- `tasks/` — **one directory per agentic coding task**. Every new request gets its own folder; see "Task directory layout" below. (Legacy `plans/` may still exist as scratch space — prefer `tasks/<task-slug>/plans/` for new work.)
- `.copilot/mcp-config.json` — project-level MCP server config (shadcn/ui, Tailwind).

## Workflow this repo implements

The intended flow when a user makes a request is:

1. **`orchestrate` (Cursor command / equivalent host entry point)** receives the user's request.
2. **Create a task directory** under `tasks/` before any other work (see next section). All subsequent artifacts for this request live there.
3. It calls **`planner`** to research and emit an ordered plan: summary → steps → edge cases → open questions. The planner is also the source of HLD/LLD spec output and never writes code. Planner writes its output into the task directory.
4. `orchestrate` parses the plan into phases. Tasks with **non-overlapping file lists** run in parallel; overlapping ones run sequentially.
5. Each phase delegates to **`coder`** (logic, scaffolding, tests) and/or **`designer`** (UI, tokens, layout, accessibility). Delegation describes **WHAT** to do, never **HOW**. Designer drops design artifacts into the task's `designs/` folder.
6. After all phases, `orchestrate` verifies, updates the task's `README.md` status, and reports.

## Task directory layout

**Every new agentic coding task gets its own directory under `tasks/`.** Do not scatter plans, specs, or designs at the repo root or in shared folders.

```
tasks/
  YYYY-MM-DD-<short-kebab-slug>/        e.g. tasks/2026-06-30-dark-mode-toggle/
    README.md            Brief: user request, target project (if external),
                         status (planning | in-progress | done | blocked),
                         links to the artifacts below.
    plans/               Planner output: implementation steps, edge cases,
                         open questions, phase/parallelization map.
    specs/               HLD and LLD/SLD: component diagrams, data contracts,
                         sequence flows, deployment/runtime constraints,
                         non-functional requirements.
    designs/             Designer output: color/type/layout tokens, ASCII
                         wireframes, signature element notes, accessibility
                         checklist, references to any built UI files.
    notes/               (optional) Research, links, decisions log,
                         rejected alternatives.
```

Rules:

- **Slug format:** `YYYY-MM-DD-<kebab-case-summary>`. Date first so directory listings sort chronologically.
- **Copy `tasks/_template/`** to bootstrap a new task — it already contains `plans/`, `specs/`, `designs/`, `notes/`, and a populated `README.md` skeleton. Fill in the README's Title / Status / Target project / Request fields before delegating.
- **One task per directory.** If a request expands in scope, either extend the existing task or spin off a new dated directory and link them from each `README.md`.
- **External-project tasks still create a directory here.** The plans/specs/designs live in this repo; only the implemented code lands in the external project. Record the external project's path or repo URL in `README.md`.
- **Agents must write to the task's sub-directory, not the repo root.** `planner` → `plans/` and `specs/`; `designer` → `designs/`; `coder` → external project (or, for prompt-only changes, the relevant `.cursor`/`.github`/`.agents` path).

The `structured-autonomy-{plan,generate,implement}` skills encode the same plan→generate→implement pipeline as standalone prompts for hosts that don't use the orchestrator command.

## Authoring conventions specific to this repo

- **Three-role contract.** The only agents are Planner / Coder / Designer. Don't add a new role to `.cursor/agents/` without also wiring it into `orchestrate.md`'s agent list, parallelization rules, and example sections.
- **Frontmatter shape.** Agents use `name`, `model`, `description`, `is_background`. Skills use `name`, `description`, optionally `license` / `metadata`. Model strings (`claude-opus-4.7[]`, `gpt-5.3-codex`, `gpt-5.5[]`, `composer-2.5[]`) are literal — preserve them exactly.
- **File-scoped parallelization is the core invariant.** Every delegated task in `orchestrate.md` carries an explicit file list. Keep this when editing examples; it's how the orchestrator avoids write conflicts between parallel agents.
- **Outcome-oriented delegation.** Prompts must say what should be true when done, not which functions/hooks to use. The "CRITICAL: Never tell agents HOW" section in `orchestrate.md` is the canonical rule.
- **Cross-skill references are path-literal.** `designer.md` reads `.cursor/skills/frontend-design/SKILL.md` and `.agents/skills/ui-ux-pro-max/SKILL.md` by path. Grep before renaming or moving any skill.
- **`web-design-guidelines` fetches a live URL** at runtime (`https://raw.githubusercontent.com/vercel-labs/web-interface-guidelines/main/command.md`). Don't inline or cache its content.
- **`coder` must web-search docs every invocation.** This is intentional (combats stale training data). Don't add "skip lookup if familiar" overrides.
- **Designer calibration.** `frontend-design/SKILL.md` explicitly calls out three AI-default looks to avoid (cream + terracotta + serif; near-black + acid green; broadsheet hairline). Other design prompts assume this baseline — don't reintroduce those defaults as examples.

## When generating ideas / docs / HLD-LLD plans

- Output goes into the current task directory's `plans/` and `specs/` sub-folders — never the repo root.
- Use the planner's required structure: **Summary → Implementation steps → Edge cases → Open questions**. For HLD/LLD in `specs/`, expand into component diagrams (ASCII is fine), data contracts, sequence flows, and deployment/runtime constraints. The planner's "Edge-case categories" checklist (input validation, authz, concurrency, state, network, security, UX, ops) is the spec coverage bar.
- A plan is good enough only if someone unfamiliar with the target codebase could implement it correctly on the first pass.

## When adding/updating features for an external project context

- Treat the external project as read-mostly: `planner` researches it, `coder`/`designer` implement changes **there**, and this repo only changes if the agent/skill prompts themselves need updating.
- If the user's request reveals a missing capability (new skill, new agent behavior, new orchestration phase), update the prompts here in the same session and call out the change in the response.

## Editing checklist for changes in this repo

1. For any new user request, create `tasks/YYYY-MM-DD-<slug>/` with `plans/`, `specs/`, `designs/`, and a `README.md` **before** delegating to agents.
2. If you touch `.cursor/{agents,commands,skills}/<x>`, update the matching `.github/{agents,commands,skills}/<x>` (and vice versa). The only allowed divergence is skill-path references inside `.github/agents/designer.md`, which point to `.github/skills/...`.
3. If you add or rename an agent/skill, grep the three trees and `orchestrate.md` for the old path/name and update every reference.
4. Preserve YAML frontmatter shape and model strings exactly — host parsers are strict.
5. Don't introduce package managers, CI, build scripts, or runnable code. Verification is review-by-reading.
