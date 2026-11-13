# Project Memory Bank

Store concise, durable project context here for future contributors and agents.

## Required baseline

Keep the memory bank grounded in verifiable repository evidence. At minimum, document:

- Product overview and the source files that demonstrate its behavior.
- Technology stack: languages, frameworks, infrastructure/tooling, and key dependencies.
- Current status: what works, known gaps, and pending verification supported by evidence.

The current baseline is in [project-overview.md](./project-overview.md). Refresh its status and citations when implementation or validation results change.

## Conventions

- Use descriptive kebab-case Markdown filenames, such as `architecture.md` or `development-workflow.md`.
- Record verified repository facts, decisions, and their source paths.
- Distinguish current behavior from proposed changes, and update or remove notes when the code changes.
- Do not present inferred recommendations as an approved roadmap. Include priorities only when a repository document or explicit user decision establishes them; otherwise label them as unprioritized observations or omit them.
- Keep operational instructions in `.agents/rules/`; use this directory for supporting context, not task logs.
