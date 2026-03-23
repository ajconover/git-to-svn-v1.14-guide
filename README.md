# SVN for Git users.

Some useful information can be found at [svnbook.red-bean.com](http://svnbook.red-bean.com).

> **Migrating from SVN to Git?** See the companion guide: [Git for SVN Users (SVN-TO-GIT.md)](SVN-TO-GIT.md).

# 0. Key Differences to Keep in Mind

| Concept | Git | SVN |
|---|---|---|
| Repository | Fully distributed; every clone is a full repo | Centralized; one authoritative server |
| Commits | Local first, then pushed | Always go directly to the server |
| Staging area | Yes (`git add` stages changes) | No staging area; `svn commit` sends all tracked changes |
| Revision IDs | SHA-1 hashes (e.g. `a3f5c1d`) | Sequential integers (e.g. `r4201`) |
| Branching | Lightweight, local pointers | Server-side directory copies (cheap, but still remote) |
| History | Stored locally | Stored on the server |
| Offline work | Full history and commits available offline | Requires server access to commit or view full log |
| Ignoring files | `.gitignore` file | `svn:ignore` property or `.svnignore` file |
| Metadata dir | `.git/` at repo root | `.svn/` in **every** subdirectory |

> **Important for Git users:** Because there is no local staging area and no local commits, every `svn commit` immediately affects the shared repository. Always run `svn update` before committing to avoid conflicts.

# 1. Create A Local Working Copy

```bash
$ svn checkout BRANCH FOLDER_NAME
```

## 1.1 Example

```bash
# Checks out repo/trunk in a folder called trunk
$ svn checkout https://code.example.com/repo/trunk

# Checks out repo/trunk in a folder called repo
$ svn checkout https://code.example.com/repo/trunk repo
```

## 1.2 Git Equivalent

```bash
$ git clone REPO
```

# 2. Create A New Branch

```bash
$ svn copy CURRENT_BRANCH NEW_BRANCH -m "MESSAGE"
```

SVN 1.5+ supports the `^/` prefix as a shorthand for the repository root URL. This means you rarely need to type (or even know) the full server URL when referencing branches, tags, or trunk.

```
^/trunk               →  https://code.example.com/repo/trunk
^/features/my_branch  →  https://code.example.com/repo/features/my_branch
```

## 2.1 Example

```bash
# Traditional: full URLs
$ svn copy https://code.example.com/repo/trunk \
    https://code.example.com/repo/features/feature_branch \
    -m "Creating a private branch of /repo/trunk."

# Modern: using ^ (caret) shorthand for the repository root (SVN 1.5+)
$ svn copy ^/trunk ^/features/feature_branch \
    -m "Creating a private branch of /repo/trunk."
```

## 2.2 Git Equivalent

```bash
$ git checkout -b BRANCH
```

# 3. Switch To a Different Branch

```bash
$ svn switch BRANCH
```

## 3.1 Example

```bash
# Traditional: full URL
$ svn switch https://code.example.com/repo/features/feature_branch

# Modern: using ^ shorthand
$ svn switch ^/features/feature_branch
```
## 3.2 Git Equivalent

```bash
$ git checkout BRANCH
```

# 4. Keep The Branch In Sync

```bash
$ svn update
```

## 4.1 Example

```bash
$ svn switch ^/trunk
$ svn update
$ svn switch ^/features/feature_branch
$ svn merge ^/trunk
$ svn ci -m "Merge branch trunk into feature_branch"
```

> **Note:** SVN automatically tracks which revisions have already been merged (via `svn:mergeinfo`). For a regular sync you do **not** need to specify a revision range — `svn merge ^/trunk` will merge only the revisions not yet merged into your branch. Explicit ranges (e.g. `-r 4100:4200`) are only needed when intentionally cherry-picking a specific set of commits.

## 4.2 Git Equivalent

```bash
$ git pull
```

# 5. Merge to Trunk

```bash
$ svn merge BRANCH
```

## 5.1 Example

```bash
$ svn switch ^/trunk
$ svn update
$ svn merge ^/features/my_feature
$ svn update
$ svn commit -m "Merge branch my_feature into trunk"
```

> **Note:** The `--reintegrate` flag is deprecated since SVN 1.8 — a plain `svn merge` now handles reintegration automatically. As with sync merges, you do **not** need to specify a revision range; SVN's merge tracking ensures only unmerged revisions are applied. Use an explicit range only when cherry-picking specific commits.

## 5.2 Git Equivalent

```bash
$ git merge BRANCH
```

# 6. Status

```bash
$ svn status
```

## 6.1 Git Equivalent

```bash
$ git status
```

# 7. Committing

```bash
$ svn commit -m "MESSAGE"
```

## 7.1 Example

```bash
$ svn update
$ svn status
$ svn add PATH/TO/NEW/FILES
$ svn commit -m "Added an awesome feature"
```

## 7.2 Git Equivalent

```bash
$ git commit -m "Added an awesome feature" && git push
```

# 8. Commit A Single File

```bash
$ svn commit FILE -m "MESSAGE"
```

## 8.1 Example

```bash
$ svn commit app/models/awesome.rb -m "Adding some awesome"
```

## 8.2 Git Equivalent

```bash
$ git add app/models/awesome.rb && git commit -m "Adding some awesome" && git push
```

# 9. View Repository Structure

```bash
$ svn ls REPO
```

## 9.1 Example

```bash
$ svn ls https://code.example.com/repo
branches/
features/
tags/
trunk/

# Using ^ shorthand from inside any working copy of this repository
$ svn ls ^/
branches/
features/
tags/
trunk/

$ svn ls ^/features
sprint_17/
sprint_18/
sprint_19/
```

## 9.2 Git Equivalent

```bash
$ git branch -a
```

# 10. View Repository Details

```bash
$ svn info
```

# 11. Revert Changes

```bash
 $ svn revert
```

## 11.1 Example

```bash
$ svn revert . -R
$ svn revert /PATH/TO/FILE
```

## 11.2 Git Equivalent

```bash
$ git checkout /PATH/TO/FILE
```

# 12. List latest revision

```bash
$ svn log
```
## 12.1 Example

```bash
$ svn log /PATH/TO/FILE -v -l3
```

## 12.2 Git Equivalent

```bash
$ git log -n 3
```

# 13. View diff of a commit

```bash
$ svn log --diff
```

## 13.1 Example

```bash
$ svn log -r 42256 --diff
```

## 13.2 Git Equivalent

```bash
$ git diff 29461219405dcdee17194d0e3112f160e1345d49
```

# 14. Merge Conflicts

Accept whatever the current directory structure is at this time:

```bash
$ svn resolve --accept working . -R
```

# 15. The "Git Way"

## 15.1 Clone Trunk

```bash
$ svn checkout https://code.example.com/repo/trunk repo
```

## 15.2 Create Feature Branch

```bash
$ svn copy ^/trunk ^/features/feature_branch \
    -m "Creating a private branch of /repo/trunk."
```

## 15.3 Switch

```bash
$ svn switch ^/features/feature_branch
```

## 15.4 Make Changes

```bash
$ rm *.php
$ rails new awesome_app
```

## 15.5 Add New Files

```bash
$ svn status
$ svn add . --force
```

## 15.6 Commit New Files

```bash
$ svn commit -m "Made some awesome"
```

## 15.7 Switch to Trunk

```bash
$ svn switch ^/trunk
```

## 15.8 Update From Upstream

```bash
$ svn update
```

## 15.9 Merge New Feature

```bash
$ svn merge ^/features/feature_branch
```

## 15.10 Commit Merge

```bash
$ svn commit -m "Merge branch feature_branch into trunk"
```

# 16. View Diff of Working Copy

SVN has no staging area, so `svn diff` compares your working copy against the last updated revision (BASE).

```bash
$ svn diff
```

## 16.1 Example

```bash
# Diff all changes in the working copy
$ svn diff

# Diff a specific file
$ svn diff app/models/awesome.rb

# Diff against a specific revision
$ svn diff -r 4200 app/models/awesome.rb

# Diff between two revisions
$ svn diff -r 4199:4200 app/models/awesome.rb
```

## 16.2 Git Equivalent

```bash
# Working copy vs last commit
$ git diff

# Specific file
$ git diff app/models/awesome.rb

# Between two commits
$ git diff abc123 def456 -- app/models/awesome.rb
```

# 17. Track File Changes (Add, Delete, Move)

SVN tracks files explicitly. Unlike Git, new files are **not** automatically staged; you must tell SVN to track them.

## 17.1 Add a New File

```bash
$ svn add PATH/TO/FILE
```

### Add all untracked files recursively

```bash
$ svn add . --force
```

### Git Equivalent

```bash
$ git add PATH/TO/FILE
```

## 17.2 Delete a File

```bash
$ svn delete PATH/TO/FILE
```

> **Warning:** This removes the file from disk **and** schedules it for deletion on the next commit. Use `svn delete --keep-local` to remove it from SVN tracking without deleting it from disk.

### Git Equivalent

```bash
$ git rm PATH/TO/FILE
```

## 17.3 Move or Rename a File

```bash
$ svn move OLD_PATH NEW_PATH
```

### Example

```bash
$ svn move app/models/old_name.rb app/models/new_name.rb
```

### Git Equivalent

```bash
$ git mv OLD_PATH NEW_PATH
```

# 18. Ignoring Files (svn:ignore)

SVN does not use a `.gitignore` file. Instead it uses the `svn:ignore` property on a directory, or (in SVN 1.8+) a global `.svnignore` file.

## 18.1 Set svn:ignore on a Directory

```bash
# Open an editor to set the ignore list on the current directory
$ svn propedit svn:ignore .

# Set a single pattern non-interactively
$ svn propset svn:ignore "*.log" logs/
```

## 18.2 Use a .svnignore File (SVN 1.8+)

Create a `.svnignore` file in the working copy root with the same syntax as `.gitignore`:

```
*.log
*.tmp
node_modules/
```

Then set the global ignore list to pick it up:

```bash
$ svn propset svn:global-ignores -F .svnignore .
```

## 18.3 Git Equivalent

```bash
# .gitignore file in the repo root
echo "*.log" >> .gitignore
```

# 19. Blame (Annotate)

Show who last modified each line of a file and at which revision.

```bash
$ svn blame PATH/TO/FILE
```

## 19.1 Example

```bash
$ svn blame app/models/awesome.rb

# Show blame for a specific revision range
$ svn blame -r 4000:4200 app/models/awesome.rb
```

## 19.2 Git Equivalent

```bash
$ git blame app/models/awesome.rb
```

# 20. Revision Specifiers

SVN uses integer revision numbers instead of SHA hashes. Several keywords are also available:

| Keyword | Meaning |
|---|---|
| `HEAD` | Latest revision on the server |
| `BASE` | The revision your working copy was last updated to |
| `COMMITTED` | Last revision in which the path changed |
| `PREV` | The revision before `COMMITTED` |

## 20.1 Examples

```bash
# Update to a specific revision (like git checkout <sha>)
$ svn update -r 4200

# View a file at a specific revision
$ svn cat -r 4200 app/models/awesome.rb

# Diff working copy against a specific revision
$ svn diff -r 4199

# Revert a single file to BASE (last updated revision)
$ svn revert app/models/awesome.rb

# Revert entire working copy to HEAD
$ svn update -r HEAD
```

## 20.2 Git Equivalent

```bash
# Checkout a specific commit
$ git checkout abc1234

# View a file at a specific commit
$ git show abc1234:app/models/awesome.rb
```

# 21. Cleanup (Recover from Interrupted Operations)

If an SVN operation (commit, update, merge) is interrupted, the working copy can be left in a locked state. Run `svn cleanup` to recover.

```bash
$ svn cleanup
```

## 21.1 Example

```bash
# Unlock and resume any interrupted operations
$ svn cleanup

# Also remove unversioned files (SVN 1.9+)
$ svn cleanup --remove-unversioned
```

## 21.2 Git Equivalent

```bash
# Git rarely gets stuck, but for an interrupted merge:
$ git merge --abort

# Or reset hard
$ git reset --hard HEAD
```

# 22. Working Copy Info

`svn info` shows the URL, revision, and other metadata for your working copy or a specific path. This is the quickest way to find out "where am I?" in SVN.

```bash
$ svn info
```

## 22.1 Example Output

```
Path: .
Working Copy Root Path: /home/user/repo
URL: https://code.example.com/repo/trunk
Relative URL: ^/trunk
Repository Root: https://code.example.com/repo
Repository UUID: 12345678-abcd-ef01-2345-6789abcdef01
Revision: 4201
Node Kind: directory
Schedule: normal
Last Changed Author: jdoe
Last Changed Rev: 4198
Last Changed Date: 2024-01-15 10:32:07 +0000 (Mon, 15 Jan 2024)
```

## 22.2 Git Equivalent

```bash
$ git remote -v
$ git log -1
```

# 23. SVN Tricks: Simulating Git-Like Behavior

SVN lacks some conveniences that Git users rely on daily. The tricks below let you approximate them using standard SVN operations.

## 23.1 Stash — Patch File Approach

Save uncommitted changes to a patch file, revert the working copy, and re-apply later.

```bash
# "Stash" — save changes and clean the working copy
$ svn diff > /tmp/my_stash.patch
$ svn revert . -R

# ... switch tasks, update, do other work ...

# "Pop" — re-apply your saved changes
$ svn patch /tmp/my_stash.patch
```

### Git Equivalent

```bash
$ git stash
$ git stash pop
```

## 23.2 Stash — Branch Copy Approach

If you want your stashed work stored safely on the server (not just a local file), copy the working copy — including uncommitted modifications — to a temporary stash branch, then revert.

```bash
# Copy the current working copy state (with local changes) to a stash branch
$ svn copy . ^/stashes/my_stash -m "Stashing work-in-progress"
$ svn revert . -R

# ... switch tasks, update, do other work ...

# Re-apply by merging the stash branch back
$ svn merge ^/stashes/my_stash

# Clean up the stash branch when done
$ svn delete ^/stashes/my_stash -m "Removing stash after applying"
```

> **Tip:** Create the `stashes/` directory in the repository once with `svn mkdir ^/stashes -m "Add stashes directory"` before using this workflow.

### Git Equivalent

```bash
$ git stash
$ git stash pop
```

## 23.3 Shelve (SVN 1.10+)

SVN 1.10 introduced native shelving, which is the closest built-in equivalent of `git stash`.

```bash
# Shelve all current changes under a name
$ svn shelve my_stash

# List shelved change sets
$ svn shelves

# Unshelve (re-apply and remove the shelf)
$ svn unshelve my_stash
```

### Git Equivalent

```bash
$ git stash save "my_stash"
$ git stash list
$ git stash pop
```

## 23.4 Dry-Run Merge (Preview Before Merging)

Check what a merge would do before actually applying it.

```bash
# Show which revisions would be merged
$ svn mergeinfo --show-revs eligible ^/features/my_feature

# Preview the diff that a merge would produce
$ svn merge --dry-run ^/features/my_feature
```

### Git Equivalent

```bash
$ git log HEAD..BRANCH --oneline
$ git diff HEAD...BRANCH
```

## 23.5 Cherry-Pick a Single Commit

To apply only one specific revision from another branch (the only time an explicit revision range is truly necessary):

```bash
# Cherry-pick revision 4205 from another branch
$ svn merge -c 4205 ^/features/my_feature

# Cherry-pick a range (e.g. r4200 through r4205)
$ svn merge -r 4199:4205 ^/features/my_feature
```

> **Note:** `-c N` is shorthand for `-r N-1:N`.  For ordinary sync and reintegration merges, leave out the revision range entirely — SVN's merge tracking (`svn:mergeinfo`) ensures only unmerged revisions are applied.

### Git Equivalent

```bash
$ git cherry-pick <commit-sha>
$ git cherry-pick <sha1>^..<sha2>
```

