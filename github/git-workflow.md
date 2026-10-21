# Git Workflow

How we work in this repo: **one branch per task, merged into `main` through a GitHub pull request (PR).**
Follow this file for every task unless the user says otherwise.

## Rules

1. Never commit directly to `main`. All work happens on a task branch.
2. One task = one branch = one PR.
3. Keep `main` linear and clean: rebase task branches onto `main`; never rebase `main` or any branch someone else has pulled.
4. Never use plain `git push --force`. Use `--force-with-lease`, and only on your own task branch.
5. Never use `git reset --hard`, `git clean -f`, or `git push --force*` without first checking `git status` and confirming nothing uncommitted would be lost.
6. Before every commit, review exactly what is staged (`git diff --staged`).
7. Before destructive or hard-to-reverse actions (force-push, deleting a remote branch, merging a PR), state what you are about to do and get confirmation.

## Conventions

- **Branch name:** `task/<number>-<short-kebab-name>`, e.g. `task/02-add-login-form`.
- **Commit message:** `task <number>: <imperative summary>`, e.g. `task 02: add login form validation`. Keep the subject under ~72 characters; add a body when the "why" is not obvious.
- **Commits:** small and focused. One logical change per commit. Do not mix refactors with feature work.
- **Staging:** prefer naming files (`git add path/to/file`). Use `git add -A` only after checking `git status` so you do not commit stray files, secrets, or build output.
- **PRs:** one task per PR, a clear title, and a short description of what changed and why.

## Workflow

### 1. Start a task

```bash
git switch main
git pull --ff-only                        # latest main from GitHub; fails instead of creating a surprise merge
git switch -c task/02-short-name          # create the task branch and switch to it
```

### 2. Work and commit

```bash
git status                                # what changed
git diff                                  # unstaged changes
git add path/to/file                      # stage specific files
git diff --staged                         # review what you are about to commit
git commit -m "task 02: implement X"
```

Commit early and often; you can tidy up before opening the PR.

### 3. First push

```bash
git push -u origin task/02-short-name     # -u links the branch to its GitHub counterpart
```

After that, a plain `git push` works on this branch. To get this behavior automatically for every new branch:

```bash
git config --global push.autoSetupRemote true
```

### 4. Update your branch with new `main` commits (rebase)

```bash
git fetch origin                          # download remote changes without touching your branches
git rebase origin/main                    # replay your commits on top of the latest main
```

If the rebase stops on a conflict:

```bash
git status                                # lists conflicted files
# edit each file and remove the <<<<<<< ======= >>>>>>> markers
git add <file>
git rebase --continue
# or abandon the rebase and return to the pre-rebase state:
git rebase --abort
```

A rebase rewrites history, so the next push must be a force-push, with the safety check:

```bash
git push --force-with-lease               # refuses if someone else pushed to the branch in the meantime
```

### 5. Open a PR

```bash
gh pr create --base main --fill           # --fill uses commit messages as the title and body
gh pr view --web                          # open it in the browser
gh pr status                              # check its state and checks
```

### 6. Merge

Preferred: through the PR.

```bash
gh pr merge --merge --delete-branch       # or --squash / --rebase
```

Fallback, locally without a PR:

```bash
git switch main
git pull --ff-only
git merge --no-ff task/02-short-name      # --no-ff keeps an explicit merge commit
git push
```

### 7. Clean up

```bash
git switch main
git pull --ff-only                        # bring the merge into your local main
git branch -d task/02-short-name          # delete the local branch
git push origin --delete task/02-short-name   # skip if gh already deleted the remote branch
git fetch --prune                         # remove stale origin/* references
```

Note: after a **squash** or **rebase** merge, `git branch -d` may refuse because the original commits are not in `main`. If the PR is confirmed merged, use `git branch -D task/02-short-name`.

## Rebase vs. merge when updating a task branch

Rebase your task branch onto `main`. Only you work on it, so rewriting its history is safe, and the result is a clean, straight history. Never rebase `main` itself, or any branch other people have pulled; merge those instead.

## Inspecting state

```bash
git log --oneline --graph --all -15       # picture of all branches
git log --oneline main..HEAD              # commits on your branch that main lacks
git log --oneline HEAD..origin/main       # commits on main you have not rebased onto yet
git branch -vv                            # each branch, its upstream, and ahead/behind counts
```

## Troubleshooting

| Problem | Fix |
| --- | --- |
| `fatal: The current branch has no upstream branch` | `git push -u origin <branch>` (see step 3) |
| Need to switch branches with uncommitted work | `git stash`, switch, then `git stash pop` |
| Rebase went wrong | `git rebase --abort` while it is in progress |
| Rebase already finished but the result is bad | `git reflog`, find the entry from before the rebase, then `git reset --hard HEAD@{N}` (this discards uncommitted changes; stash first) |
| Push rejected after a rebase | Expected. Use `git push --force-with-lease` on your task branch |
| Push rejected on `main` | Do not force it. Something is wrong; stop and ask |
| Committed to `main` by mistake (not yet pushed) | `git switch -c task/NN-name` to keep the commits on a new branch, then reset local `main` to `origin/main` |

## Quick checklist

- [ ] Branched from an up-to-date `main`
- [ ] Branch named `task/NN-short-name`
- [ ] Commits are small, focused, and prefixed `task NN:`
- [ ] Reviewed `git diff --staged` before each commit
- [ ] Rebased onto `origin/main` before opening the PR
- [ ] PR opened against `main`, merged, then local and remote branches cleaned up
