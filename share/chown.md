# chown

`chown` changes the owner and group of a file. It does not change the mode. That is `chmod`. It does not move the file. Only root can give a file to another user. A normal user cannot hand a file away, and cannot claim someone else's file. With no operand it prints a short error and exits non-zero. GNU chown from coreutils on Linux is assumed here. macOS `chown` is BSD. It has `-R` and `-h`, and it has no `--reference` or `--from`. BusyBox `chown` usually has `-R` and `-h`. Exit status is 0 when every named file was changed, or was already as requested. It is non-zero when a file is missing, you are not allowed, or a recursive walk cannot enter a directory.

ic form: chown user -- file

The first operand is `user`, `user:group`, or `:group`. Later operands are paths. A name that starts with `-` is a flag unless it comes after `--`. Quote a name with spaces. A glob is expanded by the shell. `user:` with a trailing colon sets the group to that user's login group. `:group` changes only the group. A bare `user` changes only the owner.

## Give a file to a user
Also asked as: chown user file; change owner; change the owner of a file; set owner; chown a file
A bare user name changes the owner and leaves the group. The user must exist. `chown user -- file` is the form. Root can do this to any file. A normal user gets `Operation not permitted`. The file stays where it is. The mode stays as it was. `chown` prints nothing on success.

```sh
chown user -- file
```

People run `chown` on a file they own and expect to reassign it. The kernel does not allow a non-root user to change the owner. That is how setuid and quota are kept meaningful. People also pass a display name. The operand is the account name or the numeric id, not a full name. A missing user is an invalid argument and a non-zero status. The file is unchanged.

## Change the owner and the group
Also asked as: chown user:group; owner and group; chown colon; set both; user and group together
`user:group` sets both. A dot used to mean the same thing and still works on GNU chown. Prefer the colon. The user and the group are looked up separately. One can be numeric. `chown 0:0 -- file` is root and root's group. Both names must exist unless you used a number.

```sh
chown user:group -- file
```

People write `user group` with a space and the group is read as a file name. The separator is a colon, with no spaces. People also expect the group to change because they changed the owner. A bare user leaves the group. The colon form is the one that sets both. A trailing colon, `user:`, is different again. It sets the group to the user's login group, not to "unchanged."

## Change only the group
Also asked as: chown :group; change group only; colon group; chown versus chgrp; set the group
`:group` changes the group and leaves the owner. `chgrp group -- file` is the same change. A user who owns the file may change its group to a group they belong to. Root may set any group. A group you are not in fails for a non-root user. The mode is still unchanged.

```sh
chown :group -- file
```

People `chown group -- file` and the group name is taken as a user. If no such user exists, the command fails. The leading colon is what makes it a group. People also use `chown` where `chmod` was required. Ownership is who, mode is what they may do. A directory's group matters for files created in it when the setgid bit is on. `chown` does not set that bit.

## Use a numeric id
Also asked as: chown uid; numeric owner; chown 1000; user id; no such user numeric
A number is a uid or a gid, not a name. `chown 1000 -- file` sets the owner to uid 1000 even if that id has no account on this machine. `chown 1000:1000` sets both. This is how you fix a file that arrived with foreign ids. A name that is all digits is looked up as a name first on some builds. Prefer the number when you mean the number.

```sh
chown 1000:1000 -- file
```

People see `1000` in `ls -n` and think the file is broken. It is owned by an id this system does not name. `chown` to a real account, or leave it. People also `chown` a name that exists only in a container's passwd. On the host that name may be a different id. The number is the durable value. `ls -n` prints it. `ls -l` prints a name only when the lookup succeeds.

## Change a directory tree
Also asked as: chown -R; recursive chown; chown a folder; chown directory and contents; chown -R user
`-R` walks the directory and changes each file and subdirectory. The directory itself is included. Without `-R`, a directory operand changes the directory inode only. The contents keep their old owner. Symlinks are not followed by default. The link's target is not walked into.

```sh
chown -R user -- dir
```

People `chown` a directory, see the directory's owner change in `ls -ld`, and the files inside are untouched. `-R` is the walk. People also `-R` from `/` and rewrite the system. `--preserve-root` makes a recursive `chown` of `/` fail. It is not the default on GNU chown. Pass it. A walk that cannot enter a subdirectory reports the error and continues with the rest, then exits non-zero. The files already changed stay changed.

## Do not follow a symlink
Also asked as: chown -h; chown a symlink; no-dereference; chown the link itself; --no-dereference
By default `chown` follows a symlink and changes the target. `-h` changes the symlink itself, on a system that allows that. Linux allows it. A dangling symlink can be chowned with `-h` and cannot be chowned without it, because there is no target. `-h` is not recursive by itself. Combined with `-R` it still does not walk through links.

```sh
chown -h user -- file
```

People `chown` a link and then `ls -l` the link. The link's own owner may be unchanged while the target moved. `ls -l` shows the target's owner in the usual listing. `ls -ld` on the link, or `ls -l` in a context that does not follow, shows the link. `-h` is the flag when the link inode is the one you mean. A system that cannot chown a symlink returns an error for `-h`. The target is then untouched.

## Copy the owner from another file
Also asked as: chown --reference; same owner as; copy ownership; reference file; match another file
`--reference=file` takes the owner and group from that file and applies them to the operands. You do not name a user. This is GNU. macOS chown has no equivalent. The reference file is not modified. A symlink as the reference is followed unless you also arranged not to follow it.

```sh
chown --reference=file -- file
```

People pass a user name to `--reference` and it is read as a path. The argument is a file. People also expect the mode to be copied. It is not. `chmod --reference` is the mode copy. A missing reference file fails and the operands are unchanged. Recursive `--reference` with `-R` applies that same owner and group to every file in the walk. It does not copy each file's counterpart.

## Change a file only if it has a given owner
Also asked as: chown --from; conditional chown; only if owned by; from current owner; GNU chown --from
`--from=user:group` changes the file only when the current owner and group match. You can omit one side. `--from=user` matches the owner and any group. A file that does not match is skipped. This is GNU. It is the guard on a recursive run when you mean to reassign one user's files and leave the rest.

```sh
chown --from=user user:group -R -- dir
```

People `-R` without `--from` and every file becomes the new owner, including files that were supposed to stay. `--from` is the filter. A skipped file is not an error. The status is still 0 if the walk succeeded. Verbose mode shows what changed. A typo in `--from` matches nothing and changes nothing, which looks like success. Check a file you expected to move.

## See what changed
Also asked as: chown -v; chown -c; verbose chown; print changes; chown --changes
`-v` prints a line for every file it processes. `-c` prints a line only when the owner or group actually changed. A file that already had the requested owner is silent under `-c` and reported under `-v`. The lines go to stdout. Errors still go to stderr. Neither flag changes what is modified.

```sh
chown -c user -- file
```

People add `-v` and think it is a dry run. The change already happened. There is no dry-run flag. List with `ls -l` first if you are unsure. `-c` on a recursive run is the way to see which files were really rewritten. A quiet success prints nothing. Silence from `-c` means nothing needed to change, or you pointed at the wrong tree. The status is the other check.

## Quiet the errors
Also asked as: chown -f; chown --silent; suppress errors; chown quiet; ignore missing
`-f` suppresses most error messages. The status is still non-zero when a change could not be made. A missing file, a permission error, and a bad user still fail the command. You just do not see the reason. This is a poor default. It is useful in a script that handles the status itself.

```sh
chown -f user -- file
```

People pass `-f` and then cannot tell why a file is unchanged. Drop it and read the error. `-f` does not make a forbidden change succeed. Root still cannot chown a file on a filesystem mounted read-only. A recursive `-f` can hide a subdirectory you were not allowed to enter, and the exit status is the only signal. Check it.

## Preserve root on a recursive run
Also asked as: chown --preserve-root; chown of slash; do not chown /; recursive root guard; --no-preserve-root
`--preserve-root` makes `chown -R` refuse to operate on `/`. Without it, GNU chown will recursively change the root directory if you ask it to. That is almost never what you want. The flag is GNU. Pass it whenever the path is a variable that might expand to `/`. `--no-preserve-root` is the explicit opposite.

```sh
chown -R --preserve-root user -- dir
```

People `chown -R user $dir` and an empty `dir` becomes `chown -R user`, which is a usage error, or worse a variable that expanded to `/`. Quote the path and pass `--preserve-root`. The flag does not protect other important directories. `/usr` is not special. A relative path that reaches `/` through `..` is still a walk. Know the directory you named.

## Traverse a symlink to a directory
Also asked as: chown -H; chown -L; chown -P; follow symlink directory; recursive and links
With `-R`, the default is `-P`. Do not follow any symlink. `-H` follows a symlink on the command line if it points at a directory, and not symlinks found during the walk. `-L` follows every symlink to a directory. The last of `-H`, `-L`, and `-P` wins. These flags do nothing useful without `-R`.

```sh
chown -R -H user -- dir
```

People `-R` through a tree of symlinks and the targets stay as they were. That is `-P`. Use `-H` or `-L` only when the targets are the files you mean to reassign. `-L` can leave the tree and chown files outside it, including through a link to `/`. That is the dangerous one. A symlink to a file, not a directory, is changed as a symlink or as a target according to `-h`, not according to `-L`.

## Fix a tree after a copy
Also asked as: chown after cp; copy preserved owner; rsync owner; files owned by the wrong user; restore ownership
`cp -a` and `rsync -a` preserve the owner when they can, which for root means the source ids come across. On a different machine those ids are different people. `chown -R user -- dir` is the reset. `--from` limits it to the foreign id. A copy done as a normal user already owns the files as that user. `chown` is then unnecessary, and it fails if they try to reassign.

```sh
chown -R user:group -- dir
```

People `chown` the archive instead of the extracted tree. The archive file changes, the members do not. Extract first, then chown the tree. People also chown a bind mount and affect the original. A mount is the same inodes. Ownership is on the inode, not on the path. Two links to one file share the change. `chown` on either name is enough.

## What you need to be allowed to do
Also asked as: chown permission denied; operation not permitted; only root can chown; cannot chown; who may change owner
Changing the owner requires the capability, which for a normal login means root. Changing the group, if you own the file and you are a member of the new group, does not. A directory you cannot search is a walk error under `-R`. A file on a read-only filesystem cannot be changed. An NFS root squash makes root on the client nobody on the server, and `chown` fails.

```sh
chown user -- file
```

People sudo the shell and not the command, or the reverse, and the error is `Operation not permitted` from the unprivileged side. The command that opens the inode needs the privilege. People also chown a path they can write. Write permission is not ownership. You can edit a group-writable file you do not own, and you still cannot `chown` it. The error is the distinction.

## What a failed chown means
Also asked as: chown exit status; chown exit code; chown failed; invalid user; cannot access
Status 0 means every file was set, or already had that owner and group. Status non-zero means at least one file could not be changed. An unknown user is non-zero and nothing is changed. A missing file is non-zero. A recursive run that hit one unreadable directory is non-zero, and the earlier files are already done. `-v` or `-c` shows how far it got. `-f` hides the reason and keeps the status.

```sh
chown user -- file
```

People check `ls` on the directory and not the status. A partial recursive run looks applied. Read the errors. An invalid user fails before any change. A permission error fails per file. Under `set -e`, a failing `chown` aborts a script, which is the right default for a setup script. A later command that assumes the new owner will otherwise run as the wrong user and fail further on.
