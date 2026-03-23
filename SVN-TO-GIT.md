# Git for SVN Users

A migration guide for experienced SVN users moving to Git. Each section leads with the SVN concept or command you already know, then shows how Git handles the same idea.

Some useful Git reference material can be found at [git-scm.com/docs](https://git-scm.com/docs).

---

# 0. Key Conceptual Mappings

The most important shift when moving from SVN to Git is that **Git is distributed**. Every developer has a full copy of the repository — history and all — on their local machine. There is no single authoritative server you must talk to in order to commit or view history.

| SVN Concept | SVN | Git |
|---|---|---|
| Repository | Centralized; one authoritative server | Distributed; every clone is a full repo |
| Working copy | Checked-out subtree (e.g. `trunk/`) | Full repository clone |
| Commit | Immediately written to the server | Written locally first; shared with `git push` |
| Staging area | None — all tracked changes go in the commit | Yes — `git add` selects which changes to include |
| Revision ID | Sequential integer (e.g. `r4201`) | SHA-1 hash (e.g. `a3f5c1d`) |
| Branching | Server-side directory copy under `branches/` | Local pointer; no server round-trip needed |
| Switching branches | `svn switch <URL>` re-writes the working copy | `git checkout <branch>` or `git switch <branch>` |
| Trunk | `trunk/` directory by convention | `main` (or `master`) branch by convention |
| Tagging | Server-side directory copy under `tags/` | Lightweight or annotated pointer to a commit |
| History | Stored on the server | Stored locally in `.git/` |
| Offline work | Cannot commit or view full log | Full history and commits available offline |
| Ignoring files | `svn:ignore` property or `.svnignore` | `.gitignore` file committed to the repo |
| Metadata dir | `.svn/` in **every** subdirectory | `.git/` at the **repo root only** |
| Conflict resolution | `svn resolve` | `git add` (mark resolved) then `git commit` |

> **Important for SVN users:** Because commits are local, you can commit as often as you like without affecting anyone else. Share your work with the team by running `git push`. Always `git pull` (or `git fetch` + `git merge`) before pushing to incorporate upstream changes.

---

# 1. Get a Working Copy (Checkout → Clone)

In SVN you check out a specific branch URL. In Git you clone the entire repository; every branch is already available locally after the clone.

**SVN**
```bash
$ svn checkout https://code.example.com/repo/trunk
```

**Git**
```bash
$ git clone https://github.com/example/repo.git
```

After cloning, `cd` into the new directory and you are already on the default branch (usually `main`).

```bash
$ git clone https://github.com/example/repo.git
$ cd repo
$ git branch -a          # see all local and remote branches
```

---

# 2. Understand the Staging Area (Index)

SVN has no staging area — `svn commit` sends every tracked change immediately. Git separates the two steps:

| Step | SVN | Git |
|---|---|---|
| Select changes | (automatic — all tracked changes) | `git add <file>` |
| Write to history | `svn commit -m "..."` | `git commit -m "..."` |
| Share with team | (happens automatically on commit) | `git push` |

```bash
# Stage specific files
$ git add app/models/user.rb

# Stage all changes in the working directory
$ git add .

# Interactively choose hunks to stage
$ git add -p
```

> **Tip:** `git status` always shows you which changes are staged (will go in the next commit) vs. unstaged (not yet selected).

---

# 3. Check Status

**SVN**
```bash
$ svn status
```

**Git**
```bash
$ git status
```

Git `status` output distinguishes three categories:

- **Staged** — changes added with `git add`, ready to commit
- **Unstaged** — tracked files with modifications not yet staged
- **Untracked** — new files Git is not yet tracking

---

# 4. Create a Branch

In SVN a branch is a directory copy on the server. In Git a branch is a lightweight local pointer — no server round-trip is needed.

**SVN**
```bash
$ svn copy ^/trunk ^/features/feature_branch \
    -m "Creating a private branch of /repo/trunk."
```

**Git**
```bash
$ git checkout -b feature_branch        # create and switch in one step
# or, using the newer syntax:
$ git switch -c feature_branch
```

Git also lets you push the branch to the remote server when you are ready to share it:

```bash
$ git push -u origin feature_branch     # publish the branch
```

---

# 5. Switch Branches

**SVN**
```bash
$ svn switch ^/features/feature_branch
```

**Git**
```bash
$ git checkout feature_branch
# or
$ git switch feature_branch
```

Unlike SVN, switching branches in Git does not contact the server. The `.git/` directory already contains all branch data.

---

# 6. Commit Changes

This is the biggest mental shift. In SVN a commit immediately writes to the shared server. In Git, **commit is a local operation**; a separate `push` shares your work.

**SVN workflow**
```bash
$ svn update                           # sync with server first
$ svn add PATH/TO/NEW/FILES            # schedule new files
$ svn commit -m "Added an awesome feature"   # write directly to server
```

**Git workflow**
```bash
$ git pull                             # sync with remote first
$ git add PATH/TO/NEW/FILES            # stage new files
$ git add .                            # stage all changes
$ git commit -m "Added an awesome feature"   # write to local history
$ git push                             # share with the team
```

## 6.1 Commit a Specific File

**SVN**
```bash
$ svn commit app/models/awesome.rb -m "Adding some awesome"
```

**Git**
```bash
$ git add app/models/awesome.rb
$ git commit -m "Adding some awesome"
$ git push
```

---

# 7. Sync With the Remote (Update → Pull)

**SVN**
```bash
$ svn update
```

**Git**
```bash
$ git pull
```

`git pull` is shorthand for `git fetch` (download new commits) followed by `git merge` (integrate them into your current branch). You can also do these steps separately for more control:

```bash
$ git fetch origin                     # download without merging
$ git merge origin/main                # integrate explicitly
```

---

# 8. Keep a Feature Branch in Sync With Main

**SVN**
```bash
$ svn switch ^/trunk
$ svn update
$ svn switch ^/features/feature_branch
$ svn merge ^/trunk
$ svn ci -m "Merge trunk into feature_branch"
```

**Git**
```bash
$ git checkout main
$ git pull
$ git checkout feature_branch
$ git merge main                       # or: git rebase main
$ git push
```

> **Tip:** Many Git teams prefer `git rebase main` over `git merge main` for syncing feature branches — it keeps the history linear and avoids "merge noise" commits.

---

# 9. Merge a Feature Branch to Main (Reintegrate)

**SVN**
```bash
$ svn switch ^/trunk
$ svn update
$ svn merge ^/features/my_feature
$ svn commit -m "Merge branch my_feature into trunk"
```

**Git**
```bash
$ git checkout main
$ git pull
$ git merge feature_branch
$ git push
```

After the merge you can safely delete the local and remote branch:

```bash
$ git branch -d feature_branch         # delete locally
$ git push origin --delete feature_branch  # delete on remote
```

---

# 10. View History (Log)

**SVN**
```bash
$ svn log
$ svn log /PATH/TO/FILE -v -l 3
```

**Git**
```bash
$ git log
$ git log -n 3                         # last 3 commits
$ git log --oneline                    # compact one-line format
$ git log --oneline --graph --all      # visual branch graph
$ git log -n 3 -- app/models/user.rb   # history for a file
```

Note that SVN revision numbers (e.g. `r4201`) do not exist in Git. Each commit is identified by a SHA-1 hash. You only need to type enough characters to be unambiguous (usually 7–8).

---

# 11. View a Diff

SVN compares your working copy against the `BASE` revision (last updated). Git compares against the last commit (or a specific commit/branch you name).

**SVN**
```bash
$ svn diff
$ svn diff app/models/awesome.rb
$ svn diff -r 4199:4200 app/models/awesome.rb
```

**Git**
```bash
# Working copy vs last commit (unstaged changes)
$ git diff

# Staged changes (what will go in the next commit)
$ git diff --staged

# Specific file
$ git diff app/models/awesome.rb

# Between two commits or branches
$ git diff main..feature_branch
$ git diff abc123 def456 -- app/models/awesome.rb
```

## 11.1 View a Specific Commit's Diff

**SVN**
```bash
$ svn log -r 42256 --diff
```

**Git**
```bash
$ git show abc1234
```

---

# 12. Revert Changes

**SVN**
```bash
$ svn revert . -R                      # revert all changes
$ svn revert /PATH/TO/FILE             # revert one file
```

**Git**

```bash
# Discard unstaged changes in a file
$ git checkout -- /PATH/TO/FILE
# or (newer syntax)
$ git restore /PATH/TO/FILE

# Discard all unstaged changes
$ git restore .

# Unstage a file (keep the changes in the working directory)
$ git restore --staged /PATH/TO/FILE
```

To undo a commit that has already been pushed, use `git revert` (creates a new "undo" commit, safe for shared branches):

```bash
$ git revert abc1234
```

---

# 13. Track File Changes (Add, Delete, Move)

## 13.1 Add a New File

**SVN**
```bash
$ svn add PATH/TO/FILE
$ svn add . --force          # add all untracked files recursively
```

**Git**
```bash
$ git add PATH/TO/FILE
$ git add .                  # add everything
```

## 13.2 Delete a File

**SVN**
```bash
$ svn delete PATH/TO/FILE
```

**Git**
```bash
$ git rm PATH/TO/FILE
```

> `git rm` removes the file from disk **and** stages the deletion for the next commit, mirroring `svn delete` behavior.

## 13.3 Move or Rename a File

**SVN**
```bash
$ svn move app/models/old_name.rb app/models/new_name.rb
```

**Git**
```bash
$ git mv app/models/old_name.rb app/models/new_name.rb
```

---

# 14. Ignoring Files

SVN uses a property (`svn:ignore`) set on a directory, or a `.svnignore` file in SVN 1.8+. Git uses a `.gitignore` file committed to the repository.

**SVN**
```bash
$ svn propset svn:ignore "*.log" logs/
```

**Git**

Create (or edit) a `.gitignore` file in the repository root (or any subdirectory):

```
*.log
*.tmp
node_modules/
```

Then commit the file:

```bash
$ git add .gitignore
$ git commit -m "Add .gitignore"
```

> Unlike `svn:ignore`, `.gitignore` is just a regular file in the repo — every team member gets the same ignore rules automatically.

---

# 15. Blame (Annotate)

Show who last modified each line of a file.

**SVN**
```bash
$ svn blame app/models/awesome.rb
$ svn blame -r 4000:4200 app/models/awesome.rb
```

**Git**
```bash
$ git blame app/models/awesome.rb
$ git blame -L 10,25 app/models/awesome.rb   # lines 10–25 only
```

---

# 16. Revision Specifiers (SVN Keywords → Git Refs)

SVN uses integer revision numbers and keywords. Git uses SHA hashes and named refs.

| SVN | Git equivalent | Meaning |
|---|---|---|
| `HEAD` | `HEAD` | Latest commit on the current branch |
| `BASE` | *(no direct equivalent)* | SVN BASE is the revision your working copy was last updated to; in Git, `HEAD` is the last commit and uncommitted changes are compared against it with `git diff` |
| `r4201` | `abc1234` (SHA) | A specific revision / commit |
| `r4200:r4205` | `abc123..def456` | A range of revisions / commits |
| `^/trunk` | `origin/main` | The trunk / main branch on the remote |

```bash
# Checkout a specific commit (like svn update -r 4200)
$ git checkout abc1234

# View a file at a specific commit
$ git show abc1234:app/models/awesome.rb

# Diff against a specific commit
$ git diff abc1234

# Diff between two commits
$ git diff abc1234..def5678
```

---

# 17. Tagging

In SVN, a tag is a directory copy under `tags/` (just like a branch). In Git, a tag is a named pointer to a commit.

**SVN**
```bash
$ svn copy ^/trunk ^/tags/v1.0.0 -m "Tagging version 1.0.0"
```

**Git**
```bash
# Lightweight tag (just a pointer)
$ git tag v1.0.0

# Annotated tag (recommended — stores tagger, date, message)
$ git tag -a v1.0.0 -m "Version 1.0.0"

# Push tags to the remote (tags are not pushed by default)
$ git push origin v1.0.0
$ git push origin --tags    # push all tags
```

---

# 18. Cherry-Pick a Specific Revision

**SVN**
```bash
$ svn merge -c 4205 ^/features/my_feature
```

**Git**
```bash
$ git cherry-pick abc1234
$ git cherry-pick abc1234..def5678      # a range of commits (abc1234 exclusive to def5678 inclusive)
$ git cherry-pick abc1234^..def5678     # include abc1234 itself through def5678
```

---

# 19. Stash Uncommitted Changes

SVN has no built-in stash (SVN 1.10+ has experimental shelving). Git's stash is a first-class feature.

**SVN (patch file workaround)**
```bash
$ svn diff > /tmp/my_stash.patch
$ svn revert . -R
# ... do other work ...
$ svn patch /tmp/my_stash.patch
```

**Git**
```bash
$ git stash                            # save and clean working copy
$ git stash list                       # see all stashes
$ git stash pop                        # re-apply the most recent stash
$ git stash push -m "WIP: my feature"  # stash with a description
```

---

# 20. Recover From Interrupted Operations (Cleanup)

In SVN, an interrupted operation can leave the working copy locked — `svn cleanup` fixes it. Git is much more resilient, but aborted merges or rebases do leave partial state.

**SVN**
```bash
$ svn cleanup
$ svn cleanup --remove-unversioned
```

**Git**
```bash
# Abort an in-progress merge
$ git merge --abort

# Abort an in-progress rebase
$ git rebase --abort

# Discard all local changes and reset to last commit
$ git reset --hard HEAD
```

---

# 21. Working Copy / Repo Info

**SVN**
```bash
$ svn info
```

**Git**
```bash
$ git remote -v                        # show configured remotes
$ git log -1                           # show the last commit
$ git status                           # current branch and working copy state
$ git branch -a                        # list all local and remote branches
```

---

# 22. The "Git Way" — End-to-End Feature Branch Workflow

This mirrors the "Git Way" section from the companion SVN guide, translated fully into Git.

## 22.1 Clone the Repository

```bash
$ git clone https://github.com/example/repo.git
$ cd repo
```

## 22.2 Create a Feature Branch

```bash
$ git checkout -b feature_branch
```

## 22.3 Make Changes

```bash
$ rm *.php
$ rails new awesome_app
```

## 22.4 Stage and Commit New Files

```bash
$ git status
$ git add .
$ git commit -m "Made some awesome"
```

## 22.5 Push the Feature Branch

```bash
$ git push -u origin feature_branch
```

## 22.6 Switch to Main and Update

```bash
$ git checkout main
$ git pull
```

## 22.7 Merge the Feature Branch

```bash
$ git merge feature_branch
```

## 22.8 Push the Merge to Remote

```bash
$ git push
```

## 22.9 Clean Up

```bash
$ git branch -d feature_branch                  # delete local branch
$ git push origin --delete feature_branch       # delete remote branch
```

---

# 23. Quick Reference Cheat Sheet

| Task | SVN | Git |
|---|---|---|
| Get a working copy | `svn checkout <URL>` | `git clone <URL>` |
| Check status | `svn status` | `git status` |
| Stage changes | *(automatic)* | `git add <file>` |
| Commit | `svn commit -m "..."` | `git commit -m "..."` then `git push` |
| Sync with server | `svn update` | `git pull` |
| Create branch | `svn copy ^/trunk ^/branches/X` | `git checkout -b X` |
| Switch branch | `svn switch ^/branches/X` | `git checkout X` |
| Merge branch | `svn merge ^/branches/X` + commit | `git merge X` |
| View log | `svn log` | `git log` |
| View diff | `svn diff` | `git diff` |
| Revert file | `svn revert <file>` | `git restore <file>` |
| Add file | `svn add <file>` | `git add <file>` |
| Delete file | `svn delete <file>` | `git rm <file>` |
| Move file | `svn move <old> <new>` | `git mv <old> <new>` |
| Ignore files | `svn:ignore` property | `.gitignore` file |
| Blame | `svn blame <file>` | `git blame <file>` |
| Tag release | `svn copy ^/trunk ^/tags/vX` | `git tag -a vX -m "..."` + `git push --tags` |
| Cherry-pick | `svn merge -c <rev>` | `git cherry-pick <sha>` |
| Stash changes | `svn diff > patch` + `svn revert` | `git stash` / `git stash pop` |
| Abort a merge | *(recover with `svn cleanup`)* | `git merge --abort` |
