# Domain Docs

How the engineering skills should consume this repo's domain documentation when exploring the codebase.

## Before exploring, read these

- `CONTEXT.md` at the repo root, if present
- `docs/adr/` when it exists and is relevant to the area being worked on

If these files do not exist, proceed without blocking or prompting. Do not create them proactively unless a later skill specifically needs them.

## File structure

This repo uses a single-context layout by default:

```text
/
├── CONTEXT.md
├── docs/adr/
├── flowerp/
├── web/
├── tests/
└── eval/
```

## Vocabulary and decisions

When output names a domain concept or task, prefer the terms used in the project docs and existing code. If a concept is missing from the repo docs, do not invent a new naming scheme without checking the surrounding domain model first.

If a planned change conflicts with an existing ADR, surface that conflict explicitly instead of silently ignoring it.
