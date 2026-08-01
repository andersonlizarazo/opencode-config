# Shell Non-Interactive Strategy

OpenCode's shell has no TTY. Commands that wait for user input or open
an editor/pager will hang until timeout. Use non-interactive flags, pipes,
and OpenCode tools instead.

## Rules

1. Use OpenCode tools (Read, Write, Edit) for file operations — prefer them
   over shell equivalents (sed, cat, echo).
2. Supply non-interactive flags on every command: `-y`, `--yes`, `-f`,
   `--force`, `--no-edit`, `--no-pager`.
3. Use `git commit -m "msg"` (not bare `git commit`) and
   `git --no-pager log` (not bare `git log`).
4. Use `python -c "code"` or `python script.py` — never a bare REPL.
5. For commands without a dedicated flag, use `yes | command` or a heredoc.

<uncertainty>
If a command's non-interactive mode is undocumented, check --help or web
docs before running it. Wrap uncertain commands in `timeout 30 command`.
Never guess a flag — verify first.
</uncertainty>

## Environment Variables (auto-set)

| Variable | Value | Purpose |
|----------|-------|---------|
| `CI` | `true` | General CI detection |
| `DEBIAN_FRONTEND` | `noninteractive` | Apt/dpkg prompts |
| `GIT_TERMINAL_PROMPT` | `0` | Git auth prompts |
| `GIT_EDITOR` | `true` | Block git editor |
| `GIT_PAGER` | `cat` | Disable git pager |
| `PAGER` | `cat` | Disable system pager |
| `GCM_INTERACTIVE` | `never` | Git credential manager |
| `HOMEBREW_NO_AUTO_UPDATE` | `1` | Homebrew updates |
| `npm_config_yes` | `true` | NPM prompts |
| `PIP_NO_INPUT` | `1` | Pip prompts |
| `YARN_ENABLE_IMMUTABLE_INSTALLS` | `false` | Yarn lockfile |

## Command Reference

### Package Managers

| Tool | Avoid | Use |
|------|-------|-----|
| **NPM** | `npm init` | `npm init -y` |
| **NPM** | `npm install` | `npm install --yes` |
| **Yarn** | `yarn install` | `yarn install --non-interactive` |
| **PNPM** | `pnpm install` | `pnpm install --reporter=silent` |
| **Bun** | `bun init` | `bun init -y` |
| **APT** | `apt-get install pkg` | `apt-get install -y pkg` |
| **APT** | `apt-get upgrade` | `apt-get upgrade -y` |
| **PIP** | `pip install pkg` | `pip install --no-input pkg` |
| **Homebrew** | `brew install pkg` | `HOMEBREW_NO_AUTO_UPDATE=1 brew install pkg` |

### Git

| Action | Avoid | Use |
|--------|-------|-----|
| Commit | `git commit` | `git commit -m "msg"` |
| Merge | `git merge branch` | `git merge --no-edit branch` |
| Pull | `git pull` | `git pull --no-edit` |
| Rebase | `git rebase -i` | `git rebase` |
| Add | `git add -p` | `git add <file>` |
| Log | `git log` | `git --no-pager log -n 10` |
| Diff | `git diff` | `git --no-pager diff` |

### File Operations

| Action | Avoid | Use |
|--------|-------|-----|
| Remove | `rm file` | `rm -f file` |
| Copy | `cp -i a b` | `cp -f a b` |
| Move | `mv -i a b` | `mv -f a b` |
| Unzip | `unzip file.zip` | `unzip -o file.zip` |
| SSH | `ssh host` | `ssh -o BatchMode=yes -o StrictHostKeyChecking=no host` |
| SCP | `scp file host:` | `scp -o BatchMode=yes file host:` |
| Curl | `curl url` | `curl -fsSL url` |
| Wget | `wget url` | `wget -q url` |

### Docker

| Action | Avoid | Use |
|--------|-------|-----|
| Run | `docker run -it image` | `docker run image` |
| Exec | `docker exec -it container bash` | `docker exec container cmd` |
| Build | `docker build .` | `docker build --progress=plain .` |
| Compose | `docker-compose up` | `docker-compose up -d` |

### REPLs

| Tool | Avoid | Use |
|------|-------|-----|
| Python | `python` | `python -c "code"` or `python script.py` |
| Node | `node` | `node -e "code"` or `node script.js` |
| IPython | `ipython` | `python -c` |

## Prompt Handling Fallbacks

When a command lacks a non-interactive flag:

```bash
yes | ./install_script.sh
```

```bash
./configure.sh <<EOF
option1
option2
EOF
```

```bash
echo "input" | sudo -S command
```

```bash
timeout 30 ./uncertain_script.sh || echo "Timed out"
```
