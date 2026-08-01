# Source Code Fetch

Prefer local codebase exploration (`grep`, `glob`, `read`) over web access.
Use the web only as a last resort.

## Reading git-forge source

When source code lives on a git forge (GitHub, GitLab, Gitea, ...), do not
issue `webfetch`/`websearch` calls per file. Clone the repository once into
the system tmp dir and read it locally with the normal file tools:

```
git clone --depth 1 --filter=tree:0 <url> /tmp/opencode/<repo-name>
```

This blobless shallow clone skips commit history; blobs are fetched on demand
by the file tools. Use `--depth 1`, not `--depth 0` — git rejects the latter
with `fatal: depth 0 is not a positive number`.

If you do not know the repository URL, use the docs tools first: the
`find-docs` skill locates the project and its repo URL (e.g. via the Context7
CLI). Search results/`websearch` may also surface the canonical URL — use it
to clone rather than fetching files individually.

## JavaScript compiled/minified bundles

Do not `grep`/`rg` single-line minified JS bundles or .map-less dist files;
matching against huge minified lines is slow and burns CPU. Clone the
repository and read the readable source (`src/`, pre-dist) instead.

## Unobtainable source

If the source genuinely cannot be obtained (no repo URL, unreadable bundle,
build artifacts only) and reading it is necessary to do the task correctly,
stop and inform the user: state exactly what is missing and ask how to
proceed. Never fake an answer from an unreadable artifact.
