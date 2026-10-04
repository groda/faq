# rsync

`rsync` copies files, locally or to another host. It compares the source to the destination and sends the difference when it can. It does not rename in place the way `mv` does. A trailing slash on the source means the contents of that directory. No trailing slash means the directory itself. With one operand it does not read a list from standard input unless you pass `--files-from`. The rsync 3 client on Linux is assumed here. macOS often ships an old rsync, or openrsync, and lacks `--info` and some archive extras. BusyBox does not include this client. Exit status is 0 when the transfer finished cleanly. It is non-zero when the syntax is wrong, a file vanished, or some files could not be copied. Partial success is not 0. The common partial-transfer status is 23.

Basic form: rsync -a -- dir dir

Operands are a source and a destination. A remote operand is `host:path` or `user@host:path`. The shell still expands globs locally, before `rsync` sees them. Quote a remote path that contains spaces or a glob you meant the remote side to see. `--` before a local path stops a leading dash from looking like a flag. `-a` is the usual copy. It does not imply `--delete`.

## Copy a directory
Also asked as: rsync a folder; rsync -a; archive mode; copy a tree; rsync local
`-a` is archive mode. It is `-rlptgoD`. Recursive, copy symlinks as symlinks, keep permissions, times, group, owner where allowed, and keep device files if you are allowed to create them. It does not keep hard links, ACLs, or xattrs. Those are `-H`, `-A`, and `-X`. The source is unchanged. The destination is created if it is missing.

```sh
rsync -a -- dir dir
```

People add `-a` and expect a mirror. Extra files already in the destination stay. `--delete` is the flag that removes them. Owner is preserved only if the receiver can set it. A normal user gets their own ownership and a warning, and the transfer can still exit non-zero. `-a` does not compress. `-z` does. Across the network, `-a` alone sends the file data uncompressed.

## Copy the contents, not the directory
Also asked as: rsync trailing slash; rsync dir/ dir; contents versus directory; why a nested folder; slash on the source
A source with a trailing slash copies the contents of the directory into the destination. A source without one copies the directory, so you get `dest/dir`. The slash is syntax on the source path. A slash on the destination does not mean the same thing. It only says the destination should be a directory.

```sh
rsync -a -- dir/ dir
```

```sh
rsync -a -- dir dir
```

Use the first when `dir` should end up with the files that are inside the source. Use the second when you want a subdirectory named `dir` created underneath. People run the no-slash form twice and get `dir/dir`. People quote a variable and drop the slash they typed next to it, because `"$src/"` is the slash and `"$src"` is not. The difference is one character, and it changes the tree.

## See what would be copied
Also asked as: rsync -n; rsync --dry-run; preview rsync; do not transfer; what would rsync change
`-n` and `--dry-run` list the work and do not write it. Combine with `-v` or you get a summary and no names. Itemized output, `-i`, shows why each name was chosen. A dry run still connects to the remote host and still reads the file lists. It does not upload the file data.

```sh
rsync -ani -- dir/ dir
```

People trust a dry run that omitted `--delete` and then add `--delete` on the real run. The preview did not show the deletions. Put the same flags on both commands. A dry run can still fail with a login error or a missing source. Status non-zero means it did not finish the preview. It does not mean "changes pending." `-n` does not imply `-v`.

## Delete files the source no longer has
Also asked as: rsync --delete; mirror a directory; remove extra files; make dest match source; rsync delete extras
`--delete` removes files in the destination that are not in the source. It runs against the destination of this transfer. A trailing slash still decides what "the source tree" is. `--delete` is not in `-a`. A dry run with `-n` and `--delete` shows the deletions if you also asked for verbose output. Without a preview, the first real run deletes.

```sh
rsync -a --delete -- dir/ dir
```

People point the source at an empty directory, or a failed mount, and `--delete` empties the destination. Check the source before you add the flag. `--delete-after` deletes after the transfer, so a failed copy is less likely to leave the destination already emptied. The default `--delete` can remove destination files during the run. Excluded files are protected from deletion only if you also pass `--delete-excluded`'s opposite carefully. By default, excluded names are not deleted. `--delete-excluded` deletes them too. Do not add that unless you mean it.

## Skip names that match a pattern
Also asked as: rsync --exclude; exclude a directory; rsync filter; skip node_modules; --exclude-from; do not copy a glob
`--exclude` takes a pattern. A matching directory is not descended. Quote the pattern, or the shell expands it locally. `--exclude-from` reads patterns from a file, one per line. The pattern is an rsync pattern, not a full regular expression. `*.o` matches in any directory. `/foo` anchors at the root of the transfer.

```sh
rsync -a --exclude 'pattern' -- dir/ dir
```

People exclude `dir` and still see it, because the pattern was anchored wrong. A leading slash is the transfer root, not the filesystem root. `--exclude` does not delete a copy already at the destination. It only stops that name from being updated. `--delete` will not remove an excluded name unless you pass `--delete-excluded`. A later `--include` does not override an earlier exclude. The first matching rule wins.

## Copy to another host
Also asked as: rsync over ssh; rsync user@host; remote copy; rsync -e ssh; push files to a server
A remote path is `user@host:path`. A colon with no slash is a path relative to the remote home directory. A colon and a slash is an absolute path. The transport is SSH unless you name a daemon URL. `-e ssh` makes the remote shell explicit. Your SSH config and keys are used. The remote host needs an `rsync` on its path.

```sh
rsync -a -- dir/ user@host:dir/
```

People write `host:/dir` and rsync looks up a host named `host` and a path `/dir`. The colon is the separator. A space around the colon splits the operand. Quote the remote operand if it contains spaces. A remote `rsync` older than the client can refuse archive flags. The error is a protocol line, not a copy. Pull is the same command with the remote path as the source. The trailing-slash rule still applies to whichever side is the source.

## Show progress
Also asked as: rsync -P; rsync --progress; progress bar; --partial; resume a copy; rsync -v
`-P` is `--partial --progress`. `--progress` prints per-file progress on a terminal. `--partial` keeps a partially transferred file, so a rerun can continue it. Without `--partial`, a killed transfer deletes the incomplete destination file. `-v` lists names. It is not a progress bar. `--info=progress2` is a whole-transfer line on rsync 3, and old macOS rsync does not have it.

```sh
rsync -aP -- dir/ dir
```

People pass `-v` and think a stall is a hang. A large file shows one name until it finishes, unless `--progress` is on. `--progress` to a file or a cron job prints a lot of carriage returns. Leave it off in a script. `--partial` leaves files that are not the full source. Do not treat a `.` file in the destination as finished if the run was interrupted. A rerun with the same flags resumes. A rerun without `--partial` starts that file over.

## Compress the transfer
Also asked as: rsync -z; rsync compress; slow link; --compress; do not compress already compressed
`-z` compresses the file data on the wire and decompresses it at the receiver. It does not produce a zip file. The destination is a normal copy. Compression costs CPU. It helps on a slow link and hurts on a fast one, or on files that are already compressed. `-a` does not include `-z`.

```sh
rsync -az -- dir/ user@host:dir/
```

People add `-z` for a local copy and only spend CPU. Local copies do not need it. A daemon transfer and an SSH transfer both honor `-z`. SSH compression is a different knob, in the SSH config. You do not need both. `-z` does not change `--delete` or the trailing slash. A partial file kept by `--partial` is uncompressed on disk. The compression is only the stream.

## Skip files that are already newer
Also asked as: rsync -u; rsync --update; do not overwrite newer; skip newer dest; update only
`-u` skips a destination file that is newer than the source. It does not skip a destination that merely differs. Size and time still decide, unless you passed `-c`. A missing destination is still copied. `-u` is not `--ignore-existing`. That other flag skips any file that already exists, whatever the time.

```sh
rsync -au -- dir/ dir
```

People use `-u` as a backup safety and then never refresh a file whose timestamp was set in the future. The destination stays wrong. `-u` does not imply `--delete`. Old extras remain. A same-size file with an older source timestamp is skipped even if the bytes differ. Add `-c` if the timestamp is not a trustworthy signal. `-c` reads both files. It is slow on a large tree.

## Compare checksums instead of size and time
Also asked as: rsync -c; rsync --checksum; ignore timestamps; content compare; files differ but times match
`-c` decides whether to send a file by checksum, not by size and modification time. A file whose time changed and whose bytes did not is not resent. A file whose bytes changed and whose time did not is resent. The sender and the receiver both read the file. It is the accurate compare, and it is expensive.

```sh
rsync -ac -- dir/ dir
```

People add `-c` to every cron copy of a large tree and the job reads every byte on both sides before it sends anything. Use it when timestamps are wrong, not as a default. `-c` does not imply `--delete`. It does not fix a trailing-slash mistake. Checksums are of the file contents rsync sees. A file changing during the run can still be copied torn. `--checksum` is the long name. It is not `--ignore-times`. `-I` ignores times and sends every size-differing decision as "send."

## Keep the partial tree and the permissions you care about
Also asked as: rsync -H; hard links; rsync -A; ACLs; rsync -X; xattrs; archive is not everything
`-a` does not copy hard links as links. Two names become two files. `-H` preserves hard links within the set being copied. `-A` copies ACLs. `-X` copies xattrs. The receiver and the filesystem have to support them. A warning and a non-zero status are normal when they do not. `-aHAX` is the form people mean by a full archive.

```sh
rsync -aHAX -- dir/ dir
```

People use `-a` on a tree of hard-linked backups and double the disk. `-H` has to see the links in one run. Two separate `rsync` commands do not recreate a link between them. ACLs on macOS and ACLs on Linux are not the same. `-A` talks to the platform rsync was built for. A daemon that was not started with ACL support drops them. The status is the thing to read. A clean-looking tree can be a failed attribute copy.

## Limit bandwidth
Also asked as: rsync --bwlimit; limit rate; do not saturate the link; bandwidth cap; kilobytes per second
`--bwlimit` caps the transfer in kilobytes per second. `--bwlimit=1000` is about a megabyte per second. It does not cap the initial file-list chatter perfectly, and it does not change SSH's own overhead. It is a polite cap, not a firewall rule. The copy still finishes. It just waits between writes.

```sh
rsync -a --bwlimit=1000 -- dir/ user@host:dir/
```

People pass `1000` and think it means bits, or think it means the whole machine. It is this `rsync`, in kilobytes per second. A second `rsync` beside it has its own cap. `--bwlimit` does not imply `-z`. On a fast disk and a local copy the cap still sleeps. Leave it off locally. A too-small cap looks like a hung transfer in `--progress`. The numbers are still moving.

## Copy only files that already exist at the destination
Also asked as: rsync --existing; update in place only; do not create new files; skip new names; existing files only
`--existing` copies only when the destination name is already there. New files in the source are skipped. It does not delete. It does not imply `-u`. A file that exists and is older is updated. This is the opposite of a full mirror. It is how you refresh a published subset without adding names.

```sh
rsync -a --existing -- dir/ dir
```

People want "do not add files" and also "remove files the source lost." `--existing` does not remove. `--delete` does, and it would remove destination files that are not in the source, which is a different request. `--ignore-existing` is the other skip. It never updates a name that is already there. `--existing` updates them. Pick the one that matches the sentence. They are easy to swap.

## Take the list of files from a file
Also asked as: rsync --files-from; copy these paths; rsync a list; null separated list; --from0
`--files-from` reads source paths from a file, one per line, relative to the source directory you named. `-` means standard input. `--from0` reads NUL-separated names, which is the safe form with `find -print0`. The source directory is still the first path operand. The names in the list are under it. This is how you copy a subset without a shell glob.

```sh
rsync -a --files-from=file -- dir/ dir
```

People put absolute paths in the list and a source of `.` and the names do not match. The list is relative to the transfer root you gave. A glob in the list is not expanded by the shell, because the shell never saw those lines. rsync can filter them. Quote nothing inside a NUL list. `--files-from` does not imply `--delete`. Destinations you did not mention stay.

## Remove the source after a successful copy
Also asked as: rsync --remove-source-files; move with rsync; delete source after transfer; rsync then rm; not quite mv
`--remove-source-files` deletes each source file after that file was transferred successfully. It does not remove source directories. It is not `mv`. A failed file is left. A second run can finish and remove. Across filesystems this is the honest move. On the same filesystem `mv` is still the rename.

```sh
rsync -a --remove-source-files -- dir/ dir
```

People expect the empty directories to disappear. They remain. A later `find` can remove empty directories if that is what you want. Do not combine this with a careless `--delete` on a destination you have not looked at. The source deletion happens per file, after the receiver has the file. An interrupted run leaves a mix. Some sources are gone, some are not. The destination is the side to trust for the files that were removed at the source.

## What a failed rsync means
Also asked as: rsync exit code; rsync exit status; rsync 23; partial transfer; rsync 24; vanished files
`rsync` exits 0 only when the run finished cleanly. 1 is a syntax or usage error. 23 means some files were not transferred. 24 means some source files vanished during the run. Other non-zero values cover protocol and I/O failures. A log line with the reason is on standard error. The destination may contain a partial tree. Status 0 is the only "done."

```sh
rsync -a -- dir/ dir
```

People test for any output, or ignore the status, and treat a warning as success. A permission denied on one file is often 23, and the rest may have copied. Rerun after you can read the source, or exclude it on purpose. `set -e` aborts a script on 23. That is usually what you want. `--ignore-missing-args` changes how a missing files-from entry is treated. It does not turn 23 into 0 for a file you could not read.
