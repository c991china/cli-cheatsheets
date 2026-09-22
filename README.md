# cli-cheatsheets

Terse command references. Not tutorials. If you want the story behind `git
rebase`, read the book; this is for when you know what you want and forgot the
flag.

I built these because most "cheatsheets" online are either padded with beginner
explanations or wrong. These are the commands I actually type, grouped by task,
with a one-line note only when the command isn't self-explanatory.

## Sheets

| Sheet | Covers |
|-------|--------|
| [git.md](cheatsheets/git.md) | branch, undo, rebase, log searching, worktrees |
| [docker.md](cheatsheets/docker.md) | run/exec/logs, cleanup, images, compose |
| [kubectl.md](cheatsheets/kubectl.md) | get/describe/logs, exec, rollout, debug |
| [aws-cli.md](cheatsheets/aws-cli.md) | s3, ec2, iam, logs, sts, profiles |
| [vim.md](cheatsheets/vim.md) | motions, edits, search, windows, macros |
| [bash.md](cheatsheets/bash.md) | variables, tests, loops, redirection, trap |
| [curl.md](cheatsheets/curl.md) | requests, headers, auth, files, debugging |
| [tmux.md](cheatsheets/tmux.md) | sessions, windows, panes, copy mode, config |
| [postgres.md](cheatsheets/postgres.md) | psql meta-commands, common SQL, ops |
| [ffmpeg.md](cheatsheets/ffmpeg.md) | convert, trim, scale, audio, subtitles |

## How to use these

- Command-first. Anything with an obvious name gets no prose.
- `<...>` is a placeholder you replace. `[...]` is optional.
- Blocks are grouped under a task heading; skim to the heading you need.
- Where a command is destructive I say so, because I've seen `docker system
  prune -a` eat someone's weekend.

## Grep them

Once you clone this, ripgrep is the fastest way to find the flag you forgot:

```bash
rg -n 'force-with-lease' cheatsheets/
rg -n 'pg_blocking_pids' cheatsheets/
```

Or add an alias that searches all of them:

```bash
cs() { rg -i --no-heading "$*" ~/cli-cheatsheets/cheatsheets/; }
```

## Conventions

- Targets the tools as of early 2026: git 2.43, docker 25, kubectl 1.29,
  aws-cli v2, tmux 3.4, ffmpeg 6.x. Flags change; if something 404s, `--help`
  is still the ground truth.
- Linux/macOS focused. Windows users: most of this works in WSL or Git Bash,
  but paths and some flags differ. I don't pretend otherwise.
- I don't cover the GUI or "best practices". These are commands.

## Notes

- Everything here is something I ran at least once. But I make no promise it's
  right for YOUR system. The ones that delete data are marked.
- PRs welcome if a command is wrong for your version; say which version.

## License

MIT. See LICENSE.
