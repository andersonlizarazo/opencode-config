# VibeGuard Secret Masking

The `opencode-vibeguard` plugin is active: sensitive strings are replaced with
placeholders before any request reaches the LLM provider. You never see
plaintext secrets.

## Placeholder format

`__MASKED_SECRET_<CATEGORY>_<hash12>__` — `hash12` is the first 12 hex chars
of HMAC-SHA256(session secret, original): stable within a session,
irreversible.

## What gets masked

Every text and reasoning part of every message, plus tool call inputs and
outputs (current and historical). Matching is driven by keywords, regexes, and
builtin rules configured for the plugin.

## Rules

1. Treat placeholders as opaque. Never decode, reverse, or guess the value.
2. Never fabricate the underlying value. If you need a secret and only have a
   placeholder, use the placeholder.
3. Tools run with real values automatically before execution
   (`bash`/`read`/`write`/`edit`), so files you write contain real values, not
   placeholders.
4. Don't echo secrets or placeholders in responses unless necessary.
5. If you can't identify a value, say so rather than inventing one.
