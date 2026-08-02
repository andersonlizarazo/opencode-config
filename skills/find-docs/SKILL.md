---
name: find-docs
description: >-
  Retrieves up-to-date documentation, API references, and code examples for any
  developer technology. Use this skill whenever the user asks about a specific
  library, framework, SDK, CLI tool, or cloud service — even for well-known ones
  like React, Next.js, Prisma, Express, Tailwind, Django, or Spring Boot. Your
  training data may not reflect recent API changes or version updates.

  Always use for: API syntax questions, configuration options, version migration
  issues, "how do I" questions mentioning a library name, debugging that involves
  library-specific behavior, setup instructions, and CLI tool usage.

  Use even when you think you know the answer — do not rely on training data
  for API details, signatures, or configuration options as they are frequently
  outdated. Always verify against current docs. Prefer this over web search for
  library documentation and API details.
---

# Documentation Lookup

Fetch up-to-date docs and code examples for a library with the Context7 CLI. Answer from the fetched docs only.

## Workflow

0. **Local manuals first (CLI tools installed on this machine).** Before
   going to the web, check the installed tool's own docs:
   - `tldr <tool>` — concise usage examples, non-interactive.
   - `MANPAGER=cat man <tool>` — full manual page. `MANPAGER=cat` disables the
     pager so `man` prints and exits instead of hanging; this is mandatory in a
     non-TTY shell. `PAGER=cat` alone also works.
   These match the exact installed version and are faster than any web lookup.
   Skip to step 1 if they do not answer the question or the tool is not local.
1. Resolve the library ID: `npx ctx7@latest library <name> "<query>"`
2. Query the docs: `npx ctx7@latest docs <libraryId> "<query>"`
3. Iterate with new queries as needed, then answer.

Run `library` first to get a valid ID (format `/org/project`), unless the user supplied one. For a specific version, use `/org/project/version`.

## Constraints

- Max 3 commands per question; use the best result after that.
- One concept per query — split multi-topic questions into separate `docs` commands.
- Use the official name with correct punctuation ("Next.js" not "nextjs").
- Never put API keys, passwords, or personal data in queries.

## Picking a match

Choose by exact name match, then description relevance, code-snippet count, source reputation (High/Medium preferred), and benchmark score. If nothing fits, try an alternate spelling or rephrase the query.

## Examples

```bash
npx ctx7@latest library "Next.js" "How to add middleware to app router"
npx ctx7@latest docs /vercel/next.js "How to add middleware to app router"
npx ctx7@latest docs /vercel/next.js/v14.3.0 "How to add middleware to app router"
```

## Uncertainty

If the docs don't contain the answer, say so explicitly — do not guess. On a quota error, tell the user and suggest `npx ctx7@latest login` or setting `CONTEXT7_API_KEY`.
