---
status: DRAFT
version: 0.1
created: 2026-07-10
last_updated: 2026-07-10
author: Andrew Crabb
approved_by:
approved_date:
task_file: ../../../tasks/tool-1-role-based-copy-tasks.md
---

# Tool 1 — PRD: Role-Based Copy

**Feature Name**: Role-Based Copy
**Product**: Agentic Design Patterns — Companion Tool Suite (Tool 1 of N)
**Status**: Draft
**Feature ID**: tool-1
**Owner**: Andrew Crabb
**Engineering Lead**: TBD

> **Note**: This PRD is a **living document** — it can be updated during
> implementation as requirements become clearer. Git history is the audit
> trail. See "Open Questions" for assumptions that need confirmation.

---

## Executive Summary

### Feature Overview

Role-Based Copy is an agentic tool that generates role-tailored copies of the
"Agentic Design Patterns" book content. Given a chapter (or the whole book)
and a target reader role — Engineer, Product Manager, Researcher, or
Executive — it produces a rewritten copy of that content adapted to the
role's vocabulary, depth, and priorities, while preserving the source
material's technical accuracy and structure.

### Business Value

- The book explicitly targets "engineers, researchers, and product managers,"
  but every reader receives the same 424-page text. Role-tailored copies let
  each audience get to their relevant material faster.
- Serves as a working, dogfooded demonstration of the book's own patterns
  (Routing, Prompt Chaining, Parallelization, Reflection) applied to a real
  content pipeline — the tool is itself a teaching artifact for the repo.
- First tool in a planned companion suite; establishes the conventions
  (naming, PRD flow, directory layout, agent pipeline shape) that later tools
  will reuse.

### Implementation Complexity

**Effort**: Medium
**Risk**: Low — read-only over source content; output is new files, nothing destructive
**Dependencies**: LLM API access (Claude), repository chapter files as input corpus

---

## Context & Problem Statement

### Current State

The repository contains the full book as ~40 markdown chapter and appendix
files organized into four parts plus front matter. Content is one-size-fits-
all: an executive skimming for strategic implications reads the same prose as
an engineer looking for implementation detail.

### Problem to Solve

Different reader roles need different "copies" of the same underlying
material:

| Role | What they need from a chapter |
|------|------------------------------|
| Engineer | Code examples, implementation trade-offs, failure modes, API specifics |
| Product Manager | Capability framing, when-to-use guidance, cost/effort implications |
| Researcher | Pattern provenance, references, open problems, comparative analysis |
| Executive | One-page strategic summary, risk posture, business impact |

Manually producing and maintaining four variants of 40+ documents is
infeasible; each source edit would require four manual rewrites.

### User Impact

**Who**: Readers of the repo across all four roles; the repo maintainer.
**Pain Point**: Non-engineer readers wade through code-heavy chapters;
engineers wade through narrative framing. Maintainers cannot hand-maintain
variants.
**Frequency**: Every reading session; every upstream content update.

### Impact of Not Implementing

The repo remains a single-audience artifact and the companion tool suite has
no established pattern to build on.

---

## User Stories & Scenarios

### Primary User Story

**As a** product manager reading the book,
**I want to** generate a PM-tailored copy of a chapter,
**So that** I can understand the pattern's product implications without
parsing implementation code.

**Acceptance Criteria**:
- Given a chapter file and role `pm`,
- When I run the tool,
- Then a new markdown file is produced containing the chapter rewritten for a
  PM audience, with code blocks summarized into capability statements, and
  the original file left untouched.

### User Story 2 — Batch generation

**As the** repo maintainer,
**I want to** regenerate all role copies for all chapters in one command,
**So that** role variants stay in sync after upstream content edits.

**Acceptance Criteria**:
- Given the full chapter set and one or more roles,
- When I run the tool in batch mode,
- Then role copies are generated in parallel per chapter, written under a
  predictable output directory, and a summary report lists successes and
  failures.

### User Story 3 — Fidelity check

**As a** reader of a role copy,
**I want** every role copy to link back to its source chapter and declare its
generation date and role,
**So that** I can always verify claims against the canonical text.

**Acceptance Criteria**:
- Given any generated copy,
- When I open it,
- Then a metadata header identifies the source file, target role, generator
  version, and generation date.

### Usage Scenarios

#### Scenario 1: Single chapter, single role
**Context**: A PM wants Chapter 7 (Multi-Agent Collaboration) before a roadmap discussion.
**User Goal**: Get a PM-view copy of one chapter.
**Steps**:
1. User runs `role-copy --role pm 01-Part_One/Chapter_7-*.md`
2. The tool routes the chapter through the PM rewrite pipeline
3. User receives `copies/pm/Chapter_7-Multi-Agent_Collaboration.md`

**Expected Outcome**: A faithful, PM-voiced copy with code summarized, in
under a few minutes.

#### Scenario 2: Full regeneration after upstream edit
**Context**: Maintainer merges a fix to Chapter 12.
**User Goal**: Refresh only the stale copies.
**Steps**:
1. Maintainer runs `role-copy --all-roles --changed-since <ref>`
2. The tool detects Chapter 12 changed, regenerates its four role copies in parallel
3. Report confirms four files updated, none failed

**Expected Outcome**: Copies never drift silently from source.

---

## Functional Requirements

### Core Functionality

#### FR-1: Role-targeted rewrite
**Description**: Given one source markdown file and one role from the
supported set {engineer, pm, researcher, executive}, produce a complete
markdown copy adapted to that role.
**Priority**: P0
**User Story**: Story 1
**Acceptance Criteria**:
- AC-1.1: Output preserves the source's section ordering and heading hierarchy.
- AC-1.2: No factual/technical claims are introduced that are absent from the source.
- AC-1.3: Role adaptations follow the per-role style contract (see Business Logic).
- AC-1.4: Source files are never modified.

#### FR-2: Batch mode with parallel execution
**Description**: Accept a glob/part/whole-book scope and one or more roles;
fan out chapter×role jobs in parallel (Parallelization pattern, Chapter 3)
and aggregate results into a run report.
**Priority**: P0
**User Story**: Story 2
**Acceptance Criteria**:
- AC-2.1: A failed chapter×role job does not abort the batch.
- AC-2.2: The run report lists every job with status and output path.

#### FR-3: Provenance metadata
**Description**: Every generated copy begins with YAML frontmatter:
`source`, `role`, `generated`, `generator_version`, `source_commit`.
**Priority**: P0
**User Story**: Story 3
**Acceptance Criteria**:
- AC-3.1: Frontmatter is machine-parseable and present in 100% of outputs.

#### FR-4: Reflection pass (quality gate)
**Description**: After the rewrite, a second agent pass (Reflection pattern,
Chapter 4) compares the copy against the source and flags hallucinated
claims, dropped sections, or role-style violations; failures trigger one
automatic revision cycle before reporting.
**Priority**: P1
**Acceptance Criteria**:
- AC-4.1: Copies failing reflection after one revision are marked
  `status: needs-review` in frontmatter and listed in the run report.

#### FR-5: Staleness detection
**Description**: Compare `source_commit` in existing copies against current
source; regenerate only stale copies when `--changed-since` is used.
**Priority**: P2

### Business Logic — Per-Role Style Contracts

#### Rule 1: Engineer copy
**Condition**: role = engineer
**Action**: Retain all code blocks verbatim; tighten narrative prose;
foreground trade-offs, failure modes, and implementation notes.
**Example**: Chapter 5's function-calling walkthrough keeps full code, drops
extended analogies.

#### Rule 2: PM copy
**Condition**: role = pm
**Action**: Replace code blocks with 1–3 sentence capability summaries;
foreground when-to-use criteria, effort/cost signals, and user-facing impact.

#### Rule 3: Researcher copy
**Condition**: role = researcher
**Action**: Retain references and comparative analysis; foreground open
problems and pattern lineage; keep code as illustrative excerpts.

#### Rule 4: Executive copy
**Condition**: role = executive
**Action**: Compress to ~1 page per chapter; strategic implications, risk
posture, and investment framing only; no code.

### Data Requirements

#### Data Element: Source chapters
**Source**: Repo markdown files under `00-Introduction/` … `05-Appendix/`
**Format**: GitHub-flavored markdown, some with embedded images/code fences
**Validation**: File must parse as UTF-8 markdown; image refs passed through unchanged
**Usage**: Read-only input corpus

#### Data Element: Generated copies
**Source**: Tool output
**Format**: Markdown with YAML frontmatter, written to `copies/<role>/<original-filename>`
**Validation**: Frontmatter schema (FR-3); reflection verdict (FR-4)

---

## Non-Functional Requirements

### Performance
- **Throughput**: Full book × 4 roles completes in ≤ 30 minutes with parallel fan-out.
- **Single job**: One chapter × one role in ≤ 3 minutes.

### Reliability
- **Error Rate**: ≤ 2% of jobs may end `needs-review`; zero silent failures.
- **Recovery**: Batch runs are resumable; completed jobs are not re-run.

### Security
- Read-only over source; no network calls other than the LLM API; API keys
  via environment variable, never committed.

### Internationalization
- Out of scope for v1 (English source only).

---

## Success Metrics

#### Metric 1: Fidelity
**Current Baseline**: n/a
**Target**: 100% of shipped copies pass the reflection gate (no hallucinated claims).
**Measurement**: Reflection pass verdicts in run reports.

#### Metric 2: Coverage
**Target**: All 21 pattern chapters available in all 4 roles.
**Measurement**: Count of generated files vs. chapter×role matrix.

#### Metric 3: Freshness
**Target**: Zero copies older than their source for more than one release cycle.
**Measurement**: `source_commit` drift check (FR-5).

---

## Technical Specifications Summary

### High-Level Architecture

The pipeline is deliberately built from the book's own patterns:

```
CLI (chapter scope + roles)
    ↓
Router (Ch. 2) — selects per-role style contract
    ↓
Parallel fan-out (Ch. 3) — one job per chapter×role
    ↓
Prompt chain (Ch. 1) — outline → rewrite → format
    ↓
Reflection gate (Ch. 4) — fidelity check vs. source, one revision cycle
    ↓
Writer — copies/<role>/<file>.md + run report
```

### Technology Stack
- **Runtime**: Python 3.12+ CLI (`uv`-managed), or Claude Code skill wrapper — TBD
- **LLM**: Claude API (latest available model tier for rewrite; smaller tier acceptable for reflection)
- **Storage**: Plain files in-repo under `copies/` (no database)

---

## Task Breakdown

### Phase 1: Foundation
- [ ] Define per-role style contracts as prompt templates
- [ ] Single-chapter, single-role pipeline (FR-1) with provenance frontmatter (FR-3)
- [ ] Golden-file test on one representative chapter per part

**Definition of Done**: One chapter renders correctly in all four roles.

### Phase 2: Scale & Quality
- [ ] Batch mode with parallel fan-out and run report (FR-2)
- [ ] Reflection quality gate with one auto-revision cycle (FR-4)

**Definition of Done**: Full-book batch completes within performance targets.

### Phase 3: Maintenance loop
- [ ] Staleness detection via `--changed-since` (FR-5)
- [ ] README section documenting the tool and linking role copies from the TOC

**Definition of Done**: Regeneration after an upstream edit is a one-command operation.

---

## Constraints & Assumptions

### Technical Constraints
- Source files use long Google-Docs-derived filenames with special characters; output naming must handle them safely.
- Book content is copyrighted (Gulli & Sauco); generated copies stay within this repo under its existing LICENSE terms.

### Assumptions
- A-1: "Copy" means *a role-adapted copy of the book's content* (copywriting variant), not file-copy-with-permissions. **Needs confirmation — see Open Questions.**
- A-2: The four roles above are the right initial set.
- A-3: Output lives in-repo under `copies/` rather than a separate site/build artifact.

---

## Risks & Mitigations

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| Rewrites drift from source meaning (hallucination) | Medium | High | Reflection gate (FR-4); provenance links for spot-checking |
| Repo size balloons (40 files × 4 roles) | High | Low | Copies are plain text; optionally gitignore and treat as build artifact (Open Question 3) |
| Upstream book edits outpace regeneration | Medium | Medium | Staleness detection (FR-5); one-command refresh |
| Role copies misread as canonical text | Low | Medium | Prominent "generated copy" banner + source link in every file |

---

## Out of Scope

- Translation / multi-language output
- Audio or slide-deck renditions
- Per-user personalization beyond the four fixed roles
- Editing or restructuring the canonical source chapters

---

## Appendix

### A. Glossary
- **Role copy**: A generated, role-adapted markdown rendition of a source chapter.
- **Style contract**: The per-role rewrite rules (Business Logic Rules 1–4).
- **Reflection gate**: Post-generation agent pass that validates fidelity to source.

### B. References
- Chapter 1 — Prompt Chaining; Chapter 2 — Routing; Chapter 3 — Parallelization; Chapter 4 — Reflection (this repo)
- README.md — audience statement ("engineers, researchers, product managers")

### C. Open Questions
- [ ] Q1 — **Interpretation**: This PRD assumes "role-based copy" = role-tailored content copies of the book. If it instead means role-based *marketing copy*, or role/permission-based file copying, the scope sections need rewriting. Confirm before Phase 1.
- [ ] Q2 — Should Executive be replaced/augmented by other roles (e.g., Educator, Student)?
- [ ] Q3 — Are generated copies committed to the repo, or gitignored build artifacts published elsewhere (e.g., GitHub Pages)?
- [ ] Q4 — Python CLI vs. Claude Code skill (or both) as the delivery vehicle?

---

## Approval

| Role | Name | Approval Date | Status |
|------|------|---------------|--------|
| Product Manager | Andrew Crabb | — | Pending |
| Engineering Lead | TBD | — | Pending |
| Maintainer | Tom Mathews | — | Pending |
