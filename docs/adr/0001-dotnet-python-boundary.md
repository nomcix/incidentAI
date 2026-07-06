# ADR 0001: .NET and Python Service Boundary

**Status:** Accepted  
**Date:** 2026-07-05

## Context

This project is a production-style GenAI incident assistant: cited answers, refusal when evidence is missing, evals, and AWS-native deployment. It should look like real software (API, auth, observability, failure handling), not a notebook demo.

That work splits naturally into two layers. The deployed application (API, auth, RAG orchestration, Bedrock, logging, deployment) fits a .NET backend stack. Offline AI work (datasets, evals, chunking experiments, cost reports) is usually scripted in Python. This ADR records where each layer lives and how they connect.

## Decision

**.NET (ASP.NET Core)** runs the production app and the full request path.

**.Python** runs offline tooling: dataset validation, eval runners, retrieval analysis, prompt regression, reports, and CLI scripts.

They talk through explicit contracts (HTTP API, JSON bodies, JSONL files, versioned schemas), not shared libraries.

### Request path (.NET only for MVP)

```
Client → ASP.NET Core → auth/validation → RAG orchestration → retriever/KB → Bedrock → response + citations
```

Python is not in the synchronous production path for the MVP.

### Eval path (Python calling .NET)

```
Developer → Python CLI → golden_questions.jsonl → call API → score → eval_results.jsonl → report
```

### Boundary rules

1. **Production behavior → .NET.** Endpoints, auth, Bedrock calls, RAG orchestration, tool policy, audit logs, error handling.
2. **Offline / eval / data work → Python.** JSONL validation, golden sets, eval suites, citation scoring, chunking experiments, cost/latency reports.
3. **No Python in the request path unless there's a strong reason.** If that changes, write a new ADR.
4. **Prefer explicit mechanics in the MVP.** LangChain/LlamaIndex/etc. can come later; prompts, retrieval, citations, and scoring should stay visible in code early on.
5. **Version shared contracts.** Both sides should include something like `"schema_version": "golden-question.v1"` and document breaking changes.

Shared artifacts live in places like `datasets/golden/`, `datasets/eval-runs/`, `prompts/prompt-manifest.json`, and documented API shapes. Neither stack should depend on the other's internals.

## Alternatives

**All .NET:** Simpler stack, but eval and data tooling are awkward to build and iterate on in C# alone.

**All Python:** Common in tutorials, but weakens the production API, auth, and ops requirements this project targets.

**.NET + Python microservice in the request path:** Useful eventually, but too much ops and debugging surface for MVP. A Python service belongs in a later ADR if needed.

**Notebooks for AI work:** Fine for private exploration; durable eval tooling should be scripts, tests, and versioned data.

## Consequences

**Positive:** Production behavior stays in C#/AWS/ASP.NET Core. Python handles real eval and data workflows. Matches how many teams split application code from AI tooling.

**Tradeoffs:** Two languages means duplicated models, schema drift, more CI, and more README. Mitigate with OpenAPI or `docs/api-contracts.md`, versioned JSONL, contract tests (at least one test that Python can parse the .NET API response), and ADRs when the boundary moves.

**Deploy note:** Normal traffic should not require Python. Python runs locally, in CI, or in scheduled jobs unless a future ADR says otherwise.

**Security:** .NET owns runtime auth, tenant access, tool authorization, audit logs, and log redaction. Python evals use synthetic or sanitized data, not production secrets in JSONL files.

## When to revisit

- A Python-only library becomes necessary for production retrieval/reranking
- Eval tooling needs to become a hosted service
- A dedicated Python service is needed for retrieval, reranking, or internal AI tooling
- Tool orchestration outgrows what makes sense in .NET

Any of those requires a new ADR, not a quiet exception.
