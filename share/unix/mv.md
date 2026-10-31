# mv

`mv` renames a file or directory. On the same filesystem that is a new directory entry for the same inode, not a copy of the bytes. Across filesystems it copies and then removes the source, and a failure in the middle can leave both. The source does not stay, on a successful move. `mv` is not `cp`. With more than two operands, the last must be an existing directory, and each earlier name is moved into it. GNU mv from coreutils on Linux is assumed here. It overwrites a destination file without asking, unless you pass `-i` or `-n`. macOS mv understands `-i`, `-n`, and `-f`, and it has no `-t`. BusyBox mv usually has `-f` and not the GNU backup flags. Exit status is 0 when every source was moved, or was skipped by `-n`. It is non-zero when a source is missing or a destination could not be written. A skipped `-n` is not a reliable failure. Check the source.

Basic form: mv -- file file

Two operands are the old path and the new path. A name that starts with `-` is a flag unless it comes after `--`. The shell expands globs before `mv` sees them. Quote a name with spaces. If the new path is an existing directory, the file is moved into it and keeps its base name. `mv` does not create parent directories.

## Rename a file
Also asked as: how to rename a file; mv old new; move a file; change the name; mv source dest
With two operands, and the second not a directory, `mv` gives the source the new name. The inode stays if the destination is on the same filesystem. The source name is gone. If the destination name already exists and is a file, it is replaced. There is no prompt unless `-i` is set or an alias added it. A missing source is an error.

```sh
mv -- file file
```

People `mv` and then look for both names. A successful rename leaves one. People also expect timestamps to change. A same-filesystem rename keeps the inode, so the modification time of the data does not change. The directory's time changes, because a name was added and removed. Across filesystems the destination is a new file, and the times follow the copy `mv` did internally. `-v` prints the rename. It does not ask.

## Move a file into a directory
Also asked as: mv into a folder; last argument is directory; move several files; mv files dir; keep the basename
If the last operand is an existing directory, every earlier operand is moved into it. The base name stays. `mv -- file dir` writes `dir/file` and removes `file`. Several sources are legal only in this form. A destination that does not exist is a new name, not a directory `mv` will create.

```sh
mv -- file file dir
```

People write `mv file dir/newname` and `dir` is missing. That fails. `mv` will not make the parent. `mkdir` first. A trailing slash on a destination that does not exist does not mean "create a directory." It means you expected a directory, and GNU mv can refuse to treat a non-directory that way. A glob that matches one file and a missing destination renames to that literal destination. Print the expansion before you move a glob.

## Rename a directory
Also asked as: mv a directory; rename a folder; move a tree; mv dir dir; directory rename
A directory is renamed the same way as a file. On the same filesystem the tree stays on the same inode and the contents are not walked. That is cheap and atomic. Across filesystems `mv` copies the tree and deletes the source, which is not atomic and can fail part way. You do not need `-r`. `mv` of a directory is already the directory.

```sh
mv -- dir dir
```

If the destination exists and is a directory, the source directory is moved inside it. You get `dir/dir`, not a renamed top. People run that twice and nest the tree. An existing destination file cannot be replaced by a directory. `mv` refuses. A directory that is a mount point is not a normal rename onto another filesystem. Move the contents, or unmount. Same-filesystem rename does not change the inodes of the files inside.

## Do not overwrite an existing file
Also asked as: mv -n; no clobber; do not overwrite; mv --no-clobber; skip if dest exists
`-n` does not overwrite an existing destination. The source stays where it was. On GNU mv 9 the status can still be 0 when the file was skipped. Do not use the status as proof of a move. `-i` asks instead. `-f` does not ask. If more than one of `-i`, `-f`, and `-n` appears, the last one wins.

```sh
mv -n -- file file
```

People pass `-n` and delete the source in the next line of the script. The source may still be the only copy. Test that the destination exists and the source is gone before you treat the move as done. `-n` does not merge directories. A destination directory still means "move into," and names inside it are what `-n` protects. macOS mv has `-n`. BusyBox often does not.

## Ask before overwriting
Also asked as: mv -i; interactive move; prompt before overwrite; confirm replace; mv alias -i
`-i` asks before replacing an existing destination. A destination that does not exist does not prompt. Answering no leaves both names where they were. An alias `mv -i` is why a terminal asks and a script does not. Scripts do not expand aliases unless you turned that on. `-f` after `-i` turns the prompt off.

```sh
mv -i -- file file
```

People expect `-i` to confirm every move, including moves that only create a new name. It confirms overwrites. A "no" is easy to miss in a long list. The file you thought you moved is still in the source directory. `-i` is not a trash can. Answering yes replaces the destination. There is no backup unless you also passed `-b`.

## Back up the destination
Also asked as: mv -b; mv --backup; backup before overwrite; mv suffix; numbered backups
`-b` renames an existing destination out of the way before the move. The usual suffix is `~`. `--backup=numbered` keeps `file.~1~`, `file.~2~`, and so on. This is GNU. macOS mv does not have it. The backup is a rename of the old destination, on the same filesystem. It is not a copy into a backup folder you named.

```sh
mv -b -- file file
```

People use `-b` and then cannot find the old file. It is beside the new one, with a tilde. A second move makes another backup, or overwrites the simple backup, depending on `--backup`. `numbered` is the form that keeps history. `-b` does not apply when the destination is missing. Nothing was replaced. It does not ask. Combine it with `-i` if you want both a prompt and a backup.

## Move only when the source is newer
Also asked as: mv -u; update move; move if newer; mv --update; skip older source
`-u` moves when the destination is missing, or when the source is newer than the destination. An older source is left in place. The comparison is the modification time, not the contents. This is GNU. A skipped file is still in the source. A moved file is not.

```sh
mv -u -- file file
```

People use `-u` to publish a tree and expect old extras in the destination to disappear. They do not. `-u` only decides whether this source replaces this destination. Equal timestamps do not move the file. A same-filesystem rename keeps the timestamp, so a second `-u` of the same file does not see a newer source. Clock skew between two machines makes "newer" lie. `rsync` is the tool when the rule is more than one timestamp.

## Name the destination directory with a flag
Also asked as: mv -t; target directory; mv --target-directory; xargs mv; destination first
`-t` names the directory that receives the files. The other operands are sources. This is the form `xargs` can call, because `xargs` appends paths. `-T` forces the two-operand form, so an existing directory is not treated as a container. Both are GNU. macOS mv does not have `-t`.

```sh
mv -t dir -- file file
```

People build `mv $files "$dest"` and a missing dest turns the last file into the destination name. `-t` fails if the directory is missing, which is the safer failure. `-T` fails if you meant a rename and the destination is a directory, instead of nesting the file inside it. Use `-T` when "into" would be the bug. Use `-t` when every other argument is a source.

## Replace a directory with a file, or not
Also asked as: mv -T; no target directory; cannot overwrite directory; mv file onto directory; destination is a directory
If the destination exists and is a directory, a normal `mv` puts the source inside it. `-T` disables that. The destination is treated as the new name of the source. `mv -T -- file dir` fails if `dir` is a directory, instead of creating `dir/file`. That is the guard for a script that always has two paths and never means "into."

```sh
mv -T -- file file
```

People lose files by moving onto a path that turned out to be a directory. The file is inside it, under the old base name. `-T` makes that a failed move and leaves the source. It does not delete the directory. Replacing a non-empty directory with a file is not what `mv` does. Remove the directory first if that is really the operation. An empty directory is still a directory.

## Move across filesystems
Also asked as: mv cross device; invalid cross-device link; mv copies then deletes; different filesystem; mv between disks
A rename on the same filesystem does not copy bytes. A move onto another filesystem cannot be a rename. GNU mv copies the file, then removes the source. The copy can take a long time. If the copy fails, the source stays. If the copy succeeds and the remove fails, you have two files. A directory across filesystems is copied as a tree and then removed.

```sh
mv -- file file
```

People interrupt `mv` and find a partial destination and a full source, or a full destination and a source that is already gone. Check both before you delete anything by hand. `df -h -- file file` shows whether the two paths are the same filesystem. Same device is the cheap rename. `mv` does not print which one it did unless you pass `-v` and read it. Permissions on the destination directory still apply. The inode's owner is preserved when the copy can preserve it.

## See each rename
Also asked as: mv -v; verbose move; print renames; mv tell me what it did; explain what is being done
`-v` prints a line for each rename. It does not ask. It does not change what is moved. The line says the old path and the new path. It goes to standard output. Errors still go to standard error. On a cross-filesystem move the verbose line still looks like a rename. The copy already happened.

```sh
mv -v -- file file
```

People turn on `-v` as a preview. It reports the move it just did. A name that failed is an error line, not a verbose success line. `-v` does not imply `-i`. A script that parses `-v` is parsing a diagnostic. The status is still the result. A skipped `-n` may print nothing for that file. Silence is not "moved."

## Move a name that starts with a dash
Also asked as: mv -- -file; mv ./-file; invalid option; file named -rf; leading dash
An operand that starts with `-` is parsed as a flag. `mv` fails with an illegal option, or overwrites something you did not mean, and the file is still there. `--` ends options. The next word is a path. `./-file` is the other way to make the name not look like a flag.

```sh
mv -- -file file
```

Flags come first, then `--`, then paths. `--` between two paths does not apply to a glob you expanded yourself if the glob sits before it. Put the glob after `--`. A destination that starts with a dash has the same problem. `mv -- file -file` is a rename to that name. Without `--`, `-file` is a bad flag and nothing moves.

## What a failed mv means
Also asked as: mv exit status; mv exit code; cannot move; permission denied; mv failed; source still there
`mv` exits 0 when it moved every operand it was willing to move. A missing source is non-zero. A permission error on the directory that holds the name is non-zero. Overwriting a file you may write is success. `-n` can exit 0 and leave the source in place. The source path is the check. `-v` shows what moved. It does not change the status.

```sh
mv -- file file
```

People delete what they think is a duplicate after a failed `mv` and remove the only copy. If the status is non-zero, the source is still the file to trust, unless a cross-filesystem move already printed that the copy finished. `set -e` aborts a script on a failed `mv` unless the command is in a conditional. A later command that assumes the new path exists is the bug a bad status was trying to stop.
