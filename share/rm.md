# rm

`rm` unlinks files. It removes a directory entry, not necessarily the bytes on disk. A file that still has another hard link, or that a process still has open, keeps its data until the last link and the last file descriptor are gone. `rm` does not remove directories unless you pass `-r` or `-d`. It does not follow a symlink and delete the target. It deletes the symlink. With no file operand it does not read standard input, and it does not default to the current directory. GNU rm from coreutils on Linux is assumed here. macOS rm understands `-r`, `-f`, `-i`, and `-d`. It has no `-I` and no `--one-file-system`. BusyBox rm usually has `-r` and `-f` and may not prompt the way GNU does. Exit status is 0 when every named operand was removed, or was already absent and you passed `-f`. It is non-zero when a named file is missing, is a directory and you did not ask to remove directories, or could not be unlinked. Refusing a prompt can still leave status 0 on GNU rm. The file is the thing to check.

Basic form: rm -- file

Operands are paths. A name that starts with `-` is a flag unless it comes after `--`, or unless you write `./-file`. The shell expands globs before `rm` sees them. `rm *` is every non-hidden name in the directory, not a request `rm` interprets. Quote a name that contains spaces. There is no trash directory. A successful `rm` does not move the file.

## Remove a file
Also asked as: how to delete a file; rm a file; unlink a file; remove one file; rm does not ask
With no flags, `rm` unlinks each file you name and prints nothing when it works. It does not ask. A directory is rejected with "Is a directory" and status non-zero. A missing file is "No such file or directory" and status non-zero. The data may still be recoverable from the disk until something reuses the blocks. `rm` does not shred them.

```sh
rm -- file
```

People expect a confirmation because an alias added `-i`. The `rm` on `PATH` does not prompt. `\rm` or `command rm` skips the alias. A symlink operand removes the link. The file it pointed at stays. A hard link removes one name. The file stays until the last name is gone. `rm` of a path you cannot write fails even if you own the file, when the directory that holds the name is not writable.

## Remove a name that starts with a dash
Also asked as: rm -- -file; rm ./-file; invalid option; file named -rf; leading dash
A operand that starts with `-` is parsed as a flag. `rm` then fails with an illegal option, or does something you did not mean, and the file is still there. `--` ends option parsing. The next word is a path. `./-file` is the other way to make the same name not look like a flag.

```sh
rm -- -file
```

```sh
rm ./-file
```

Use `--` when you want the name as stored. Use `./` when the next command in a script does not understand `--`. Do not put `--` between `-r` and a directory if you still need `-r` to apply. Flags come first, then `--`, then paths. A glob that expands to a name starting with a dash has the same bug. `--` before the glob does not protect names the glob produces. Pass the names after `--` from a list you control.

## Remove an empty directory
Also asked as: rm -d; rmdir; remove empty directory; rm directory without -r; delete empty folder
`-d` removes an empty directory. It does not remove a directory that still has names in it, and it does not walk. GNU `rmdir` does the same job and only that job. Without `-d` or `-r`, `rm` on a directory fails with "Is a directory."

```sh
rm -d -- dir
```

People pass `-r` to remove a folder they believe is empty. If it is not empty, `-r` deletes the contents. `-d` refuses. `.` and `..` do not count as contents. A directory that contains only hidden names is not empty, and a glob that skipped those names does not make it empty. `-d` on a file fails. It is the directory form, not a quiet flag.

## Remove a directory and everything in it
Also asked as: rm -r; rm -rf; recursive delete; remove a folder; rm -R; delete directory tree
`-r` and `-R` remove the directory, its files, and its subdirectories. A symlink is removed as a symlink. `rm` does not follow it into the target. A directory that is a mount point is a different filesystem. GNU rm still descends into it unless you pass `--one-file-system`. The walk is depth-first. Contents go before the directory.

```sh
rm -r -- dir
```

`-r` does not mean "this directory only if I typed it right." A wrong path, a typo that expands, or a variable that is empty can select a different tree. `rm -r -- dir` with `dir` empty becomes `rm -r --`, which removes nothing and is the lucky case. `rm -r $dir` with `dir` empty becomes `rm -r` and then whatever else is on the command line. Quote `"$dir"`. There is no undo.

## Force a remove and ignore a missing file
Also asked as: rm -f; rm --force; ignore nonexistent; do not prompt; rm -f missing file
`-f` does not prompt, and it does not complain when the file is already gone. The status is 0 if the only problem was a missing operand. `-f` does not give you permission you lack. A directory you cannot write still fails. `-f` does not imply `-r`. A directory operand without `-r` is still "Is a directory."

```sh
rm -f -- file
```

People write `rm -rf` out of habit for a single file. The `-r` is what makes a directory operand dangerous, not the `-f`. `-f` is the flag that makes a cleanup script succeed when the file was already removed. It also hides a typo, because a misspelled name is "already gone" and the real file stays. Use `-f` when absence is success. Leave it off when a missing file means the script is in the wrong place.

## Ask before each removal
Also asked as: rm -i; interactive remove; prompt before delete; confirm each file; rm -I
`-i` asks before every removal. GNU `-I` asks once, and only when you named more than three files or you used `-r`. `-f` turns prompting off and overrides a previous `-i`. An alias `rm -i` is why an interactive `rm` asks and a script's `rm` does not. Scripts do not use the alias unless the shell was told to expand aliases.

```sh
rm -i -- file
```

A "no" answer leaves the file. On GNU rm that refusal can still exit 0, so a script cannot use the status to see what you declined. `-i` on a recursive directory asks for the directory and for the files under it, which is a lot of prompts and not a review of the tree. It is not a trash can. Answering yes unlinks. `-I` is the lighter GNU prompt. macOS rm has `-i` and not `-I`.

## Do not remove the root directory
Also asked as: rm -rf /; preserve root; rm --preserve-root; do not delete slash; --no-preserve-root
GNU rm refuses `rm -r /` unless you pass `--no-preserve-root`. The default is `--preserve-root`. The check is the operand `/`, not "a path that might be important." `--preserve-root=all` also rejects an argument that is on a different device from its parent. That form is GNU.

```sh
rm -r --preserve-root -- dir
```

People hear that `rm` protects them and then run `rm -rf /tmp/old /` with a stray slash, or `rm -rf $dir/` when `dir` is empty. The protection is specifically the root directory as an operand. It does not stop `rm -rf /home` or `rm -rf /*`. `--no-preserve-root` is how you turn the check off. You do not need it for a normal directory. macOS rm does not implement this GNU check. Do not rely on it there.

## Stay on one filesystem
Also asked as: rm --one-file-system; do not cross mounts; rm -r skip other filesystems; recursive stay on filesystem
`--one-file-system` tells recursive `rm` not to enter a directory that is on a different filesystem from the operand you named. A mount point inside the tree is left alone. The flag is GNU. It does not apply to a non-recursive remove. It does not unmount anything.

```sh
rm -r --one-file-system -- dir
```

People use it as "do not delete more than this folder" and still lose every file on that filesystem under `dir`. The flag only skips other devices. A bind mount of the same filesystem is still the same filesystem and is still entered. Without the flag, `rm -r` on a tree that contains a mount point deletes the contents of that mount. Check `df -h -- dir` if you are not sure what is mounted under it.

## See each name as it goes
Also asked as: rm -v; verbose remove; print removed files; rm tell me what it deleted
`-v` prints a line for each file it removes. It does not ask. It does not change what is removed. A recursive remove prints the contents and the directories. The lines go to standard output. Errors still go to standard error.

```sh
rm -v -- file
```

People turn on `-v` and think the list is a preview. It is a report after the unlink. A name that failed is an error, not a verbose success line. `-v` on a large tree is a lot of output and does not make the command safer. Pair it with a command you already trust, or print the names with `find` before you remove anything.

## Remove files found by a search
Also asked as: find -delete; find -exec rm; remove matching files; rm from find; do not pipe find to rm
`find` can unlink what it walked. GNU and macOS `find` have `-delete`, which implies a depth-first walk and does not spawn `rm`. `-exec rm -f -- {} +` is the portable form. A pipe from `find` to `xargs rm` without `-print0` and `xargs -0` splits names on spaces.

```sh
find dir -type f -name 'pattern' -delete
```

```sh
find dir -type f -name 'pattern' -exec rm -f -- {} +
```

Use `-delete` on GNU or macOS find when the test is simple. Use `-exec rm` when you need `-f` or the find is BusyBox. Test the `find` with `-print` first. `-delete` is an action. A wrong test deletes the wrong files and exits 0 if the unlinks worked. Do not add `-delete` on the same run you are still editing the expression. `rm` itself has no name pattern. The glob is the shell's, or `find`'s.

## Remove only the files a glob matched
Also asked as: rm *.log; glob delete; rm star; hidden files not matched; nullglob
The shell expands `*.log` before `rm` runs. `rm` receives names, not a pattern. A name that starts with a dot is not matched. A directory that matches is not removed unless you also passed `-r`. If nothing matches, bash leaves the literal `*.log`, `rm` looks for that file, and the status is non-zero.

```sh
rm -f -- *.log
```

`nullglob` makes an unmatched pattern disappear, so `rm -f -- *.log` becomes `rm -f --` and removes nothing. `failglob` makes the unmatched pattern an error before `rm` runs. People write `rm -rf *` in a directory they have not `cd`'d into and delete the script's working directory. Print the glob first, or use `find` with `-name`. `-f` hides the "no match" error. It does not hide a match you did not expect.

## Remove a file you do not have write permission on
Also asked as: rm permission denied; file not writable; directory write bit; cannot remove file; rm owned file
Unlink checks the directory that contains the name, not the write bit on the file. A read-only file in a directory you can write is removed. GNU rm asks before removing a read-only file when standard input is a terminal, unless you passed `-f`. In a script there is no question. A directory you cannot write produces "Permission denied" and a non-zero status.

```sh
rm -f -- file
```

People `chmod u+w` the file and still cannot remove it, because the directory is the thing that is not writable. People also expect the file's mode to protect it from `rm`. It does not, if the directory permits the unlink. The sticky bit on a directory changes that rule. In `/tmp`, you can remove a name you own, not a name owned by someone else. `rm` does not override the sticky bit.

## Tell rm from rmdir
Also asked as: rmdir; rm -d versus rmdir; remove empty folder; rmdir not empty; rm or rmdir
`rmdir` removes empty directories and nothing else. It has no recursive mode. `rm -d` is the same idea with `rm`'s flags. `rm -r` removes a directory that is not empty. Use `rmdir` when a non-empty directory should be an error. Use `rm -r` only when the contents are meant to go.

```sh
rmdir -- dir
```

```sh
rm -r -- dir
```

Use the first on a folder that should already be empty. Use the second when you mean the tree. `rmdir` on a file fails. `rmdir` on a non-empty directory fails and leaves the contents. That failure is the point. People alias `rm` to `rm -r` and then `rmdir`'s caution is gone for every name. A script should call the command it means.

## What a failed rm means
Also asked as: rm exit status; rm exit code; cannot remove; rm failed; status in a script
`rm` exits 0 when it removed every operand. With `-f`, an operand that was already absent also counts as success. Without `-f`, a missing file is non-zero. A directory without `-r` or `-d` is non-zero. A permission error is non-zero. GNU rm can exit 0 when you answer "no" to `-i`. The file is still there.

```sh
rm -f -- file
```

People check the status after `rm -i` and treat 0 as "deleted." Check the path. People also ignore the status of `rm -r` in a cleanup and continue as if the tree were gone. A non-zero status means at least one name survived. `-v` shows what was removed. It does not turn a failure into success. `set -e` aborts a script on that status unless the command is in a conditional.
