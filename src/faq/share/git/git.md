# git

`git` records snapshots of a project and moves those snapshots between machines. It does not deploy, and it does not build. A repository is a `.git` directory plus a work tree. With no arguments it prints help and exits non-zero. Git 2.39 on Linux is assumed here. macOS ships git as well, often a different version, with the same common commands. There is no BusyBox git. Exit status is 0 when the command did what you asked. It is non-zero when the arguments are wrong, the network failed, or the work tree is not in the state the command requires. A commit that records the wrong files is still exit 0. Git does not know the edit was a mistake.

Basic form: git status

The first argument is the subcommand. Flags for that subcommand come after it. Paths come after `--` when a path could be read as a flag or a revision. Quote paths with spaces. A new clone has a remote named `origin` and a branch named `main` or `master`, depending on the server and your `git` version. Commands act on the current repository. Run them from inside the work tree, or pass `-C dir`.

## See what changed
Also asked as: git status; what changed; untracked files; staged changes; is this committed
`git status` lists files that differ from the last commit, split into staged, unstaged, and untracked. Staged means the change is in the index and will be in the next commit. Unstaged means you edited the file after that, or never staged it. Untracked means git has no record of the file. Status does not print the diff. It names the files.

```sh
git status
```

People read "nothing to commit, working tree clean" and assume the remote has the commit. Status is local. People also ignore untracked files and commit without them. A new file is not in the commit until `git add`. A line in `.gitignore` hides a file from this list. `git status -u` shows ignored files too when you need to see them. Status exits 0 even when there is plenty to commit. It is a report, not a test.

## Record a snapshot
Also asked as: git commit; save a commit; commit message; git commit -m; make a commit
`git commit` records the index as a new snapshot and moves the branch tip to it. `-m` sets the message. Without `-m`, git opens an editor. An empty message aborts the commit. A commit with no staged changes also aborts, unless you pass `--allow-empty`. The commit is local. Nothing is sent to a server until you push.

```sh
git commit -m 'pattern'
```

People edit a file and commit, and the new edit is missing. They did not `git add` it. The index is what gets recorded. People also amend a commit they already pushed. `--amend` rewrites the tip. That requires a force push if the old commit was published. Write a new commit instead on a shared branch. A hook can reject the commit and the status is non-zero. The message from the hook is the reason. Fix that, then commit again.

## Stage files
Also asked as: git add; stage a file; git add -p; add to index; track a new file
`git add file` copies the current contents of the file into the index. A new file becomes tracked. An edited file has that version staged. `-p` asks hunk by hunk. `-A` stages everything under the current directory, including deletions. The index is not the commit. You can keep editing after you add, and the later edit is unstaged until you add again.

```sh
git add -- file
```

People `git add` a directory and a secret file goes in. Check `git status` before you commit. People also `git add -p` and leave some hunks unstaged on purpose, then forget. Status shows both. A file in `.gitignore` is not added unless you force it with `-f`. Forcing is how a build artifact gets committed. Do not. `git add` of a missing path exits non-zero. The path is the file as it is now, not as it was at the last commit.

## Show the changes
Also asked as: git diff; unstaged diff; git diff --staged; see the patch; what will be committed
`git diff` shows unstaged changes, work tree against the index. `git diff --staged` shows staged changes, index against the last commit. `git diff main` shows the branch against `main`. With no output, there is no difference of that kind. The diff is a patch. It is not the file.

```sh
git diff
```

```sh
git diff --staged
```

Use the first to see what you have not staged. Use the second to see what the next commit will contain. People commit and are surprised by a hunk they did not mean to include. They looked at `git diff` after staging, which no longer shows those lines. Look at `--staged` before you commit. A binary file is named and not printed. A rename may show as a delete plus an add unless you pass a detection flag. The default is good enough to read.

## Push commits to a remote
Also asked as: git push; upload commits; push origin; set upstream; git push -u
`git push` sends commits to a remote and moves the remote branch. The first push of a new branch needs `-u origin HEAD`, which sets the upstream so later plain `git push` works. A push that is not a fast-forward is rejected. Someone else pushed, or you rewrote history. Pull or rebase first. `--force-with-lease` overwrites and refuses if the remote moved unexpectedly.

```sh
git push -u origin HEAD
```

People push and the server rejects a hook. The commits are still local. Fix the problem the hook named and push again. People also force-push `main`. Protected branches refuse, and unprotected ones lose other people's commits. Force-push a feature branch, not the shared trunk. A missing upstream error means this branch has no tracking branch. `-u` sets it. Push does not upload untracked files or unstaged edits. It uploads commits.

## Fetch and integrate
Also asked as: git pull; git fetch; update from remote; pull rebase; get remote commits
`git fetch` downloads new commits and updates remote-tracking names such as `origin/main`. It does not move your branch. `git pull` fetches and then merges, or rebases if you configured it to. A pull with a dirty work tree can refuse, or it can merge and stop on conflicts. Fetch first when you want to see what arrived before you integrate.

```sh
git fetch origin
```

```sh
git pull
```

Use fetch when you want to look. Use pull when you want your branch updated. People pull and get a merge commit on every update, because pull merges by default. `git pull --rebase` replays your commits on top instead. Set it as the default if that is the project habit. A pull that stops on conflicts is a merge in progress. Continue or abort like any merge. The remote-tracking name does not move your branch. Checking out `origin/main` detaches `HEAD`.

## Clone a repository
Also asked as: git clone; copy a repo; clone a url; git clone --depth; fresh checkout
`git clone URL` creates a new directory, downloads the repository, and checks out the default branch. The URL is HTTPS or SSH. `--depth 1` takes only the tip, which is faster and is a poor base for later history surgery. The clone sets `origin` to that URL. You need a network and, for a private repo, a credential that works.

```sh
git clone -- 'URL'
```

People clone and then look for their files in the parent directory. The files are in the new directory the URL implied. People also clone with a depth and then try to rebase onto old history that was not downloaded. A shallow clone does not have it. Omit `--depth` unless you know you only want the tip. A failed authentication is non-zero and the directory may be half-created. Remove it and clone again after the credentials work.

## Switch branches
Also asked as: git switch; git checkout; change branch; checkout a branch; git switch -c
`git switch name` checks out that branch. Your work tree changes to match it. Uncommitted changes that would be overwritten refuse the switch. `git switch -c name` creates the branch and switches to it. `git checkout` is the older command and also restores files, which is why `switch` exists. A branch name and a file name can collide. `--` is how you say the next word is a path.

```sh
git switch name
```

People have a dirty tree and the switch fails. Commit, stash, or discard. People also `checkout` a file and lose the edits. `checkout -- file` replaces the work tree with the index. `switch` does not take a file. Use `restore` to throw away edits on purpose. A new branch is `-c`. Switching to a name that does not exist is an error. `git branch` lists the ones you have. Remote branches are `origin/name` until you create a local one.

## Create and list branches
Also asked as: git branch; new branch; list branches; delete a branch; git branch -d
`git branch` lists local branches. `-a` includes remote-tracking names. `git branch name` creates a branch at the current commit and does not switch to it. `-d` deletes a branch that has been merged. `-D` deletes even if it has not. Deleting a branch does not delete the commits if something else still points at them. The current branch cannot be deleted.

```sh
git branch
```

People create a branch and wonder why `status` still shows the old one. Create does not switch. `switch -c` does both. People also delete a branch and think the commits are gone. The reflog still has them for a while. A remote branch is not deleted by deleting the local one. `git push origin --delete name` removes it from the server. `-d` refusing means the branch is not merged into the current tip. Check you are on the right branch before you force it with `-D`.

## Read the history
Also asked as: git log; commit history; git log --oneline; who changed this; git log -p
`git log` lists commits on the current branch, newest first. `--oneline` is one line each. `-p` includes the patch. `-- path` limits the log to commits that touched that path. `git log main..HEAD` shows commits on your branch that are not on `main`. Log does not show uncommitted work. Status and diff do.

```sh
git log --oneline
```

People run `git log` on a fresh clone of a huge repo and it pages forever. `--oneline -20` is the short view. People also expect the log to show other branches. It shows the current one unless you pass `--all`. A merge commit has more than one parent. `--first-parent` follows the branch line and hides the merged-in detail. Log exits 0 even when a pager is quit. A path that never existed in this history prints nothing. That is an empty result, not an error.

## Name a remote
Also asked as: git remote; origin; add a remote; git remote -v; change the url
`git remote -v` lists remotes and their URLs. `origin` is the default name of the place you cloned from. `git remote add name URL` adds another. `git remote set-url origin URL` changes where `origin` points. The remote is a name for a URL. Fetch and push use the name. The URL decides the transport, HTTPS or SSH.

```sh
git remote -v
```

People have two remotes and push to the wrong one. `-v` shows fetch and push URLs. They can differ. People also rename `origin` and a script still pushes to `origin`. The name is local. The server does not know it. A remote with no fetch does not update tracking branches. `git fetch name` talks to that remote. A wrong URL fails at fetch or push with a network or an authentication error. `set-url` is the fix. The commits you have are unchanged.

## Set aside uncommitted work
Also asked as: git stash; stash changes; git stash pop; stash push; hide my edits
`git stash` takes staged and unstaged changes and stores them, then restores a clean tree. `git stash pop` applies the latest stash and removes it from the list. `git stash apply` applies it and keeps the stash. A stash is a commit-like object. It is not a branch. A conflict on pop stops and leaves the stash in place so you do not lose it.

```sh
git stash
```

People stash and forget. `git stash list` shows them. People also stash an untracked file and it is not included. `git stash -u` includes untracked files. A pop that conflicts is a merge. Resolve it, or `git checkout` the files and drop the stash if you have given up. `git stash drop` deletes one. There is no trash. A stash is local. It is not pushed with the branch.

## Throw away local changes
Also asked as: git restore; git checkout -- file; discard changes; reset a file; undo edits
`git restore file` puts the work tree copy back to the index. `git restore --staged file` unstages, and leaves the work tree alone. `git checkout -- file` is the older "discard work tree changes." These do not move the branch. They change files. A discard of an untracked file does not happen with `restore`. `git clean` is the untracked-file broom, and it is easy to over-reach.

```sh
git restore -- file
```

People `restore` and expect the last commit to change. The commit is untouched. People also `reset --hard` to discard one file and lose every other uncommitted edit too. `--hard` resets the whole tree to the commit you name. Name a file if you mean one file. There is no undo for a discarded work tree change that was never stashed or committed. It is gone. Commit or stash first if you are unsure.

## Move the branch tip
Also asked as: git reset; undo a commit; git reset --hard; mixed reset; point at a commit
`git reset commit` moves the current branch tip to that commit. The default is `--mixed`. The tip moves, the index matches the new tip, and the work tree is left alone. `--soft` moves the tip and leaves the index. `--hard` moves the tip and replaces the index and the work tree. A hard reset throws away uncommitted work. Pushed commits that you reset away still exist on the remote until you force-push.

```sh
git reset --hard HEAD
```

People `--hard` to undo one bad line and lose a day's edits. Use `restore` for a file. People also reset a shared branch and push. That is a history rewrite. Reset is local until you push. A reset to `HEAD~1` undoes the last commit and, with the default mixed mode, leaves the changes staged. `--hard` discards them. The old tip is in the reflog. `git reset --hard ORIG_HEAD` or a reflog entry is the way back for the tip. It is not a way back for a work tree you discarded.

## Merge a branch
Also asked as: git merge; merge main; fast-forward; merge conflict; integrate a branch
`git merge name` integrates that branch into the current one. A fast-forward moves the tip, because there was nothing to combine. Otherwise git creates a merge commit, or it stops on conflicts. A conflict is both sides edited the same lines. You edit, `git add`, and `git commit` to finish. `git merge --abort` returns to the pre-merge state if you have not committed.

```sh
git merge main
```

People merge and get a conflict, then try to pull again. Finish or abort the merge first. People also merge the wrong way around. The branch you are on receives the merge. Check out the branch that should gain the commits, then merge the other one in. A fast-forward is not a failure. It is a clean move. `--no-ff` forces a merge commit when you want the branch point recorded. A merge does not delete the other branch. It only combines the history.

## Configure your name
Also asked as: git config user.name; git config user.email; set author; first time git; identity
`git config --global user.name` and `user.email` set the author on commits you make. Without them, git refuses to commit, or it guesses from the machine account. `--global` writes `~/.gitconfig`. Omit it to set a value for this repository only. Config is not the remote account. The server has its own user. This is the name in the commit object.

```sh
git config --global user.name 'pattern'
```

People set the name in one clone and expect every clone to have it. `--global` is per machine user, not per repository. A repository-local config overrides it. People also commit before setting the email and the commit has the wrong author. `--amend` on an unpushed tip can fix the latest. Older commits need a rebase. The email is not verified by git itself. A host such as GitHub matches it to an account only if you registered that address.

## Ignore files
Also asked as: .gitignore; stop tracking a file; ignore a pattern; gitignore not working; untrack
`.gitignore` lists patterns that `git status` and `git add` skip. A file that is already tracked is not affected. The ignore hides untracked files. To stop tracking a file you committed by mistake, `git rm --cached file` removes it from the index and leaves the work tree copy. Then ignore it. A pattern of `*.log` ignores that suffix in every directory.

```sh
git rm --cached -- file
```

People add a line to `.gitignore` and the file still appears in `status`. It was already tracked. Remove it from the index. People also ignore a directory and wonder why a force-added file inside it is still tracked. Ignore does not untrack. A leading slash anchors the pattern to the repository root. `secret.txt` matches in any directory. `/secret.txt` matches only at the root. The ignore file is itself a file you should commit, so the whole team skips the same build junk.

## What a failed git means
Also asked as: git exit status; git failed; not a git repository; authentication failed; merge conflict exit
Exit 0 means the command completed. A merge that stops on conflicts exits non-zero and leaves you in the merge. A push that is rejected exits non-zero and changes nothing on the server. "Not a git repository" means you are not in a work tree. Authentication failures come from the credential helper or from SSH, and the message says which. A hook failure is non-zero and the commit or the push did not land.

```sh
git status
```

People treat a conflict stop as a broken repository. It is waiting. `git status` says "all conflicts fixed" or lists the files. People also retry a rejected push in a loop. Read the rejection. A non-fast-forward needs a pull or a rebase. A hook message needs a fix in the content. Under `set -e`, a failing git command aborts a script. That is what you want for a clone or a push in a deploy. A `git diff` that finds differences still exits 0. It is not `grep`. Empty output and status 0 means no difference. Check status before you assume the tree is clean.
