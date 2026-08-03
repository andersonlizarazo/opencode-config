# Scripts

## `opencode`

A `fish` wrapper around OpenCode that sandboxes the agent with `bwrap`
and auto-enables `--auto`.

Ideally, link this script into your `PATH` so you can run `opencode`
from anywhere:

```fish
ln -s "$HOME/.config/opencode/scripts/opencode" "$HOME/.local/bin/opencode"
```

Note that the wrapper execs the real binary at `/usr/bin/opencode`, so
shadowing the command name on `PATH` does not cause recursion.

### Environment variables

| Variable | Purpose |
|----------|---------|
| `BWRAP` | Set to `0` to skip the sandbox; the wrapper then strips the `--auto` flag and runs `opencode` unsandboxed (auto-approval disabled). |
| `OPENCODE_ENABLE_EXA` | Always set to `1` by the wrapper to enable the Exa search tool in the sandbox. |
