# Mortgage Loan Processing System

> **AI-powered multi-agent mortgage loan processing system — a Red Hat AI Quickstart reference implementation demonstrating supervisor-worker orchestration, human-in-the-loop workflows, and compliance-first design for regulated industries.**

## Project Context

| Attribute | Value |
|-----------|-------|
| Maturity | `mvp` |
| Domain | Fintech — mortgage lending (regulated) |
| Primary Users | AI developers/solutions architects, loan officers, compliance officers |
| Compliance | Fair lending (ECOA, Fair Housing Act) — demonstrated patterns, not certified. Red Hat internal AI compliance. |

### Maturity Expectations

**Important:** Maturity level governs **implementation quality** — test coverage, error handling depth, documentation thoroughness, infrastructure complexity. It does **not** govern **workflow phases**. A PoC still follows the full plan-review-build-verify sequence when SDD criteria are met (see `workflow-patterns` skill). The artifacts may be lighter, but they are not skipped.

| Concern | MVP |
|---------|-----|
| Testing | Happy path + critical edges |
| Error handling | Basic error responses |
| Security | Auth + input validation |
| Documentation | README + API basics |
| Performance | Profile obvious bottlenecks |
| Code review | Light review |
| Infrastructure | Basic CI + single deploy target |

## Goals

1. Demonstrate multi-agent AI patterns (supervisor-worker orchestration, confidence-based escalation, human-in-the-loop) in a regulated industry context
2. Provide a compelling, runnable developer quickstart for the Red Hat AI Quickstart template that showcases real value with mocked external services
3. Maintain complete, immutable audit trails with explainable AI reasoning for every agent decision
4. Show compliance-first design patterns (fair lending, adverse action notices, fraud detection) that translate to production regulated systems

## Non-Goals

- Not a production-certified loan origination system — demonstrates patterns, not regulatory certification
- No end-user authentication (registration, password management, OAuth) — uses API key auth
- No real credit bureau integration — mocked with synthetic data
- No payment processing — application lifecycle ends at approval/denial
- No mobile application — web only (desktop/tablet)
- No multi-tenancy
- No custom ML model training or fine-tuning — uses off-the-shelf LLMs via API
- No real-time collaboration (UI polls for updates, no WebSockets for app status)
- No internationalization — English only, US mortgage regulations only
- No high-availability deployment — basic deployment for demo/dev

## Constraints

- Must build on the Red Hat AI Quickstart template (Turborepo monorepo with React 19, FastAPI, PostgreSQL, Helm charts)
- OpenShift for deployment, Podman for containers, Helm for orchestration
- Agent orchestration must use LangGraph with persistent checkpointing (PostgresSaver)
- Hybrid LLM strategy: Claude for reasoning, GPT-4 Vision for document analysis, optional LlamaStack for local/data-residency
- PostgreSQL + pgvector for both application data and RAG embeddings (no separate vector DB)
- Self-contained quickstart: `make setup && make dev` must get to a working system
- Every agent decision, human action, and workflow transition must produce an immutable audit record

## Stakeholder Preferences

| Preference Area | Observed Pattern |
|-----------------|-----------------|
| Security posture | Upgrade, don't defer. Real API key auth from day one, image redaction before LLMs, separate DB roles from Phase 1, global rate limits before public access. |
| Feature richness | Prefers impressive over minimal. Include fraud detection + denial coaching agents, PDF metadata examination, sentiment analysis. More agents and richer demos preferred. |
| Scope decisions | Prefers industry-standard approaches over simpler custom alternatives. Three roles (not two), cross-session context for authenticated users, expanded FRED data series. |
| Risk tolerance | Conservative on security, ambitious on features. All agent conflicts escalate to human review — no automated tie-breaking. |
| Communication style | Concise and direct. |

## Red Hat AI Compliance

All AI-assisted work in this project must comply with Red Hat's internal AI policies. The full machine-enforceable rules are in `.claude/rules/ai-compliance.md`. Summary of obligations:

1. **Human-in-the-Loop** — All AI-generated code must be reviewed, tested, and validated by a human before merge
2. **Sensitive Data Prohibition** — Never input confidential data, PII, credentials, or internal hostnames into AI tools
3. **AI Marking** — Include `// This project was developed with assistance from AI tools.` (or language equivalent) at the top of AI-assisted files, and use `Assisted-by:` / `Generated-by:` commit trailers
4. **Copyright & Licensing** — Verify generated code doesn't reproduce copyrighted implementations; all dependencies must use [Fedora Allowed Licenses](https://docs.fedoraproject.org/en-US/legal/allowed-licenses/)
5. **Upstream Contributions** — Check upstream project AI policies before contributing AI-generated code; default to disclosure
6. **Security Review** — Treat AI-generated code with the same or higher scrutiny as human-written code, especially for auth, crypto, and input handling

See `docs/ai-compliance-checklist.md` for the developer quick-reference checklist.

## Key Decisions

- **Languages:** TypeScript 5.x (frontend), Python 3.11+ (backend)
- **Runtime:** Node.js (frontend), Python async (backend)
- **Backend:** FastAPI (async)
- **Frontend:** React 19 + Vite + TanStack Router + TanStack Query + Tailwind CSS + shadcn/ui
- **Database:** PostgreSQL + pgvector + SQLAlchemy 2.0 async + Alembic
- **Caching:** Redis
- **Object Storage:** MinIO (S3-compatible)
- **Agent Orchestration:** LangGraph + LangChain with PostgresSaver checkpointing
- **LLM Observability:** LangFuse
- **Testing:** Vitest + React Testing Library (UI), Pytest (API/DB)
- **Package Managers:** pnpm (Node), uv (Python)
- **Build System:** Turborepo
- **Containers:** Podman
- **Deployment:** Helm charts on OpenShift

---

## Agent System

This project uses a multi-agent system with specialized Claude Code agents. The main session handles routing and orchestration using the routing matrix in `.claude/CLAUDE.md`. Each agent has a defined role, model tier, and tool set optimized for its task.

### Quick Reference — "I need to..."

| Need | Agent | Command |
|------|-------|---------|
| Plan a feature or large task | **Main session** | Describe what you need; routing matrix and workflow-patterns skill guide orchestration |
| Shape a product idea into a plan | **Product Manager** | `@product-manager` |
| Gather requirements | **Requirements Analyst** | `@requirements-analyst` |
| Design system architecture | **Architect** | `@architect` |
| Design feature-level implementation approach | **Tech Lead** | `@tech-lead` |
| Break work into epics & stories | **Project Manager** | `@project-manager` |
| Write backend/API code | **Backend Developer** | `@backend-developer` |
| Build UI components | **Frontend Developer** | `@frontend-developer` |
| Design database schema | **Database Engineer** | `@database-engineer` |
| Design API contracts | **API Designer** | `@api-designer` |
| Review code quality | **Code Reviewer** | `@code-reviewer` |
| Write or fix tests | **Test Engineer** | `@test-engineer` |
| Audit security | **Security Engineer** | `@security-engineer` |
| Optimize performance | **Performance Engineer** | `@performance-engineer` |
| Set up CI/CD or infra | **DevOps Engineer** | `@devops-engineer` |
| Debug a problem | **Debug Specialist** | `@debug-specialist` |
| Write documentation | **Technical Writer** | `@technical-writer` |

### How It Works

1. **Describe what you need** — for non-trivial tasks, the main session uses the routing matrix and workflow-patterns skill to select agents and sequence work.
2. **Use a specialist directly** when you know exactly which agent you need (e.g., `@backend-developer`).
3. **Rules files** enforce project conventions automatically — global rules are imported below, and path-scoped rules (API, UI, database development) load automatically for matching files.
4. **Spec-Driven Development** is the default for non-trivial features — plan review before code review, machine-verifiable exit conditions, and anti-rubber-stamping governance.
5. **Skills** provide workflow templates and project convention references.

## Project Conventions

@.claude/rules/ai-compliance.md
@.claude/rules/code-style.md
@.claude/rules/git-workflow.md
@.claude/rules/testing.md
@.claude/rules/security.md
@.claude/rules/error-handling.md
@.claude/rules/observability.md
@.claude/rules/api-conventions.md
@.claude/rules/agent-workflow.md
@.claude/rules/review-governance.md
@.claude/rules/architecture.md
@.claude/rules/domain.md

## Project Commands

<!-- Uncomment and fill in the actual commands for your project. -->
<!-- The defaults below assume a Makefile-based workflow with Turborepo. -->
<!-- This is the canonical location for project commands. architecture.md -->
<!-- cross-references this section rather than duplicating it. -->

```bash
# make setup              # Install all dependencies (Node + Python)
# make build              # Build all packages
# make dev                # Start all dev servers (UI + API)
# make test               # Run tests across all packages
# make lint               # Run linters across all packages
# pnpm type-check         # TypeScript type checking
# make db-start           # Start database container
# make db-upgrade         # Run database migrations
# make containers-build   # Build all container images
# make containers-up      # Start all services via compose
```
