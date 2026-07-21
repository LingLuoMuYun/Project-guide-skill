---
name: project-guide-skill
description: Analyze, onboard, document, or audit a software project/repository and produce a newcomer-oriented project guide. Use when asked to understand an unfamiliar codebase, explain architecture, map project structure, identify technical stack and conventions, trace core flows, prepare learning documents, generate handoff docs, or create evidence-backed project analysis reports with risks, verification steps, and next actions.
---

# Project Guide Skill

Use this skill to turn an unfamiliar repository into a practical learning guide. Focus on how to run it, where to start reading, how the main flow works, what conventions to follow, what can break, and what still needs confirmation.

This skill is designed for `LingLuoMuYun/Project-guide-skill`. The Codex skill name is `project-guide-skill` to satisfy skill naming rules.

## Operating Rules

- Inspect before concluding. Separate observed facts, documented claims, runtime verification, inferences, and open questions.
- Prefer the repository's actual structure over generic architecture labels.
- Tie important claims to evidence: file paths, configs, commands, test output, or docs.
- Do not print secrets. Mention sensitive config generically without exposing values.
- Avoid exhaustive file-by-file summaries. Explain by role, flow, boundary, convention, and risk.
- If a command is not run, say it was not run and why.
- For user-facing reports in Chinese projects, produce concise Chinese summaries unless the user asks otherwise.

## Analysis Depth

Default to **standard** if the user does not specify depth.

| Depth | Scope | Execution | Output |
| --- | --- | --- | --- |
| quick | README, manifests, top-level dirs, scripts | read-only discovery | stack, key files, obvious risks, next commands |
| standard | quick + entrypoints, core modules, config, tests | safe read-only plus lightweight verification when appropriate | architecture map, startup path, core flow, risks, learning path |
| deep | standard + 1-3 traced flows and relevant contracts/tests | targeted tests/build/typecheck when safe | flow traces, state/contracts, failure modes, missing coverage |
| audit | broad risk surface: auth, config, state, CI/CD, dependencies | validation/dry-run/read-only unless user approves more | evidence ledger, risk register, prioritized remediation |

## Core Workflow

1. Clarify the goal only when necessary: onboarding, architecture report, development guide, debugging map, refactor planning, audit, or handoff.
2. Inventory the repository with fast read-only discovery:
   - top-level files and directories
   - docs, manifests, locks, configs, scripts, tests, CI, containers, env templates
   - generated/vendor/cache/build directories to skip
3. Detect the technical stack:
   - languages, frameworks, package manager, runtime, build tools
   - infrastructure and deployment artifacts
4. Map the project shape:
   - frontend, backend, full-stack, library, CLI, monorepo, data/ML, infra, mobile/desktop, plugin, or mixed system
5. Find entry points:
   - startup commands, app root, route/menu definitions, API clients, service entrypoints, state roots, tests
6. Trace the main path:
   - input/user action -> routing/dispatch -> components/services -> external dependency -> state/output
7. Identify conventions:
   - directory layout, naming, service/type patterns, state usage, style, error handling, permissions, testing
8. Evaluate verification:
   - commands available, commands actually run, what they prove, what remains unverified
9. Produce the guide:
   - overview, stack, architecture, key directories, core flows, development path, risks, open questions, newcomer reading path

## Reference Loading

Load only the reference needed for the current task:

- For the step-by-step checklist, read `references/analysis-checklist.md`.
- For report structures and reusable templates, read `references/output-templates.md`.
- For frontend, Next.js, React, Vue, API clients, routing, state, permissions, and UI projects, read `references/frontend-projects.md`.
- For security/auth/permission/secret-sensitive analysis, read `references/security-review.md`.

## Baseline Discovery

Prefer these read-only commands when available:

```bash
rg --files
ls -la
cat README.md 2>/dev/null | head -80
```

PowerShell equivalents:

```powershell
Get-ChildItem -Force
Get-ChildItem -Recurse -Depth 2 -File | Select-Object -First 120 FullName
Get-Content README.md -TotalCount 80
```

Inspect scripts before running them. Do not run install, update, format, codegen, migration, seed, deploy, apply, destroy, or production-contacting commands unless explicitly requested or clearly safe.

## Evidence Labels

Use labels when helpful:

- **Observed**: confirmed by reading files.
- **Documented**: stated in docs, not independently verified.
- **Runtime-verified**: confirmed by command/test/API output.
- **Inferred**: reasonable conclusion from structure or names.
- **Open**: needs user, backend, ops, or runtime confirmation.

## Default Output

For standard project guides, include:

1. Evidence summary
2. Project overview
3. Technology stack
4. Directory and key file map
5. Startup and runtime path
6. Architecture and core flow
7. Core modules and responsibilities
8. State, contracts, and external dependencies
9. Development workflow and conventions
10. Testing and verification
11. Risks and open questions
12. Recommended next steps
13. Newcomer reading path

For reviews, lead with risks. For debugging, lead with symptoms and isolation plan. For handoff, lead with what works, what is blocked, and what should not be changed yet.

## Completion Criteria

Before finalizing, ensure the analysis answers or labels open:

- What is this project and what value does it provide?
- How is it installed, started, built, tested, and deployed or packaged?
- Where are the main entrypoints and runtime boundaries?
- What are the most important data/control flows?
- Where do config, state, schemas/contracts, and external dependencies live?
- What has been verified by commands or runtime behavior?
- What are the highest-risk areas and safest next actions?
- What should a newcomer read first?
