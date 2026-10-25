# tar

`tar` stores files in one archive and pulls them back out. The name is tape archive. In the olden days the archive was a magnetic tape. Today it is almost always a file on disk. `tar` does not compress by itself. A `.tar.gz` is a tar archive that gzip compressed, which `-z` asks for. With no mode flag it prints help and exits non-zero. GNU tar 1.34 on Linux is assumed here. macOS ships bsdtar. It accepts `-c`, `-x`, `-t`, and `-f`, and it differs on `--strip-components`, `--one-top-level`, and the exit status of a warning. BusyBox tar has create, list, and extract, and a smaller set of compression flags. Exit status is 0 when the operation finished cleanly. It is 1 when some files changed while being read, or differed. It is 2 on a fatal error, including a missing archive and a bad option. A warning is not always 0.

Basic form: tar -cf archive.tar file

The mode comes first. `-c` creates, `-x` extracts, `-t` lists. `-f` names the archive, and it consumes the next argument. That is the order bug. `-cf archive.tar` is create. `-fc archive.tar` treats `c` as the archive name. Operands after the archive are member paths. `--` before a path stops a leading dash from looking like a flag. Quote a name with spaces. A glob is expanded by the shell before `tar` sees it.

## Create an archive
Also asked as: tar -cf; make a tar; create a tarball; tar a directory; pack files
`-c` creates a new archive. `-f` names it. Everything after the archive name is stored. A directory is stored with its contents. The archive is truncated if it already exists. Compression is a separate flag. Without `-z` or `-J` or `-a` you get an uncompressed `.tar`. The source files are not removed.

```sh
tar -cf archive.tar -- file
```

People write `-cfz` and tar takes `z` as the archive name, then fails. The `f` must be followed by the filename. `-czf archive.tar` is the compressed form. People also expect the archive to appear inside the directory they named. It appears in the current directory, under the name you gave `-f`. A directory operand stores the directory name as a prefix. `tar -cf archive.tar -C dir .` stores the contents without that prefix.

## Create a gzipped archive
Also asked as: tar -czf; tar.gz; tarball gzip; compress a tar; tar cvzf
`-z` filters the archive through gzip. The usual file name is `archive.tar.gz`. `-j` is bzip2. `-J` is xz. `-a` picks the compressor from the archive name, so `archive.tar.gz` gets gzip. The flag still has to sit with the mode, and `-f` still consumes the next word. Listing and extracting a compressed archive need the same idea, or `-a`.

```sh
tar -czf archive.tar.gz -- file
```

People create `archive.tar` and then gzip it in a second step, which works and is the same bytes. One command is enough. People pass `-z` when extracting a `.tar.gz` and also pass the uncompressed name. The `-f` argument is the real filename. A corrupt gzip stream fails at extract with status 2. Create does not check that you can read every file until it reads it. A permission denied on one file is a warning and a non-zero status. The archive can still have been written.

## List what is in an archive
Also asked as: tar -tf; list tarball; tar tvf; what files are in this tar; tar table of contents
`-t` lists member names. `-f` names the archive. `-v` adds the mode, owner, size, and time, in a form like `ls -l`. Compression is detected from the file on GNU tar, so `-z` is optional for a `.tar.gz`. The list is the names stored in the archive, not the names they would have after `--strip-components`.

```sh
tar -tf archive.tar
```

People `tar -tf` a file that is not an archive and get a short error and status 2. People also read the verbose owner as the owner the extract will use. Extract maps the owner unless you are root or you pass a preserve flag. The list does not say whether the member is a symlink. `-v` does. A member named `file` and a member named `./file` are different names. Extract writes the name that is stored.

## Extract an archive
Also asked as: tar -xf; untar; extract tarball; unpack tar.gz; tar xvf
`-x` extracts. `-f` names the archive. Member names are written relative to the current directory, unless you pass `-C`. A compressed archive is detected. Existing files are overwritten. Existing directories are not replaced. They are entered. `-v` prints each name as it is written. `-k` does not overwrite an existing file.

```sh
tar -xf archive.tar
```

People extract in the wrong directory and scatter files. `-C dir` extracts into `dir`, which must already exist. People expect `-x` to delete files that are not in the archive. It only writes members. A member with an absolute path is stripped to a relative path on GNU tar unless you pass `-P`. That strip is the safe default. `-P` is how you ask for the dangerous form.

## Extract into a directory
Also asked as: tar -C; extract to a folder; tar change directory; unpack into dir; -C and -f order
`-C` changes the directory `tar` uses, and it applies to what follows it. `tar -xf archive.tar -C dir` extracts into `dir`. `tar -C dir -xf archive.tar` does the same. The directory must exist. `-C` does not create it. You can give `-C` more than once. Each one applies to the operands after it.

```sh
tar -xf archive.tar -C dir
```

People put `-C` after a list of member names and it does not apply to those names. Place it before the operands it should affect, or before `-x` if the whole extract should land there. A relative `-C` is relative to the shell's current directory, not to the archive. Extracting as root into `/` is how a tarball with a member named `etc/passwd` becomes a bad day. Look at `-t` first. Extract into an empty directory you made.

## Drop leading directories on extract
Also asked as: tar --strip-components; strip components; remove the top folder; extract without the prefix; --strip-components=1
`--strip-components=1` removes one leading path element from each member name before writing it. A member `src/file` becomes `file`. A member `src/sub/b` becomes `sub/b`. It does not rename the archive's top directory as a special case. It strips that many components from every name. This is GNU. macOS bsdtar has the same long option on current systems.

```sh
tar -xf archive.tar --strip-components=1
```

People strip one component from an archive that has no common top directory, and `file` and `other/file` both lose their first element, which may collide. List first. A strip count larger than a member's depth skips that member. It is not extracted. People also expect strip to create the destination directory. It does not. Combine it with `-C` and a directory you created.

## Extract one file
Also asked as: tar extract one file; tar -xf archive file; pull a single member; extract a path from a tarball
Name the member after the archive. The name is the name `-t` printed, not a glob, unless you pass `--wildcards`. A directory member extracts that directory and, on a normal extract, the files under it that are also in the archive. Compression does not change the form. The member is written to the current directory, or to `-C`.

```sh
tar -xf archive.tar -- file
```

People pass a path that looks right and get nothing, because the stored name is `./file` or `src/file`. Copy the name from `-t`. A wildcard without `--wildcards` is a literal member name and matches nothing. With `--wildcards`, `*.txt` is a pattern, and the shell will also expand it unless you quote it. Quote the pattern. `--no-anchored` lets a pattern match in any directory. The default anchor is the start of the member name.

## See a verbose listing
Also asked as: tar -tvf; verbose tar; tar tvf; long listing; permissions in the archive
`-v` on a list prints one line per member, like `ls -l`, with mode, owner, size, time, and name. `-vv` on create or extract prints the name and the attribute details. Verbose goes to stdout on a list. On create and extract it goes to stdout as well, which can mix with a pipe. `-f` still names the archive.

```sh
tar -tvf archive.tar
```

People pipe `-tvf` into `grep` and also have a locale date format that splits into extra fields. The name is the last field only when the name has no spaces. A name with spaces breaks a column cut. Use `-t` without `-v` if you only need names. The owner in the listing is the stored owner. It is not a promise about the extracted file unless you preserve ownership.

## Append files to an archive
Also asked as: tar -r; tar --append; add a file to a tar; tar -u update; append to tarball
`-r` appends named files to an uncompressed archive. `-u` appends only files that are newer than the copy already in the archive. Neither works on a compressed archive. Gzip is not a format you can append to. Neither deletes a member. An old member of the same name stays, and the new one is added after it. Extract uses the last one.

```sh
tar -rf archive.tar -- file
```

People `-r` onto a `.tar.gz` and get an error. Create a new archive if it is compressed. People also expect `-u` to replace the old member. The archive can contain two members with the same name. `-t` shows both. Extract writes the last. `--delete` removes a member, and GNU tar refuses it on a tape, which is the old constraint. On a file it rewrites the archive.

## Exclude names
Also asked as: tar --exclude; skip a directory; exclude node_modules; tar exclude pattern; do not archive a path
`--exclude` takes a pattern. A matching file or directory is not stored. You can repeat the flag. The pattern is a tar pattern, not a full regular expression. `*.o` matches in any directory. A leading `./` in the stored name can surprise a pattern that does not account for it. `--exclude-from` reads patterns from a file.

```sh
tar -cf archive.tar --exclude='pattern' -- file
```

People exclude `dir` and still see `./dir` stored, because the operand was `.` and the stored prefix differs. Test with `-t`. An exclude does not remove a member already in the archive. It only affects this create. Quote the pattern or the shell expands it. `--exclude` on extract skips matching members. The same flag, the other direction.

## Change directory before adding files
Also asked as: tar -C create; archive the contents not the folder; tar from another directory; -C and create
`-C dir` changes directory before the following operands are read. `tar -cf archive.tar -C dir .` stores what is inside `dir`, under names starting at `.`, not under `dir/`. A later `-C` switches again. This is how you build an archive whose members are not prefixed by the source folder.

```sh
tar -cf archive.tar -C dir .
```

People `tar -cf archive.tar dir` and every member starts with `dir/`. Extract then creates `dir`. The `-C` form is the one that does not. `.` is an operand. It means the directory you just entered. It is not optional. Forgetting it stores nothing from that directory. `-C` on create does not imply `-C` on extract. The reader still extracts into the current directory.

## Preserve permissions and owners
Also asked as: tar -p; tar --preserve-permissions; same owner; extract as root; --no-same-owner
`-p` on extract preserves permissions. GNU tar does this by default when you are root. `--same-owner` restores the stored user and group, which usually requires root. `--no-same-owner` makes the extracted files yours. Times are restored by default. `--touch` gives them the current time instead.

```sh
tar -xpf archive.tar
```

People extract as a normal user and expect to become the stored owner. The kernel will not allow it. The file is owned by you. People extract as root and spread a tarball's numeric ids onto a machine where those ids are different people. `--no-same-owner` is the guard. Permissions still come across with `-p`. A setuid bit in a tarball is a reason to list before you extract as root.

## Follow symlinks, or store the links
Also asked as: tar -h; tar --dereference; symlink in a tarball; store the link; tar a symlink
By default `tar` stores a symlink as a symlink. The target is not copied in. `-h` stores the target's content instead, as a regular file. A dangling symlink fails under `-h` and is stored as a link without it. Hard links are stored as links when the other name is also in the archive.

```sh
tar -cf archive.tar -- file
```

```sh
tar -chf archive.tar -- file
```

Use the first when the link is the thing you are archiving. Use the second when you want the bytes. People extract an archive of symlinks onto another machine and the links dangle, because the targets were never members. `-h` at create time is how you snap the targets. `-h` at extract time is a different request and is not the usual flag. Check `-tvf` for the `l` type before you assume the bytes are in the archive.

## Compress with xz or bzip2, or pick from the name
Also asked as: tar -J; tar -j; tar -a; xz tarball; auto compress; tar.bz2
`-J` is xz. `-j` is bzip2. `-z` is gzip. `-a` looks at the archive name and chooses. `archive.tar.xz` gets xz. `archive.tar.bz2` gets bzip2. On extract, GNU tar detects the compression regardless of the flag. You can `tar -xf archive.tar.xz` with no `-J`.

```sh
tar -cJf archive.tar.xz -- file
```

People rename `archive.tar.gz` to `archive.tar.xz` and expect extract to cope. The bytes are still gzip. The name lied. Detection uses the bytes. Create uses the flag or, with `-a`, the name. A missing `xz` binary fails the create. The archive is not left half-written in a useful way. Check `tar -tf` after a long create if the job might have been killed.

## Read the list of files from a file
Also asked as: tar -T; files-from; tar a list; null separated tar; -T -
`-T` reads member names from a file, one per line. `-T -` reads standard input. `--null` reads NUL-separated names, which is the safe pair with `find -print0`. The names are paths to store, relative to the current directory or to a `-C`. This is how you archive a generated list without a shell glob.

```sh
tar -cf archive.tar -T file
```

People put archive member names in the list and also expect exclude logic to apply. Excludes still apply. A line with a leading `-` is a path if you used `-T`, but a path that starts with a dash can still be read as a flag in other positions. `--add-file` is the single-file form of that escape. A missing path in the list is a warning and a non-zero status. The rest are still archived.

## Extract and keep existing files
Also asked as: tar -k; tar --keep-old-files; do not overwrite; extract without clobber; keep existing
`-k` does not replace a file that already exists. The member is skipped. Directories are not a conflict in the same way. A file in the way of a directory member is still an error. `-k` is the opposite of the default, which overwrites. There is no prompt.

```sh
tar -xkf archive.tar
```

People pass `-k` and think a same-named file was updated. It was left as it was. The status can still be non-zero if any file was skipped this way, on GNU tar. A directory you expected to refresh keeps its extra files. `tar` does not delete those. It only decides whether to write members. `--skip-old-files` is the GNU flag that skips without the warning noise of `-k`.

## Make the archive's top directory
Also asked as: tar --one-top-level; extract into a new folder; wrap the extract; avoid scattering files; GNU one-top-level
`--one-top-level` extracts into a single new directory. The directory name comes from the archive name, with the compression suffix removed, unless you pass `--one-top-level=dir`. This is the guard against a tarball that has no common top directory and would spill into the current directory. It is GNU.

```sh
tar -xf archive.tar --one-top-level
```

People extract a tarball from a stranger and files land next to their own. `--one-top-level` puts them in a new folder. It does not help if you also pass a `-C` you did not mean. The new directory is created beside the current directory's files, not inside the archive. An archive that already has one top directory ends up with two levels. That is the cost of the guard. List first if you care.

## Compare the archive with the files
Also asked as: tar -d; tar --diff; tar --compare; verify an archive; does the tar match
`-d` compares the archive with the files on disk. It reports members that differ in size, mode, or time, and members that are missing on disk. It does not extract. It does not modify the archive. A compressed archive works. The names are the stored names, so you must be in the directory where those paths make sense, or use `-C`.

```sh
tar -df archive.tar
```

People use `-d` as a checksum of the archive file. It is not. It compares members with the filesystem. A matching compare prints nothing and exits 0. A difference prints a line and exits non-zero. Content that differs with the same size and time can slip past a metadata compare. For bytes, extract to a scratch directory and diff, or use a format that stores checksums you can recompute.

## What a failed tar means
Also asked as: tar exit code; tar exit status; tar failed; unexpected EOF; file changed as we read it
Status 0 is a clean run. Status 1 is a warning. A file changed while `tar` was reading it, or a compare found a difference. Status 2 is fatal. A missing archive, a bad flag, a corrupt compressed stream, and an unreadable file you did not ignore are fatal. GNU tar prints the reason on stderr. `-v` does not change the status.

```sh
tar -xf archive.tar
```

People treat status 1 as success because files appeared. A file that changed during create may be torn. `--warning=no-file-changed` turns that particular warning off, and it is GNU. A truncated download fails at extract with an unexpected EOF and status 2. The partial tree it wrote is not a successful extract. Check the status before you delete the archive. `set -e` aborts a script on status 1 unless you handled it. That is usually what you want.
