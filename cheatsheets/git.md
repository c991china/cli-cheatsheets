# git cheatsheet

git 2.43. `git switch`/`git restore` over `checkout` where possible.

## Start / clone

```bash
git clone <url>                      # full clone
git clone --depth 1 <url>            # shallow, fast, no history
git clone --branch v2.1 <url>        # specific tag/branch
git init                             # new repo in current dir
```

## Status / log

```bash
git status -sb                       # short + branch name
git log --oneline --graph --decorate -20
git log --pretty=format:'%h %ad %an %s' --date=short
git log -p -- path/to/file           # history of one file, with diffs
git log -S 'def foo' --oneline       # commits that added/removed a string
git log -G 'regex' --oneline         # like -S but regex
git log --author='ann' --since='2 weeks ago'
git show <sha> --stat
git blame -L 40,60 file.py
git shortlog -sn --no-merges         # commit counts per author
```

## Staging / commit

```bash
git add -p                           # stage hunks interactively
git add -u                           # stage tracked changes only
git commit -m "msg"
git commit --amend -m "new msg"      # rewrite last commit (local only!)
git commit --amend --no-edit         # add staged files to last commit
git commit --fixup <sha>             # for autosquash later
```

## Undo (read before running)

```bash
git restore file                     # discard working changes to file
git restore --staged file            # unstage, keep changes
git restore -p file                  # discard hunks interactively
git reset --soft HEAD~1              # undo commit, keep staged
git reset HEAD~1                     # undo commit, keep unstaged
git reset --hard HEAD~1              # undo commit AND DISCARD CHANGES (destructive)
git revert <sha>                     # new commit that undoes <sha> (safe on shared)
git clean -nd                        # dry run: what would git clean remove
git clean -fd                        # remove untracked files/dirs (destructive)
```

## Branch

```bash
git switch -c feat/x                 # create + switch
git switch main
git switch -                         # previous branch
git branch -vv                       # local branches + upstream + ahead/behind
git branch -a                        # include remotes
git branch -m old new                # rename
git branch -d feat/x                 # delete (refuses if unmerged)
git branch -D feat/x                 # force delete (destructive)
git push origin --delete feat/x      # delete remote branch
git branch --merged main             # branches merged into main
```

## Sync

```bash
git fetch --all --prune              # update refs, drop deleted remotes
git pull --rebase                    # fetch + rebase local commits on top
git push -u origin feat/x            # first push, sets upstream
git push                             # after -u
git push --force-with-lease          # safer force push; refuses if remote moved
```

`--force-with-lease` over `--force`. It won't clobber commits you haven't seen.

## Rebase / history

```bash
git rebase origin/main               # replay current branch onto main
git rebase -i origin/main            # squash/reword/reorder
git rebase --continue                # after resolving conflicts
git rebase --abort
git rebase --onto main old-base feat # move a range of commits
git cherry-pick <sha>                # apply one commit here
git cherry-pick <a>..<b>             # apply a range
```

## Stash

```bash
git stash push -m "wip auth"         # named stash
git stash -u                         # include untracked files
git stash list
git stash pop                        # apply + drop
git stash apply stash@{2}            # apply, keep it
git stash show -p stash@{0}          # view diff
git stash drop stash@{0}
```

## Tags

```bash
git tag -a v1.0.0 -m "release 1.0.0"
git tag                              # list
git push origin v1.0.0               # tags are NOT pushed by default
git push --tags
git tag -d v1.0.0                    # delete local
git push origin --delete v1.0.0      # delete remote
```

## Recovery

```bash
git reflog                           # where HEAD has been; the safety net
git switch -c rescue HEAD@{5}        # resurrect from a reflog entry
git fsck --lost-found                # find dangling commits/blobs
git reset --hard <sha>               # move branch back to a known-good sha
```

## Worktrees (multiple checkouts at once)

```bash
git worktree add ../proj-hotfix hotfix
git worktree list
git worktree remove ../proj-hotfix
```

Handy when you need to review a branch without stashing your current work.

## Diff

```bash
git diff                             # unstaged
git diff --staged                    # staged
git diff main..feat                  # between branches
git diff --stat HEAD~3
git diff --word-diff                 # word-level, good for prose
git diff --no-index a.txt b.txt      # diff two files outside a repo
```

## Config

```bash
git config --global user.name "Ann"
git config --global pull.rebase true
git config --global rerere.enabled true
git config --global core.autocrlf input   # mac/linux; 'true' on Windows
git config --list --show-origin
```

## Notes

- `git switch -c` replaces `checkout -b`; `git restore` replaces the file/undo
  half of `checkout`. Fewer ways to shoot yourself.
- `.gitignore` doesn't untrack files already committed: `git rm --cached f`.
- `rerere` remembers conflict resolutions. Weird name, real time saver on long
  rebases.
