# Technical Skills 3 — Developer Tooling (Git and GitHub)

This covers the "Developer Tooling" line on my resume. Git questions are almost guaranteed, and they are easy marks if you can explain the *model*, not just the commands.

---

## 1. What is Git, and what is version control?
**Version control** records changes to files over time so you can recall any version, see who changed what and why, and work in parallel without overwriting each other.

**Git** is a **distributed** version control system: every clone is a complete repository with the full history, not a thin checkout of a central server. That is the key architectural fact, and everything else follows from it — you can commit, branch, diff, and view history entirely offline, and there is no single point of failure.

**Git vs GitHub**: Git is the tool that runs on your machine. GitHub is a hosting service built around Git that adds collaboration features — pull requests, issues, code review, CI/CD via Actions, and access control. GitLab and Bitbucket are alternatives. Saying "Git is the engine, GitHub is one of the places you can park it" is a clean way to answer.

**Centralised (SVN) vs distributed (Git)**: SVN has one authoritative server; you need it to commit or see history, and a server outage stops work. Git gives everyone a full copy, so commits are local and instant, branching is cheap, and the "central" repository is a convention rather than a requirement.

## 2. The three areas — the mental model that answers most questions
```
Working Directory  --git add-->  Staging Area (Index)  --git commit-->  Local Repository  --git push-->  Remote
       ^                                                                        |
       +---------------------------- git checkout / restore --------------------+
```
- **Working directory** — your actual files on disk.
- **Staging area (index)** — a scratch pad where you assemble exactly what the next commit will contain. This is what lets you commit part of your changes.
- **Local repository** (`.git`) — the committed history, stored as objects.
- **Remote** — the shared copy on GitHub.

**Why does the staging area exist?** So a commit can be a deliberate, coherent unit rather than "everything I happened to have open". You can stage one file, or with `git add -p` even one hunk within a file.

## 3. How Git actually stores data (asked to separate the memorisers from the understanders)
Git is a content-addressable object store with four object types:
- **blob** — the contents of a file (no name, no metadata).
- **tree** — a directory listing: names, modes, and the blob or tree each maps to.
- **commit** — a snapshot pointer: one root tree, one or more parent commits, author, committer, timestamp, and message.
- **tag** — an annotated pointer to a commit.

Every object is named by the SHA-1 (now moving to SHA-256) hash of its contents, which is why history is tamper-evident: changing any old commit changes its hash, which changes every descendant's hash.

Crucially, **Git stores snapshots, not diffs**. If a file is unchanged between commits, the new tree simply points at the same existing blob. Diffs are computed on demand for display.

A **branch is just a movable pointer to a commit** — a 41-byte file. That is why branching in Git is instant and why people branch freely. `HEAD` is a pointer to the branch you are currently on (or directly to a commit, which is "detached HEAD").

## 4. Everyday commands
```bash
git init                        # start a repository
git clone <url>                 # copy a remote repository including full history
git status                      # what is modified, staged, untracked
git add <file> / git add .      # stage changes
git add -p                      # stage selected hunks interactively
git commit -m "message"         # record the staged snapshot
git commit --amend              # rewrite the most recent commit
git log --oneline --graph --all # readable history
git diff                        # working directory vs index
git diff --staged               # index vs last commit
git show <hash>                 # what one commit changed
git branch                      # list branches
git switch -c feature/x         # create and switch (modern form of checkout -b)
git merge feature/x             # merge a branch into the current one
git rebase main                 # replay my commits on top of main
git fetch                       # download remote changes without applying them
git pull                        # fetch + merge (or --rebase)
git push origin main            # upload commits
git stash / git stash pop       # shelve work temporarily
git restore <file>              # discard working-directory changes
git reset --soft|--mixed|--hard # move the branch pointer, optionally discarding
git revert <hash>               # a new commit that undoes an old one
git cherry-pick <hash>          # apply one commit from elsewhere onto here
git tag -a v1.0 -m "release"    # mark a release
git blame <file>                # who last changed each line
git bisect start                # binary search history for the commit that broke something
```

## 5. Merge vs rebase — the classic question
**Merge** creates a new commit with two parents, joining the histories. It is non-destructive, preserves exactly what happened, and shows the branch structure — but produces a history with many merge commits that can be hard to read.

**Rebase** takes your commits and replays them one at a time on top of the target branch, creating **new commits with new hashes**. The result is a clean, linear history that reads as though you had started from the latest main.

**The golden rule of rebasing**: never rebase commits that have been pushed and that others may have based work on. Rewriting shared history forces everyone else into a painful recovery. Rebase your own local feature branch before merging; merge into shared branches.

Practical policy: `git pull --rebase` to update your feature branch, and a merge (or squash merge) to land it.

## 6. Resolving a merge conflict
A conflict happens when two branches change the same lines of the same file, and Git cannot decide which wins.
```
<<<<<<< HEAD
the version on my current branch
=======
the version from the branch being merged
>>>>>>> feature/x
```
Steps: `git status` to list conflicted files, open each and edit to the correct final content, delete the conflict markers, `git add` the resolved file, then `git commit` (or `git rebase --continue`). `git merge --abort` backs out entirely. Reduce conflicts by pulling frequently, keeping branches short-lived, and keeping changes focused.

## 7. Undoing things — pick the right tool
| Situation | Command |
|---|---|
| Discard uncommitted changes to a file | `git restore <file>` |
| Unstage a file but keep the edits | `git restore --staged <file>` |
| Fix the last commit message or add a forgotten file | `git commit --amend` |
| Undo a commit but keep the changes staged | `git reset --soft HEAD~1` |
| Undo a commit and unstage the changes | `git reset --mixed HEAD~1` (default) |
| Undo a commit and destroy the changes | `git reset --hard HEAD~1` |
| Undo a commit that is already pushed | `git revert <hash>` — safe, because it adds a new commit rather than rewriting history |
| Recover something you think you destroyed | `git reflog` — it records every position HEAD has held, usually for 90 days |

`reset` versus `revert` is the most common trap: **reset rewrites history, revert adds to it.** On a shared branch you use revert.

## 8. Branching strategies
- **Git Flow** — long-lived `main` and `develop`, plus `feature/`, `release/`, and `hotfix/` branches. Thorough but heavy; suits scheduled releases.
- **GitHub Flow** — one `main` that is always deployable; every change is a short-lived branch merged through a pull request. Simple, suits continuous deployment.
- **Trunk-based development** — everyone commits to `main` frequently behind feature flags; requires strong automated testing.

**How I use branches**: my NetSpecter repository has `main` plus `cli` and `frontend-backend`, separating the stable CLI from the in-progress web dashboard work. My ModelAuth repository uses branches as *experiment tiers* — `main` holds the unified three-tier benchmark, `easy` holds the cross-architecture substitution experiments, and `medium+hard` holds the scale and quantisation experiments. That is a slightly unusual use of branches and it is worth explaining honestly: it keeps each experiment's data and results reproducible in isolation, though for pure code a directory or a tag would be more conventional.

## 9. Pull requests and code review
A **pull request** proposes merging one branch into another, and is where review, discussion, and automated checks happen before code lands. A good PR is small, has a clear title and description of *why*, links the issue it addresses, and passes CI. As a reviewer, look for correctness, edge cases, tests, naming, and whether it is doing more than one thing.

**Merge strategies on GitHub**: *Merge commit* (preserves every commit plus a merge commit), *Squash and merge* (collapses the branch into one clean commit on main — the most common for feature branches), *Rebase and merge* (replays commits linearly with no merge commit).

## 10. .gitignore and what should never be committed
`.gitignore` lists path patterns Git should not track. Never commit: secrets and API keys, `.env` files, `node_modules/` and other installable dependencies, build outputs (`dist/`, `__pycache__/`, `*.pyc`, `.o` files), IDE settings, OS junk (`.DS_Store`), and large binary data.

**If a secret is committed, changing it in a later commit is not enough** — it is still in history and must be treated as compromised. Rotate the credential immediately, then optionally scrub history with `git filter-repo` or BFG and force-push, coordinating with everyone who has a clone. My ClassRoom Code repository ships a `.env.example` with placeholder values and gitignores the real `.env`, which is the correct pattern.

## 11. Other GitHub features worth naming
- **Issues** for bug and task tracking, with labels, milestones, and assignees.
- **GitHub Actions** for CI/CD — YAML workflows in `.github/workflows/` triggered by push, PR, schedule, or manual dispatch; used to run tests, lint, build, and deploy.
- **Releases and tags** for versioned artefacts, usually with semantic versioning (MAJOR.MINOR.PATCH — breaking, feature, fix).
- **Forks** for contributing to a repository you cannot push to: fork, branch, commit, push to your fork, open a PR upstream.
- **README, LICENSE, CONTRIBUTING** — the documentation that makes a repository usable. Every one of my repositories has a substantial README, and that is deliberate: a project nobody can run is a project nobody can evaluate.
- **Protected branches** requiring reviews and passing checks before merge.
- **SSH keys vs HTTPS tokens** for authentication; SSH keys are the convenient default for a machine you control.

## 12. Rapid-fire Git questions
- **`git fetch` vs `git pull`?** `fetch` downloads remote changes into your remote-tracking branches without touching your working directory — safe, and lets you inspect first. `pull` is `fetch` followed by `merge` (or `rebase` with `--rebase`).
- **`git reset` vs `git checkout`/`restore`?** `reset` moves the branch pointer and optionally the index and working tree; `restore` only changes files.
- **What is `HEAD`?** A pointer to the current branch, and through it to the current commit. `HEAD~1` is one commit back; `HEAD^` is the first parent.
- **What is a detached HEAD?** HEAD points directly at a commit rather than a branch. Commits made there are unreachable once you leave, unless you create a branch.
- **What is `origin`?** The conventional name for the default remote — nothing special about it, just a default alias.
- **What is `git stash` for?** Temporarily shelving uncommitted work so you can switch context, then `git stash pop` to bring it back.
- **What is cherry-pick for?** Applying a single commit from another branch — useful for hotfixes that must go into both a release branch and main.
- **What is `git bisect`?** A binary search over history: you mark a known-good and known-bad commit, and Git checks out midpoints for you to test until it identifies the first bad commit. Enormously faster than reading diffs.
- **What is a squash?** Combining several commits into one, so the history records the finished change rather than every "fix typo" step.
- **What makes a good commit message?** A short imperative summary line under about 50 characters, a blank line, then a body explaining *why* rather than *what* (the diff already shows what). One logical change per commit.
- **What is a monorepo?** One repository holding multiple projects — my ClassRoom Code repository is one, with `server/` and `web/` as separate npm packages sharing a history.

## 13. If they ask "walk me through your Git workflow"
"I branch off `main` for each piece of work with a descriptive name. I commit in small logical units with imperative messages. Before opening a PR I rebase onto the latest `main` so the history stays linear and I resolve any conflicts locally rather than in the merge. I push, open a pull request describing what changed and why, and squash-merge once it is reviewed. Anything sensitive stays in a gitignored `.env` with a committed `.env.example` template. If I need to undo something already pushed I use `revert`, not `reset`, because rewriting shared history breaks everyone else's clone."
