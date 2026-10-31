# git rebase

`git rebase` replays your commits on top of another commit. It rewrites history. It does not merge two lines of work into a merge commit. It does not push. With no arguments it rebases the current branch onto its upstream. Git 2.39 on Linux is assumed here. macOS ships git as well, often a different version, with the same common flags. There is no BusyBox git. Exit status is 0 when the rebase finished. It is non-zero when the rebase stopped for a conflict, or when the arguments are wrong. A stopped rebase is not a failed command in the sense that your branch is ruined. It is waiting. `--abort` returns you to where you were.

Basic form: git rebase upstream

`upstream` is the commit your work should sit on, often `main` or `origin/main`. Git takes the commits that are on your branch and not on that upstream, and replays them. Your branch pointer moves to the new commits. The old commits become unreachable, except through the reflog. A rebase of commits you already pushed requires a force push. Do not rebase commits other people have based work on.

## Replay your commits on top of main
Also asked as: git rebase main; update my branch; rebase onto main; replay commits; catch up with main
`git rebase main` takes the commits that are on your current branch and not on `main`, and replays them on the tip of `main`. Your branch now starts where `main` starts, plus your work. The commit hashes change. The messages and the patches stay, unless a conflict edited them. Fetch first if `main` is a remote-tracking name and you need it current.

```sh
git rebase main
```

People rebase the wrong branch. Check `git status` before you start. People also rebase `main` onto their feature branch, which moves `main`. The branch you are on is the one that moves. Check out the feature, then rebase it onto `main`. A rebase that says "up to date" means there was nothing to replay. You are already based on that commit. It is not an error.

## Stop when a commit conflicts
Also asked as: rebase conflict; could not apply; fix conflicts and continue; rebase stopped; resolve and go on
If a replayed commit conflicts, rebase stops. The index holds the conflicts. Edit the files, `git add` the resolutions, and `git rebase --continue`. Do not create a new commit yourself. `--continue` uses the resolution as the replayed commit. `--skip` drops the commit that conflicted. `--abort` throws the whole rebase away and puts you back on the branch as it was.

```sh
git rebase --continue
```

```sh
git rebase --abort
```

Use `--continue` when you fixed the conflict. Use `--abort` when you want out. People `git commit` in the middle of a rebase and create a stray commit. The rebase wants `--continue`. People also `--skip` a conflict they meant to fix, and that commit is gone from the branch. It is still in the reflog for a while. A stopped rebase is remembered in `.git`. A new `git rebase` will refuse until you continue or abort. Status says you are in the middle of one.

## Abort and go back
Also asked as: git rebase --abort; undo a rebase; cancel rebase; back to where I was; stop rebasing
`--abort` returns the branch to the tip it had when the rebase started, and clears the rebase state. Use it when the conflicts are a mess or you rebased onto the wrong thing. It does not undo a rebase that already finished. A finished rebase is undone with the reflog, `git reset --hard` to the old tip. `--abort` is only for one that is still stopped.

```sh
git rebase --abort
```

People look for an undo after the rebase completed and `--abort` says there is nothing to abort. The reflog has `HEAD@{1}` from before the rebase. People also `--abort` after resolving, expecting to keep the resolutions. Abort discards them. Continue if you want the resolutions. A rebase that was `--quit` rather than `--abort` leaves `HEAD` where it is and drops the rebase state. That is the rarer exit. `--abort` is the one that restores the branch.

## Rebase onto a different base
Also asked as: git rebase --onto; move commits; transplant a branch; rebase onto; drop the old base
`--onto` picks the new base, and the upstream argument picks where the commits to replay start. `git rebase --onto newbase oldbase` replays commits that are on the current branch and not in `oldbase`, onto `newbase`. This is how you move a branch off an old parent, or drop a few commits at the front of the range. It is more precise than a plain rebase.

```sh
git rebase --onto main oldbase
```

People pass one argument to `--onto` and expect it to know which commits. The commits are "not in the next argument." A missing upstream rebases everything reachable, which can be far more than you meant. People also use `--onto` where a plain `git rebase main` would do. Plain rebase is the common case. `--onto` is the transplant. Draw the three points before you run it. The reflog is the way back if the picture was wrong.

## Interactive rebase
Also asked as: git rebase -i; interactive; reorder commits; edit commits; squash; reword
`-i` opens the list of commits to replay. You can reorder lines, delete a line to drop a commit, or change `pick` to `reword`, `squash`, `fixup`, or `edit`. Save and close, and git follows the list. A conflict still stops you. `--continue` resumes the list. This is the tool for cleaning a branch before you push it. It rewrites every commit from the first you touch.

```sh
git rebase -i main
```

People change `pick` to `squash` on the first line. The first commit has nothing above it to squash into. Put `squash` on a later line. People also reorder a commit under one it depends on, and the replay conflicts. The list is applied top to bottom. An empty list aborts the idea and leaves the branch. A `reword` opens the message. A `fixup` squashes and keeps the earlier message. Both change hashes. Anything already pushed needs a force push after.

## Squash fixup commits
Also asked as: git rebase --autosquash; fixup!; squash!; autosquash; fold a fixup
`--autosquash` with `-i` moves commits whose message starts with `fixup!` or `squash!` under the commit they name. You make the small commit with `git commit --fixup OLD`, then rebase. The list comes up already arranged. `fixup!` keeps the original message. `squash!` asks you to combine messages. This is the tidy way to absorb review fixes.

```sh
git rebase -i --autosquash main
```

People write `fixup` in the message by hand with the wrong hash, and autosquash does not match. The subject has to match the target. `--fixup` on the commit command writes it correctly. People also autosquash onto a base that does not contain the target commit. The fixup stays where it is. The target has to be in the list. Autosquash does nothing without `-i`. It only rearranges the todo. You still save the list.

## Skip a commit that should not be replayed
Also asked as: git rebase --skip; drop a conflicting commit; skip this patch; ignore this commit; continue without it
`--skip` drops the commit that is currently stopped and goes on to the next. Use it when the change is already in the new base, or when you have decided the commit should not exist. It is not an undo of the whole rebase. The skipped commit is gone from the branch. It remains in the reflog until it expires.

```sh
git rebase --skip
```

People `--skip` a conflict they only half resolved. The resolution is discarded and the commit is dropped. Continue if you wanted the resolution. People also skip and expect the changes to still be in the work tree. They are not. Skip moves on. A commit that becomes empty because the base already has the change is skipped automatically on modern git, or it stops, depending on `--empty`. The default is to ask or to drop, by version. An empty commit you meant to keep needs `--empty=keep`.

## Keep going after you fixed the files
Also asked as: git rebase --continue; resume rebase; after resolving; go on; finish the rebase
`--continue` reads the index and makes the replayed commit from it, then proceeds to the next commit in the list. The conflict markers must be gone. The files you resolved must be staged. If you fixed a file and did not `git add` it, continue refuses. An empty resolution, where your fix undoes the commit, may skip or stop depending on settings.

```sh
git rebase --continue
```

People add the files and then `git commit`. In a rebase, continue is the commit. A second commit splits the replay. People also continue with conflict markers still in the file. Git often notices and refuses. If it does not, the markers become part of the commit. Search the tree before you continue. A rebase with a long list stops on every conflict. Each one is a continue. The branch tip does not move to the final result until the list is done.

## See the patch that stopped
Also asked as: git rebase --show-current-patch; what is conflicting; current patch; show the commit being applied
`--show-current-patch` prints the commit rebase is trying to apply right now. Use it when the conflict is hard to read and you want the original change. It does not resolve anything. It does not continue. It is a view. `git status` names the files. The patch names the change.

```sh
git rebase --show-current-patch
```

People read the conflict markers alone and guess. The patch is the intent. People also expect this flag to work outside a rebase. It is for a stopped rebase or a stopped am. A finished rebase has nothing to show. The message says so. Pair it with the file in your editor. The patch is the "before." The work tree is the "during."

## Rebase commits that were already pushed
Also asked as: rebase and force push; rewritten history; force-with-lease; already pushed; do not rebase main
A rebase creates new commits. If you had pushed the old ones, the remote still has them. A normal push is rejected. `git push --force-with-lease` updates the remote branch to the new commits, and it refuses if someone else pushed in the meantime. A plain `--force` overwrites regardless. Do not force-push a shared branch such as `main`. Rebase feature branches.

```sh
git push --force-with-lease
```

People rebase `main` and force-push it. Everyone else's work is now based on commits that no longer exist. Rebase your own branch. People also `--force` out of habit and clobber a colleague's push. `--force-with-lease` is the narrower hammer. A protected branch on the server rejects the force anyway. That rejection is the server saving you. Pull, or rebase again onto the new tip, instead of forcing harder.

## Stash local changes before rebasing
Also asked as: git rebase --autostash; dirty work tree; cannot rebase; uncommitted changes; stash and rebase
A dirty work tree can block a rebase. `--autostash` stashes your uncommitted changes, runs the rebase, and pops the stash after. Conflicts in the pop are ordinary stash conflicts. Commit or stash yourself if you do not want that automatic. `--autostash` is a convenience, not a license to leave the tree messy.

```sh
git rebase --autostash main
```

People start a rebase with a dirty tree and get an error. Commit, stash, or pass `--autostash`. People also autostash a pile of unrelated edits and the pop conflicts with the replayed work. The stash is still there if the pop fails. `git stash list` shows it. A rebase that aborts still tries to restore the stash. Check `git status` after. Autostash does not stash untracked files unless you configured that. `git status` tells you what is untracked.

## Replay onto the same base to clean up
Also asked as: git rebase -i HEAD~n; rebase last commits; clean up recent commits; HEAD~3; fixup recent
`git rebase -i HEAD~3` opens the last three commits for edit. There is no upstream branch involved. You are replaying those three onto the commit before them. This is the local cleanup. Squash, reword, reorder. The commits before `HEAD~3` are untouched. Count carefully. `HEAD~3` is three back, so three commits are in the list.

```sh
git rebase -i HEAD~3
```

People count the commits wrong and include one they already pushed, then have to force-push a range they did not mean to rewrite. `git log --oneline` first. People also use `HEAD^` and `HEAD~` interchangeably. `HEAD~3` is three first-parents back. A merge in that range has more than one parent, and the count follows the first. An interactive rebase of one commit can still reword it. A range of zero is an error. There is nothing to replay.

## Preserve merge commits
Also asked as: git rebase --rebase-merges; rebase a merge; keep merges; -r rebase; merge commit in history
A plain rebase drops merge commits and replays a linearized history. `--rebase-merges` tries to recreate the merges as it replays. Use it when the branch contains intentional merges you want to keep. It is slower and the conflicts can be harder. A plain rebase is the right tool when you wanted a straight line anyway.

```sh
git rebase --rebase-merges main
```

People rebase a branch that merged `main` into itself, and the merge commit vanishes. That is the default. Pass `--rebase-merges` if the merge mattered. People also expect this flag to avoid conflicts. It does not. It preserves structure. A conflict in a recreated merge stops the rebase like any other. Continue after resolving. If the history is disposable, a plain rebase is simpler. Linear history is the usual goal of rebasing.

## Run a command on each commit
Also asked as: git rebase -x; --exec; test each commit; exec during rebase; run tests while rebasing
`-x command` runs a shell command after each replayed commit. If the command exits non-zero, the rebase stops. This is how you check that every commit builds, not only the tip. The command sees the tree at that commit. It runs in a shell. Quote it.

```sh
git rebase -x 'command' main
```

People `-x` a test that needs a service, and every commit fails for the same environmental reason. The exec is not the place to start a daemon. People also expect `-x` to roll back a commit that fails the test. It stops and waits. You fix the commit, or you `--skip`, or you `--abort`. An exec that changes files leaves the tree dirty for the next commit. Make the command read-only unless you mean to amend.

## What a failed rebase means
Also asked as: rebase exit status; rebase failed; rebase stopped; nothing to do; cannot rebase
Exit 0 means the rebase finished and the branch moved. Exit non-zero with a clean status means the arguments were wrong, or there was nothing to do, depending on the message. A non-zero exit with "could not apply" means the rebase stopped on a conflict. That is waiting, not ruined. `--continue`, `--skip`, or `--abort`. A second rebase while one is in progress is refused.

```sh
git rebase main
```

People see a conflict stop and run `git rebase` again. The message says a rebase is already in progress. Continue or abort. People also treat "up to date" as an error. It is success with no work. A rebase that finishes and loses a commit usually means the commit was empty against the new base and was dropped, or it was skipped. `git log` against the old reflog entry shows the difference. `git reflog` is the list of tips you have stood on. The pre-rebase tip is in there. A force-push you regret is a second force-push back to that tip, if nobody else has pulled.
