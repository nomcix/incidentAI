# ADR 0002: AI Assistance Policy

**Status:** Accepted  
**Date:** 2026-07-05

## Context

This project uses AI coding and content tools during development. That is useful for speed on repetitive work, but it creates quality and credibility risks if generated output becomes source of truth without review.

Eval datasets, prompts, and architecture choices must be defensible in code review, in production, and in discussion. Core design must stay understandable and debuggable without relying on unreviewed generated code.

This ADR defines where AI assistance is allowed and where human ownership is required.

## Decision

### AI may draft

- Synthetic incident documents, runbooks, tickets, and logs
- Documentation, ADR drafts, and README sections
- Prompt templates and prompt variants for experimentation
- Candidate golden questions, expected sources, and required facts
- Small examples, test fixtures, and boilerplate scaffolding

AI output is treated as a **draft**. It is not committed as authoritative project material until reviewed.

### Human ownership is required

- Review and edit all AI-generated artifacts before they enter the repo
- Validate schema shape, factual consistency, and fit with project goals
- Approve final versions of prompts, datasets, docs, and eval labels
- Own architecture decisions and record them in ADRs

### AI may not

- Generate the full codebase with minimal human implementation
- Make final architecture or boundary decisions
- Mark eval data as golden without human verification
- Bypass tests, security checks, or contract validation

### Golden question rule

A question is not golden until a human has verified:

- the question is realistic and answerable from project sources
- expected sources and required facts are correct
- refusal cases are labeled intentionally, not assumed

Until verified, candidates live outside `datasets/golden/` or carry a non-golden status in workflow docs.

## Alternatives

**No AI assistance.** Slower iteration and less exposure to how teams actually use AI tools today. Rejected for a project that should reflect current engineering practice.

**Unrestricted AI generation.** Fast early progress, but weak ownership signal, higher risk of incorrect eval labels, and harder debugging when the author did not shape core design. Rejected.

**AI writes code, human only reviews.** Tempting for speed, but hides implementation learning and makes production debugging harder. Rejected.

## Consequences

**Positive**

- Faster drafting of synthetic data, docs, and eval candidates
- Clear line between AI-assisted drafts and human-approved artifacts
- Golden sets and prompts remain trustworthy enough to score regressions
- Easier for reviewers to see engineered software with AI as a tool, not the author

**Negative**

- More manual review work than pure AI generation
- Slower than accepting first-pass model output
- Requires discipline to reject convenient but unverified AI suggestions

## Risks

| Risk | Mitigation |
|------|------------|
| Schema-valid but wrong golden labels | Human verification before promotion to `datasets/golden/` |
| Prompt or doc drift from actual behavior | Review against code and contracts before merge |
| Over-reliance on AI for core logic | Implement auth, RAG orchestration, and eval scoring by hand |
| Plausible-sounding synthetic docs that mislead retrieval | Spot-check sources against chunking and indexing assumptions |
| Reviewers cannot verify human ownership | Keep ADRs, tests, and key paths clearly human-designed |

## When to revisit

- Team size grows and review workflow needs formal checklists or CI gates
- AI tooling becomes part of the product itself, not just development
- Eval volume makes manual golden verification a bottleneck

Any policy change that weakens human verification for labels or architecture should be a new ADR.
