---
name: aconfmgr
description: >-
  Maintain the user's aconfmgr configuration — the declarative Arch Linux
  package and file management repo at ~/.config/aconfmgr. Use this skill
  whenever the task involves editing aconfmgr config scripts (*.sh), running
  aconfmgr save/check/diff, sorting 99-unsorted.sh entries, managing
  packages/files/links via aconfmgr helper functions, or performing a
  maintenance cycle on the aconfmgr repo.

  Only the four verbs save, apply, check, and diff exist — there are no init,
  track, show, or list subcommands. Never invent commands that don't exist.
  The agent never runs aconfmgr apply — that is reserved for the user.
---

# aconfmgr Skill

Your duty is to **maintain the user's aconfmgr configuration** located at ~/.config/aconfmgr (or wherever $ACONFMGR_CONFIG points). Every task you handle serves this single purpose: keeping the declarative system config accurate, organized, and version-controlled.

## Hard Constraint

You never run `aconfmgr apply`. Applying configuration changes to the live system is the only task reserved for the user. You may run `aconfmgr save`, `aconfmgr check`, and `aconfmgr diff` as needed to inspect and organize the config. When the user is ready, they apply changes themselves.

## CLI Reference

| Action | Description |
| --- | --- |
| `aconfmgr save` | Update the config directory to reflect current system state; appends new entries to 99-unsorted.sh. Idempotent. Key options: `-c DIR`, `--aur-helper HELPER`. |
| `aconfmgr apply` | Update the system to reflect the config dir (installs/removes packages, installs/restores/deletes files, sets file properties). Does **NOT** upgrade packages. **Forbidden — user only.** Options: `--yes`, `--paranoid`, `--skip-checksums`. |
| `aconfmgr check` | Syntax-check/lint the config; catches unused `files/` entries and 99-unsorted.sh ordering problems. |
| `aconfmgr diff /path...` | Diff between the config's version of a file and the system's version (direction: system → config). |

Also note the environment/options: `$ACONFMGR_CONFIG`, `-c/--config DIR`, `--skip-config`, `--skip-inspection`, `--color`, `-v`.

## Environment

- The agent's shell is bash. The user's interactive shell is **fish**. Any command handed to the user for copying must use fish syntax: `(cmd)` command substitution, `(cmd | psub)` instead of bash's `<()` process substitution, no bash-isms. The agent may run bash in its own shell.

## Config File Model

- Plain numbered bash scripts sourced in lexicographic order (10-*.sh → 99-unsorted.sh).
- `files/` subdirectory mirrors the filesystem root (e.g. `files/etc/pacman.conf`). Default mode 644, owner/group root/root.
- All helper functions are shell functions injected from /usr/lib/aconfmgr/src/helpers.bash before sourcing user scripts.

## Helper Functions

| Function | Defaults / Notes |
| --- | --- |
| `AddPackage [--foreign] PKG...` | Mark package explicitly installed; unlisted packages are removed on apply. |
| `RemovePackage [--foreign] PKG...` | Unmark a package as explicitly installed. |
| `IgnorePackage [--foreign] PKG...` | Mark a package as ignored (never removed). |
| `AddPackageGroup GROUP` | Mark an entire package group as installed. |
| `CopyFile /PATH [MODE [OWNER [GROUP]]]` | Copies `files/PATH` into output; default mode 644, owner/group root. |
| `CopyFileTo SRC DST [MODE [OWNER [GROUP]]]` | Source and destination paths differ. |
| `CreateFile [--no-clobber] /PATH [MODE [OWNER [GROUP]]]` | Creates empty file, prints absolute path for redirects. |
| `GetPackageOriginalFile [--no-clobber] PKG /PATH` | Extracts original file from package archive. |
| `CreateLink /PATH TARGET [OWNER [GROUP]]` | Symlink (used for systemd `.wants/` links). |
| `CreateDir /PATH [MODE [OWNER [GROUP]]]` | Creates a directory. |
| `RemoveFile /PATH` | Removes a file from the managed set. |
| `SetFileProperty /PATH TYPE VALUE` | TYPE ∈ {owner, group, mode, deleted}; `deleted y` deletes a package-owned file; empty string resets to default. |
| `IgnorePath PATTERN` | Shell glob pattern skipped during stray-file detection; **always quote the pattern** (unquoted globs expand in the shell). |
| `AddFileContentFilter PATTERN FUNCTION` | Normalize volatile file content. |

## Repo Style

When making changes, match the user's existing conventions observed from git history and file structure:

- **Commit messages**: imperative mood, capitalized, no period. No Conventional Commits prefix. Short subjects (≤50 chars). Examples: `Add Prettier`, `Configure SSH server`, `Move Caddy to a better category`, `Replace Helix with Neovim`, `Fix missing login screen`, `Add a SQL linter`, `Add package \`youtube-music\``.
- **Commit granularity**: one commit per related package group plus its configs — e.g. `Add Node.js and npm`, `Add Gradle and JDK 21`, `Add Zathura` (zathura + zathura-pdf-mupdf). A `CopyFile`/`CreateLink`/`SetFileProperty` declaration must be committed together with its `files/<path>` copy. Keep truly independent concerns separate (e.g. a package replacement like `Replace llama-cpp with llama-cpp-git`); do not batch unrelated changes.
- **Comments**: single-line `# ...` above the line they explain; wrap tool/command names in backticks (e.g. `` `systemd-tmpfiles` ``); state the reason (hardening, auto-unlock), not the tracking rationale. Example: `# \`systemd-tmpfiles\` resets / to 555 on every boot (/usr/lib/tmpfiles.d/root.conf), hardening against unprivileged writes to the root directory.`
- **File naming**: all config scripts use the `10-` prefix: `10-ignored.sh`, `10-packages.sh`, `10-misc.sh`, `10-systemd.sh`, `10-refind.sh`.
- **Section headers**: `# || Category Name.` for top-level categories, `# Subcategory.` for sub-sections.
- **Formatting**: config scripts are formatted with `shfmt -ln bash -w -s *.sh`. Package entries: `AddPackage pkgname  # Short description`, where the description is copied verbatim from 99-unsorted.sh (aconfmgr derives it from `pacman -Qi <pkg>`).
- **No .gitignore**: the repo has no .gitignore. `99-unsorted.sh` is never committed — it is a live drift indicator, always deleted after sorting its contents into the numbered files.

## Maintenance Workflow

When asked to do aconfmgr maintenance, follow this exact sequence:

1. Navigate to the config directory: `cd ~/.config/aconfmgr` (or `$ACONFMGR_CONFIG` if set).
2. **Check for a dirty working tree**: `git status`. If uncommitted changes exist (modified files, new untracked files in `files/`, a populated 99-unsorted.sh, etc.): **STOP**. Inform the user of the dirty state, summarize what changed (list the modified/new files), and ask how they want to proceed. Do not do further work on a dirty tree. If clean, continue.
3. **Create a session branch** prefixed `ai/session_<date>`: `git checkout -b ai/session_2026-08-01`. The branch is per maintenance session, not per feature. Within this branch, make commits one per related package group / logical change, matching the user's commit style. The user reviews the session's work as one unit.
4. Run `aconfmgr save` to capture current system drift into 99-unsorted.sh and `files/`.
5. **Triage the output**: read 99-unsorted.sh and any new entries in `files/`; sort entries into the appropriate numbered scripts (10-packages.sh, 10-systemd.sh, etc.); add `IgnorePath` rules in 10-ignored.sh for volatile paths; delete 99-unsorted.sh (or empty it). For `.pacnew` files left by package upgrades, and for tracking files/dirs not previously managed (new `CopyFile`/`CreateDir` entries), confirm with the user before committing — they may prefer the file deleted on apply, ignored, or not tracked at all.
6. **Validate**: `aconfmgr check`. Fix any warnings before committing.
7. **Commit by related group**: one commit per related package group plus its configs, matching the user's commit message style (see Repo Style). Stage the `files/` copies together with their `CopyFile` declarations.
8. **When in doubt, ask the user**: if unsure where to place an entry, what to ignore, or whether a config decision is correct — pause and ask. Never guess on config decisions.

## Additional Workflows

- **Bootstrap a New Machine**: install aconfmgr-git (`paru -S aconfmgr-git` or `yay`), clone the config repo to ~/.config/aconfmgr (or set `$ACONFMGR_CONFIG`), run `aconfmgr apply` to reproduce the system (user runs apply).
- **Editing Managed Files Inline**: `CreateFile` + heredoc (`cat > "$(CreateFile /etc/fstab)" << 'EOF' ... EOF`); `GetPackageOriginalFile pkg /etc/file` + `sed`/`cat >>` to patch package-owned files; `AddFileContentFilter` for volatile content (timestamps).
- **Debugging**: `bash -x aconfmgr save` gives a full trace. Error handler prints a stack trace. Low `/tmp` disk → `IgnorePath` the large paths or set `$TMPDIR`.
- **Multi-Machine Setup**: git branch per machine or hostname conditionals; `CopyFileTo` for per-host file variants.

## Common Pitfalls

1. **Unquoted IgnorePath patterns** — shell globs must be quoted: `IgnorePath '/etc/*shadow*'` not `IgnorePath /etc/*shadow*`.
2. **99-unsorted.sh drift** — `aconfmgr check` warns when files sorted later shadow declarations in 99-unsorted.sh. Delete 99-unsorted.sh after triage.
3. **Optional dependencies are pruned** — any optionally-depended-upon package must be listed with `AddPackage` or it is removed on apply.
4. **apply does not upgrade** — run `sudo pacman -Syu` separately; apply only reconciles the package set.
5. **Don't run under sudo** — aconfmgr elevates internally; running as root is a no-op.
6. **Working on a dirty tree** — always check `git status` first. Uncommitted drift from a prior `aconfmgr save` means the config is out of sync; triage and commit (or ask the user) before continuing.
7. **Package-owned file "drift"** — `save` flags a package-owned config as new/changed when it no longer matches the package original (e.g. a PAM file edited for KDE Wallet auto-unlock). Detect it natively with `pacman -Qkk <pkg>` (detailed check; `pacman -Qii <pkg>` lists backup files, marking modified ones `[modified]`), then show the actual customization with `paccat <pkg> <path> | diff <path> -` (`paccat` from the `extra` repo prints the original file from the package, downloading it if not cached).

## Guardrails

- Never run `aconfmgr apply`. The user applies changes to the live system.
- Never begin work on a dirty working tree — check `git status` first.
- Always branch from a clean `main` with the `ai/session_<date>` naming convention (one branch per maintenance session).
- Make commits one per related package group plus its configs. Match the user's commit style (imperative, capitalized, short subjects like `Add <thing>`).
- When unsure about a config decision, pause and ask the user. In particular, ask before tracking new files/dirs (new `CopyFile`/`CreateDir`/`CreateLink` entries) — the user may want them ignored or removed instead. Label assumptions as assumptions — never present a guess as fact.
- Prefer the four documented verbs. If unsure whether a subcommand exists, consult `aconfmgr --help` or the man page.
- After editing config, run `aconfmgr check` before committing.

## Final Report

At the end of a maintenance session, give the user a clear report covering:

1. **What changed**: which files were modified (e.g. "Added 3 packages to `10-packages.sh` under `# || Development.`, created a new IgnorePath rule in `10-ignored.sh` for `/var/cache/foo`").
2. **Where the work lives**: the session branch name (e.g. `ai/session_2026-08-01`) and how many commits it contains.
3. **How the user proceeds** — manual steps to verify and apply: review the branch (`git log main..ai/session_<date>` and `git diff main..ai/session_<date>`); if satisfied, `git checkout ai/session_<date>` and `aconfmgr check`; then apply to the system (the user does this, not the agent): `aconfmgr apply`; verify the system works; cherry-pick individual commits or merge the branch into `main` (see Git Cheatsheet).
4. **Any open questions**: things flagged as uncertain that need the user's input.

## Git Cheatsheet

Common operations the user may need after an agent session:

- Review the session work: `git log main..ai/session_<date>` and `git diff main..ai/session_<date>`
- Cherry-pick a single commit into main: `git checkout main`; `git cherry-pick <commit-sha>`
- Cherry-pick a range of commits: `git cherry-pick <oldest-sha>^..<newest-sha>`
- Move a commit from the session branch to main, then remove from session branch: `git checkout main`; `git cherry-pick <commit-sha>`; `git checkout ai/session_<date>`; `git reset --hard HEAD~1`
- Move a range of commits (rebase --onto): `git rebase --onto main <oldest-sha>^ ai/session_<date>` (replays commits from `<oldest-sha>` through tip of the session branch onto main)
- Abort a cherry-pick in progress: `git cherry-pick --abort`
- Undo a completed cherry-pick: `git revert <cherry-picked-commit-sha>`
- Delete the session branch after merging: `git branch -d ai/session_<date>` (use `-D` to force-delete without checking merge status)

### Tracking which session commits are already on main

- `git log --cherry-pick`/patch-id detection is **order-dependent**: the patch-id includes hunk context lines, so a commit cherry-picked onto main in a different order produces a different patch-id and won't be recognized as applied. Only reliable when commits are picked in session order.
- Order-independent list of session commits not yet on main (by subject, fish syntax):

```fish
set applied (git log --format=%s main)
git log --reverse --format='%h %s' ai/session_<date> | while read -l h s
    if not contains -- $s $applied
        echo $h $s
    end
end
```

### Editing commits on the session branch

- To edit all commits of the branch in place, rebase onto its fork point (lists every commit, no conflicts):
  `git rebase -i (git merge-base main HEAD)`
- Do **not** rebase the session branch onto a moved `main` — it replays commits already cherry-picked there and conflicts.
