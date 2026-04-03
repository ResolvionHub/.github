# ResolvionHub — Agentic SDLC Repository Setup Proposal

**Date:** 2026-03-26
**Author:** Claude (Platform Setup Agent)
**Status:** Proposal — ready for review

---

## Executive Summary

ResolvionHub has 6 GitHub repositories designed to support an agent-augmented software development lifecycle. One repo (`platform-context`) contains a completed RDN integration spec. The other five are empty. This proposal defines exactly what files each repo needs and what goes in them so that Claude Code and Claude Projects can effectively build, review, and manage the Resolvion platform.

---

## How Agents Use These Repos

```
Developer opens Claude Code in platform-core
        │
        ▼
Claude Code reads CLAUDE.md (auto-loaded at repo root)
        │
        ▼
CLAUDE.md tells it: "Read the relevant spec from platform-context via GitHub MCP"
        │
        ▼
Agent reads context-index.md → loads rdn.md → starts implementing
```

Three mechanisms make this work:

1. **`CLAUDE.md`** — Claude Code auto-reads this file at the repo root and in the current working directory. It tells the agent what this repo is, what conventions to follow, and where to find more context.

2. **GitHub MCP Server** — connected to Claude Projects, this gives agents live read access to all ResolvionHub repos. Agents pull specs, rules, and standards at conversation start.

3. **`.claude/commands/`** — custom slash commands that give Claude Code reusable workflows (e.g., `/build`, `/review`, `/new-service`).

---

## Repository Map

| Repo | Purpose | Consumers |
|------|---------|-----------|
| **platform-context** | What to build — service specs, core business context, glossary | All agents |
| **platform-prompts** | How agents behave — personas, shared rules, bootstrap templates | Claude Projects |
| **platform-core** | The code — C#/.NET implementation of specs | Developer + QA agents |
| **platform-docs** | Operational knowledge — runbooks, ADRs, standards, onboarding | All agents + humans |
| **platform-compliance** | Guardrails — SCRA, fee rules, data handling, escalation triggers | Compliance + all agents |
| **platform-infra** | Deployment — Bicep, Dockerfiles, pipelines, scripts | Developer + platform agents |

---

## What Each Repo Needs

### Legend

| File | Why Agents Need It |
|------|-------------------|
| `CLAUDE.md` | Auto-loaded by Claude Code — orients the agent to the repo |
| `.claude/commands/*.md` | Custom slash commands for Claude Code workflows |
| `.gitignore` | Prevents build artifacts and secrets from being committed |
| `.github/CODEOWNERS` | Enforces PR review by the right team |
| `.github/pull_request_template.md` | Standardizes PR descriptions for human and agent reviewers |
| `README.md` | Human-readable orientation + setup checklist |

These 6 files are the **minimum viable setup** for every repo. Below is what each repo needs beyond that.

---

### 1. platform-context

**Purpose:** The knowledge base. Every service spec, core business context, and domain reference lives here. Agents read from this repo to understand what to build.

**What exists today:** `rdn.md` (RDN integration spec, ~44KB)

```
platform-context/
├── CLAUDE.md                     # Orients Claude Code — explains this is a multi-service spec library
├── README.md                     # Human orientation + setup checklist
├── .gitignore
├── .github/
│   ├── CODEOWNERS
│   └── pull_request_template.md
├── .claude/commands/
│   ├── new-spec.md               # /new-spec — creates a spec from template
│   └── validate-spec.md          # /validate-spec — checks a spec against authoring standards
│
├── context-index.md              # CRITICAL — routing table mapping files → agents → loading strategy
│
├── core/                         # Foundational context (not service-specific)
│   ├── company-overview.md       # What Resolvion does, stakeholders, repossession lifecycle
│   ├── business-rules.md         # Cross-cutting rules: fees, data handling, integrations
│   └── decision-principles.md    # How we make technical and product decisions
│
├── glossary.md                   # Domain terms shared across all specs
├── _spec-template.md             # Template for new service specs
│
└── rdn.md                        # [EXISTS] RDN integration spec — first of many
```

**Key file: `context-index.md`** — this is the routing table. Every agent reads it first to determine which files to load for their role and current task. Without it, agents either load everything (wasting context window) or nothing (missing critical information).

---

### 2. platform-prompts

**Purpose:** Defines how each agent behaves. Personas, shared rules, and the bootstrap template that goes into each Claude Project's system prompt.

```
platform-prompts/
├── CLAUDE.md
├── README.md
├── .gitignore
├── .github/
│   ├── CODEOWNERS
│   └── pull_request_template.md
├── .claude/commands/
│   └── new-agent.md              # /new-agent — creates an agent persona from template
│
├── agents/                       # One file per agent
│   ├── _template.md              # Copy-and-fill template for new agents
│   ├── developer.md              # Developer agent — implements specs from platform-context
│   ├── pm-oversight.md           # PM agent — tracks progress, reviews specs
│   ├── qa-reviewer.md            # QA agent — reviews PRs for quality, edge cases
│   └── compliance-guardrail.md   # Compliance agent — enforces regulatory rules
│
├── shared/                       # Prompt fragments inherited by ALL agents
│   ├── shared-rules.md           # Org-wide rules: no PII in logs, escalation triggers
│   ├── tone-guide.md             # Communication standards
│   └── context-loading.md        # How to discover and load specs from platform-context
│
└── bootstrap/
    └── github-mcp-bootstrap.md   # THE system prompt template — paste into each Claude Project
```

**Key file: `bootstrap/github-mcp-bootstrap.md`** — this is the only thing that goes into a Claude Project's system prompt. It's a thin loader that tells the agent to read everything else from Git. All substance lives in version-controlled files, not in project settings.

---

### 3. platform-core

**Purpose:** The codebase. C#/.NET implementation of every service spec'd in platform-context.

```
platform-core/
├── CLAUDE.md                     # Build commands, conventions, "read the spec before coding"
├── README.md
├── .gitignore                    # C#/.NET: bin/, obj/, .vs/, coverage/
├── .editorconfig                 # Code style enforcement
├── Directory.Build.props         # Shared MSBuild properties (target framework, nullable)
├── Resolvion.sln                 # Solution file referencing all projects
├── .github/
│   ├── CODEOWNERS
│   └── pull_request_template.md
├── .claude/commands/
│   ├── build.md                  # /build — builds the solution
│   ├── test.md                   # /test — runs tests with coverage
│   ├── review.md                 # /review — runs QA checklist against current changes
│   └── load-spec.md             # /load-spec — pulls the relevant spec from platform-context
│
├── src/
│   ├── Resolvion.Shared/                    # Cross-cutting concerns (all services use this)
│   │   └── CLAUDE.md                        # What belongs here vs. service-specific projects
│   ├── Resolvion.RdnApiClient/              # RDN SOAP client (shared NuGet package)
│   │   └── CLAUDE.md                        # Points to rdn.md Section 7.2
│   ├── Resolvion.Rdn.Domain/               # RDN business logic, fee rules, event routing
│   │   └── CLAUDE.md                        # Points to rdn.md Sections 7.1, 7.3, 7.4
│   ├── Resolvion.Rdn.Infrastructure/        # EF Core, repos, Service Bus publisher
│   │   └── CLAUDE.md                        # Points to rdn.md Section 8
│   ├── Resolvion.Rdn.FirehosePoller/        # Worker Service: polls RDN
│   │   └── CLAUDE.md                        # Contains the polling loop pseudocode
│   ├── Resolvion.Rdn.EventProcessor/        # Worker Service: processes events
│   │   └── CLAUDE.md                        # Lists all 10 handlers and event type mappings
│   ├── Resolvion.Rdn.FeeProcessor/          # Worker Service: fee approval
│   │   └── CLAUDE.md                        # Lists fee rules in evaluation order
│   └── Resolvion.Rdn.Reconciliation/        # Azure Function: daily reconciliation
│       └── CLAUDE.md                        # Reconciliation behavior and schedule
│
└── tests/
    ├── Resolvion.RdnApiClient.Tests/        # Contract tests (WireMock.NET)
    │   └── CLAUDE.md
    ├── Resolvion.Rdn.Domain.Tests/          # Unit tests (fee rules, routing)
    │   └── CLAUDE.md
    ├── Resolvion.Rdn.Integration.Tests/     # SQL + Service Bus integration tests
    │   └── CLAUDE.md
    └── Resolvion.Rdn.E2E.Tests/             # End-to-end with mock RDN
        └── CLAUDE.md
```

**Key design: subdirectory `CLAUDE.md` files.** Claude Code reads the `CLAUDE.md` in whatever directory it's working in. When Claude Code is editing fee rules in `Resolvion.Rdn.Domain/`, it automatically sees the fee rule evaluation order and a pointer to rdn.md Section 7.4 — without loading the entire 44KB spec.

---

### 4. platform-docs

**Purpose:** Operational knowledge for humans and agents: how to set up, how to respond to incidents, why decisions were made, what standards to follow.

```
platform-docs/
├── CLAUDE.md
├── README.md
├── .gitignore
├── .github/
│   ├── CODEOWNERS
│   └── pull_request_template.md
├── .claude/commands/
│   ├── new-runbook.md            # /new-runbook — creates a runbook from template
│   └── new-adr.md                # /new-adr — creates an ADR with next sequence number
│
├── onboarding/
│   ├── agent-setup.md            # How to create a new Claude agent + Project
│   ├── developer-setup.md        # Local dev environment setup
│   └── new-service-guide.md      # End-to-end: spec → code → infra → docs → compliance
│
├── architecture/
│   ├── service-catalog.md        # Living inventory: service, spec, repo, status, owner
│   └── data-flow-patterns.md     # Reusable patterns (poll→archive→publish, etc.)
│
├── adr/
│   ├── _template.md
│   └── 001-context-repo-structure.md
│
├── runbooks/
│   ├── _template.md
│   ├── rdn/                      # RDN-specific runbooks (grows per service)
│   │   ├── firehose-lag.md
│   │   └── dead-letter-triage.md
│   └── shared/                   # Cross-service runbooks
│       ├── key-rotation.md
│       └── service-bus-triage.md
│
└── standards/
    ├── coding-standards.md       # C#/.NET conventions (class size, interfaces, etc.)
    ├── testing-standards.md      # Unit/integration/contract/E2E expectations
    └── pr-review-checklist.md    # What human + agent reviewers check on every PR
```

---

### 5. platform-compliance

**Purpose:** Hard guardrails. Regulatory rules that agents must treat as non-negotiable constraints, not suggestions.

```
platform-compliance/
├── CLAUDE.md
├── README.md
├── .gitignore
├── .github/
│   ├── CODEOWNERS                # Requires compliance team review (2 approvals)
│   └── pull_request_template.md
├── .claude/commands/
│   └── check-service.md          # /check-service — audits compliance coverage for a service
│
├── guardrails/                   # Platform-wide rules ALL agents follow
│   ├── agent-guardrail-rules.md  # Hard rules: never auto-close SCRA, never approve fees on closed
│   ├── data-handling.md          # PII rules, log sanitization, retention
│   └── escalation-triggers.md    # When agents MUST hand off to human
│
├── scra/                         # SCRA-specific documentation
│   ├── scra-overview.md
│   └── scra-hold-rules.md
│
├── fee-compliance/               # Fee rules that constrain automated decisions
│   └── fee-approval-guardrails.md
│
└── service-compliance/           # Per-service compliance (grows with platform-context)
    └── rdn-compliance.md         # RDN-specific: acknowledgment timing, audit requirements
```

---

### 6. platform-infra

**Purpose:** Everything between `git push` and running in Azure. Organized by shared infrastructure + per-service modules.

```
platform-infra/
├── CLAUDE.md
├── README.md
├── .gitignore                    # Infra-specific: .terraform/, secrets, PEM files
├── .github/
│   ├── CODEOWNERS
│   └── pull_request_template.md
├── .claude/commands/
│   ├── validate-bicep.md         # /validate-bicep — runs az bicep build on all templates
│   └── new-service-infra.md      # /new-service-infra — scaffolds infra for a new service
│
├── bicep/
│   ├── main.bicep                # Root template composing all modules
│   ├── modules/
│   │   ├── shared/               # Used by all services
│   │   │   ├── service-bus.bicep
│   │   │   ├── sql-server.bicep
│   │   │   ├── key-vault.bicep
│   │   │   ├── app-insights.bicep
│   │   │   └── container-registry.bicep
│   │   └── rdn/                  # RDN-specific resources
│   │       ├── rdn-service-bus-topics.bicep
│   │       ├── rdn-worker-services.bicep
│   │       └── rdn-function-app.bicep
│   └── parameters/
│       ├── dev.parameters.json
│       ├── staging.parameters.json
│       └── prod.parameters.json
│
├── docker/rdn/
│   ├── Dockerfile.firehose-poller
│   ├── Dockerfile.event-processor
│   └── Dockerfile.fee-processor
│
├── pipelines/                    # GitHub Actions
│   ├── ci.yml                    # Build + test on every PR
│   ├── cd-staging.yml            # Deploy to staging on merge to main
│   └── cd-prod.yml              # Deploy to prod with manual approval
│
├── scripts/
│   ├── shared/
│   │   ├── rotate-keys.sh
│   │   └── replay-dead-letters.sh
│   └── rdn/
│       └── seed-event-types.sql  # Seeds the 42 RDN event types
│
└── environments/
    ├── dev.env                   # Non-secret env vars only
    ├── staging.env
    └── prod.env
```

---

## What's NOT In This Proposal

Things that should be created during development, not upfront:
- `.csproj` files — generated by `dotnet new` when scaffolding begins
- EF Core migrations — created when the data model is implemented
- NuGet package configs — created when the API client is packaged
- `.devcontainer/` — can be added later if the team adopts Codespaces

---

## Next Steps

1. **Review this proposal** — does the structure match your mental model?
2. **Fill in core context** — `core/company-overview.md`, `core/business-rules.md`, `core/decision-principles.md` need real content from your team
3. **Fill in agent personas** — `agents/developer.md` etc. need role-specific instructions
4. **Connect GitHub MCP** — each Claude Project needs the MCP server connected with a PAT
5. **Start building** — open Claude Code in `platform-core`, run `/load-spec`, and begin implementing from `rdn.md`
