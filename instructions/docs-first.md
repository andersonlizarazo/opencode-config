# Docs First, Web Last

When the task involves a library, framework, SDK, API, CLI tool, cloud service,
or "how do I use X" question — even well-known ones like React, Next.js,
Prisma, Express, Tailwind, Django, or Spring Boot — load the `find-docs`
skill with the `skill` tool and follow its workflow. Do this FIRST, before
any `webfetch` or `websearch`.

Training data is frequently outdated. `find-docs` fetches current docs via
the Context7 CLI (`npx ctx7@latest library` / `docs`) and is the canonical
source for API syntax, configuration options, version migration, and
library-specific debugging.

## Order of operations

1. Load the `find-docs` skill: `skill` tool with name `find-docs`, then run
   the ctx7 commands it specifies.
2. Use `Grep_MCP_searchGitHub` for real-world code examples and usage patterns
   of the API in question (it searches literal code, not keywords).
3. Only if `find-docs` fails or the question has no answer, fall back to
   `websearch`/`webfetch`.

## Rules

- Never reach for `webfetch`/`websearch` first on a docs/library question.
  Verify against current docs instead of relying on training data.
- One concept per `docs` query; keep queries specific.
- If ctx7 returns a quota error, inform the user and suggest
  `npx ctx7@latest login` or setting `CONTEXT7_API_KEY`.
- If the docs don't contain the answer, say so explicitly — do not guess.
- This rule does not apply to general programming concepts, refactoring,
  code review, or business logic — answer those directly.
