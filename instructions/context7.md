# Context7 Tool

Use the `ctx7` CLI to fetch current documentation whenever the user asks about a library, framework, SDK, API, CLI tool, or cloud service — even well-known ones like React, Next.js, Prisma, Express, Tailwind, Django, or Spring Boot. Cover API syntax, configuration, version migration, library-specific debugging, setup instructions, and CLI usage. Prefer ctx7 over web search for library docs, even when you think you know the answer — training data may not reflect recent changes.

## Workflow

1. Resolve the library: `npx ctx7@latest library <name> "<what to look up>"` — use the official name with proper punctuation (e.g., "Next.js" not "nextjs", "Customer.io" not "customerio", "Three.js" not "threejs").
2. Pick the best match (ID format: `/org/project`) by exact name match, description relevance, code snippet count, source reputation (High/Medium preferred), and benchmark score. If results don't fit, try alternate names or rephrased queries.
3. Fetch docs: `npx ctx7@latest docs <libraryId> "<what to look up>"` — one `docs` command per distinct concept, unless the question is about how concepts interact.
4. Answer using the fetched documentation only.

## Rules

- Run `library` first to get a valid ID, unless the user provided one directly in `/org/project` format.
- Keep each query specific and limited to a single concept; combined multi-topic queries dilute ranking and return shallow results.
- Limit to 3 commands per question. For version-specific docs, use `/org/project/version` (e.g., `/vercel/next.js/v14.3.0`).
- Exclude API keys, passwords, and credentials from query text.
- If a command fails with a quota error, inform the user and suggest `npx ctx7@latest login` or setting `CONTEXT7_API_KEY` for higher limits.
- If the docs don't contain the answer, state that explicitly instead of guessing.

## Skip

Refactoring, writing scripts from scratch, debugging business logic, code review, and general programming concepts — answer these directly without ctx7.
