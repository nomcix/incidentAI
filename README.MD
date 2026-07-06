# AI Incident Intelligence Platform

Production-style GenAI application for incident investigation. Engineers ask operational questions and get cited answers from runbooks, postmortems, architecture notes, tickets, logs, and service metadata.

## Problem

Incident responders waste time searching scattered sources during production issues.

## Goal

Build an incident assistant that:

- answers operational questions with cited sources
- refuses when evidence is missing
- measures quality with evals
- runs on AWS with production-style security and observability

## Non-goals

- Not a customer-facing production system
- Not a replacement for human incident commanders
- Not an autonomous remediation bot
- Not a research ML project

## Architecture

| Layer | Stack | Role |
|-------|-------|------|
| Production API | .NET / ASP.NET Core | Auth, RAG orchestration, Bedrock integration, audit logging |
| AI / eval tooling | Python | Datasets, golden-question evals, retrieval analysis, reports |
| Infrastructure | Terraform | AWS deployment |

.NET owns the synchronous request path. Python owns offline eval and data workflows. The stacks connect through HTTP APIs, JSONL files, and versioned schemas.

See [docs/adr/](docs/adr/) for decision records.

## Status

Early development. MVP targets a .NET API with Bedrock RAG, synthetic incident corpus, and a Python eval runner.

## Documentation

- [ADR 0001: .NET and Python Service Boundary](docs/adr/0001-dotnet-python-boundary.md)
- [ADR 0002: AI Assistance Policy](docs/adr/0002-ai-assistance-policy.md)
