# `.claude` — Agents, Skills & Prompts

This repository is a **version-controlled home directory for your Claude Code configuration**.
Everything under `agents/`, `skills/`, and `prompts/` is committed to git so it can be shared,
reviewed, and rolled back. Machine-specific runtime state (sessions, caches, history) is
excluded via [.gitignore](.gitignore).

At its core this repo holds an **automated development pipeline**: a three-tier system of
agents that takes a raw requirement and delivers tested, CI-green, deployed code. The agents
are deliberately **generic** — they discover the target project's stack, conventions, and
tooling at runtime rather than hard-coding framework specifics, so they work against any repo.

---

## Quick links

| What | Where |
|------|-------|
| Agents (full detail) | [`agents/README.md`](agents/README.md) |
| Skills | [`skills/`](skills/) — one directory per skill, each with a `SKILL.md` |
| Prompts | [`prompts/`](prompts/) — parameterised slash-command tasks |
| Tool permission reference | [`agents/TOOLS.md`](agents/TOOLS.md) |
| Excluded files | [`.gitignore`](.gitignore) |

---

## Agents

Agents are invoked with `@agent-name`. They are organised in three tiers — a router, workflow
agents that own a process end-to-end, and specialists that do one job well.

### Tier 1 — Intent router

| Agent | File | What it does |
|-------|------|--------------|
| **orchestrator** | [`orchestrator.agent.md`](agents/orchestrator.agent.md) | Thin router. Parses your request, runs an `init` pre-flight if project scaffolding is missing, then delegates to the right Tier 2 workflow agent. Contains no workflow logic itself. |

### Tier 2 — Workflow agents

These own a full process and coordinate Tier 3 specialists.

| Agent | File | What it does |
|-------|------|--------------|
| **feature-delivery** | [`feature-delivery.agent.md`](agents/feature-delivery.agent.md) | End-to-end feature pipeline: spec → implement → review → quality-gate → deploy → learn. Accepts requirement text, a spec file, or a `plan/ROADMAP.md` heading. |
| **bug-fix** | [`bug-fix.agent.md`](agents/bug-fix.agent.md) | Defect repair: reproduce → root-cause → minimal targeted fix → regression test → verify. Accepts a description, a `plan/BUG_TRACKER.md` entry, or an issue number. |
| **refactor** | [`refactor.agent.md`](agents/refactor.agent.md) | Analysis & remediation: architect/code-reviewer audit → triage → implementer fixes → verify → deploy. Supports `report-only` / `audit-only` to suppress auto-fix. |
| **release-manager** | [`release-manager.agent.md`](agents/release-manager.agent.md) | Production releases: determine release type → bump version → changelog → pre-flight quality gate → deploy → git tag. Loops back through the implementer if the gate fails. |

### Tier 3 — Specialists

Single-purpose agents. Most are invoked *by* Tier 2 agents rather than directly.

| Agent | File | What it does |
|-------|------|--------------|
| **spec-expander** | [`spec-expander.agent.md`](agents/spec-expander.agent.md) | Bridges a one-line requirement to a fully-specified, testable change. Writes a spec to `specs/<slug>.md`. |
| **implementer** | [`implementer.agent.md`](agents/implementer.agent.md) | Writes and modifies code under strict test-driven discipline, running its own tests incrementally. Accepts a spec path or a fix-list. |
| **code-reviewer** | [`code-reviewer.agent.md`](agents/code-reviewer.agent.md) | Reviews code smells, design issues, AI-generated code pitfalls, and maintainability. Four scopes: file, branch, commit, or whole project. Writes `agent-output/Code-Review.md`. |
| **architect** | [`architect.agent.md`](agents/architect.agent.md) | Audits the codebase against project standards (`.github/copilot-instructions.md`) and engineering best practices, categorised by severity. Documents findings — does not fix code. |
| **quality-gate** | [`quality-gate.agent.md`](agents/quality-gate.agent.md) | Runs unit tests, lint, and E2E in sequence. On failure it auto-invokes the implementer and re-runs — a self-healing feedback loop. Returns "all green" or "blocked after N retries". |
| **designer** | [`designer.agent.md`](agents/designer.agent.md) | Full-scope creative direction and front-end design: site-wide redesigns, targeted refinements, or inspiration-driven redesign from a reference URL. |
| **deployer** | [`deployer.agent.md`](agents/deployer.agent.md) | Runs the deployment pipeline and reports the outcome. Assumes quality gates already passed (defaults to `--skip-local`). |
| **mentor** | [`mentor.agent.md`](agents/mentor.agent.md) | Meta-agent. Reviews a completed session and extracts lessons learned into targeted improvements for every agent whose instructions were exercised. |
| **init** | [`init.agent.md`](agents/init.agent.md) | Idempotent scaffolding. Creates only what is missing, never overwrites. Run manually to bootstrap a project, or automatically by the orchestrator when project instructions are absent. |
| **digester** | [`digester.agent.md`](agents/digester.agent.md) | Analyzes the codebase and updates agent instructions to stay accurate as the project changes. |
| **knowledge-graph-builder** | [`knowledge-graph-builder.md`](agents/knowledge-graph-builder.md) | Generates a structured knowledge graph of a codebase — entities, dependencies, inheritance, and call relationships — per `plan/AI_CONTEXT_BUILD.md`. |
| **change-summary-generator** | [`change-summary-generator.md`](agents/change-summary-generator.md) | Diffs the current branch against `main` and writes a human-readable, educational change summary to `agent-output/CHANGE-SUMMARY.md`. Useful for onboarding and PR context. |

### How they fit together

```
Your request
    ↓
orchestrator ──(runs init if needed)
    ↓
feature-delivery / bug-fix / refactor / release-manager
    ↓
spec-expander → implementer → code-reviewer → quality-gate → deployer → mentor
```

---

## Skills

Skills are **on-demand knowledge modules** — a directory under [`skills/`](skills/) containing
a `SKILL.md`. Agents load them at runtime when relevant; they are never injected by default.
Some are marked `invocation: manual`, meaning you trigger them explicitly.

| Skill | File | What it does |
|-------|------|--------------|
| **bug-diagnosis** | [`skills/bug-diagnosis/SKILL.md`](skills/bug-diagnosis/SKILL.md) | Builds a precise mental model before any fix, by asking one focused clarifying question at a time. Use when investigating a bug, unexpected behaviour, test failure, or error — before touching code. |
| **concept** | [`skills/concept/SKILL.md`](skills/concept/SKILL.md) | Gives a quick structured overview of any concept — basics of a topic, or a high-level map of a complex subject. |
| **create-playwright-tests** | [`skills/create-playwright-tests/SKILL.md`](skills/create-playwright-tests/SKILL.md) | Generates or updates Playwright E2E tests. Initialises config if needed and enforces best practices: correct HTTP methods, no default values, real user actions over mocks. |
| **diff-decision-walkthrough** | [`skills/diff-decision-walkthrough/SKILL.md`](skills/diff-decision-walkthrough/SKILL.md) | Explains a diff as a series of embedded design decisions, then answers your questions about them. Works from a branch diff, commit pair, or `.diff`/`.patch` file. A comprehension tool, not a repair tool — best used on AI-generated code before merging. |
| **knowledge-graph** | [`skills/knowledge-graph/SKILL.md`](skills/knowledge-graph/SKILL.md) | Builds or updates a three-layer Markdown knowledge base for any codebase: `START_HERE.md`, `Knowledge/KNOWLEDGE_GRAPH.md`, and `AGENTS.md`. Classifies files by authority tier and maps real relationships between them. |
| **repo-vuln-audit** | [`skills/repo-vuln-audit/SKILL.md`](skills/repo-vuln-audit/SKILL.md) | Scans a repo for exposed secrets and overly-permissive GitHub config: leaked API keys/tokens/private keys, workflow permissions, `pull_request_target` RCE vectors, branch protection, repo visibility, CODEOWNERS. **Read-only by design** — never writes or mutates, so it is safe to run unattended against untrusted repos. |
| **security-check** | [`skills/security-check/SKILL.md`](skills/security-check/SKILL.md) | Vet an *untrusted* repository before installing it: OWASP Top 10, malware and keylogger patterns, obfuscated code, excessive privileges, unexpected network activity, and malicious install scripts. Manual invocation. |
| **socratic-code-review** | [`skills/socratic-code-review/SKILL.md`](skills/socratic-code-review/SKILL.md) | A learning-focused review that asks 3 challenging questions per round instead of rewriting your code. Covers memory leaks, thread safety, distributed-systems risk, SOLID, YAGNI/KISS, performance, and security — adapting to your experience level and producing a structured review report. |

### Linked skills

Three entries in `skills/` are symlinks to a shared location outside this repo and are not
tracked here:

- `skills/copy-editing@` → `~/.agents/skills/copy-editing`
- `skills/design-taste-frontend@` → `~/.agents/skills/design-taste-frontend`
- `skills/minimalist-ui@` → `~/.agents/skills/minimalist-ui`

These are shared across tool configs. They will not resolve on a fresh clone until the symlink
target exists.

---

## Prompts

Prompts are parameterised slash-command tasks in [`prompts/`](prompts/). Each is a single
Markdown file whose text becomes the task specification when invoked.

| Prompt | File | What it does |
|--------|------|--------------|
| **summarize** | [`prompts/summarize.md`](prompts/summarize.md) | Summarises session progress so it can be picked up later — what's completed, what's been tried, what remains. Writes to `agent-output/chat-summary.md`. |

---

## Artefacts these agents expect

Agents write their output to conventional paths. If your target project doesn't have them,
run `@init` to scaffold.

| Artefact | Path | Produced by |
|----------|------|-------------|
| Specifications | `specs/<slug>.md` | spec-expander |
| Project instructions | `.github/copilot-instructions.md` | init |
| Roadmap | `plan/ROADMAP.md` | manual / init |
| Bug tracker | `plan/BUG_TRACKER.md` | manual |
| Changelog | `CHANGELOG.md` | release-manager |
| Architecture report | `agent-output/Architect-Review.md` | architect |
| Code review report | `agent-output/Code-Review.md` | code-reviewer |
| Quality gate report | `agent-output/quality-gate.md` | quality-gate |
| Design summary | `agent-output/design-summary.md` | designer |
| Mentor report | `agent-output/Mentor-Report-*.md` | mentor |
| Change summary | `agent-output/CHANGE-SUMMARY.md` | change-summary-generator |
| Session summary | `agent-output/chat-summary.md` | summarize prompt |

---

## What's tracked vs. ignored

**Tracked** — the portable configuration worth sharing and reviewing:
`agents/`, `skills/`, `prompts/`, `settings.json`, `README.md`.

**Ignored** — machine-specific runtime state that would leak context or churn the diff.
See [.gitignore](.gitignore) for the full list; it covers sessions, history, caches, backups,
plugins, IDE locks, and secrets (`*.key`, `*.pem`, `*.env`).

---

## Example invocations

```
@orchestrator Add rate limiting to the API
@orchestrator Fix the login page crashing on empty email input
@orchestrator Analyse the codebase

@init                                  # scaffold a new project (or "report-only")
@spec-expander Add a caching layer to the data access module
@implementer specs/add-caching-layer.md
@code-reviewer scope:branch target:feat/add-caching
@architect focus:performance and testing
@designer redesign the site inspired by https://example.com
@change-summary-generator              # document this branch vs main
```

---

## Adding your own

**Agent** — drop a `.md` file in `agents/` with YAML frontmatter: `name`, `description`,
optional `argument-hint` and `tools`. See [agents/README.md](agents/README.md) for the
conventions and [agents/TOOLS.md](agents/TOOLS.md) for the tool-permission groupings.

**Skill** — create `skills/<name>/SKILL.md` with frontmatter `name` and `description`
(optionally `invocation: manual`), then write the instructions below it.

**Prompt** — create `prompts/<name>.md` containing the task text, with any output path
spelled out explicitly.