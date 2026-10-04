# tar

`tar` packs files and directories into one archive, and later lists or extracts that archive. It does not search file contents, it does not encrypt, and it does not delete the originals. On GNU tar a `.tar.gz` is not compressed just because of the name. Compression is a filter (`-z`, `-j`, `-J`, `--zstd`, or `-a`) wrapped around one tar stream. Gzip alone compresses one file and forgets the tree. You must choose one operation: `-c` to create, `-x` to extract, or `-t` to list. Without one of those, tar has nothing to do.

Basic form: tar -cf archive.tar file

`-f` takes the archive name as its argument. Every later path is a file to store or a member to extract, not the archive. Short flags cluster, and the dash is optional on GNU tar, so `tar czf archive.tar.gz dir` is the same idea as `tar -czf`. Quote globs with single quotes. The shell expands `*` before tar sees it. Put paths you will replace after `--`, so a leading dash stays a path. GNU tar on Linux is assumed here. macOS ships bsdtar: the same `-c`, `-x`, `-t`, `-f`, `-z`, `-j`, `-J`, `-v`, and `-C`, but it often compresses from the filename even when you never asked. BusyBox tar can create, list, and extract, and a given build often lacks `--wildcards`, `--delete`, `--exclude-vcs`, `-a`, and `--one-top-level`. Exit status is 0 when the run finished cleanly. It is 1 when `-d` finds a difference, or when a file changes while `-c`, `-r`, or `-u` is reading it. It is 2 when the archive will not open, a member you named is absent, a compressor rejects the stream, or a source path is missing. A bad command line can exit 64.

## Create an archive
Also asked as: how to create a tar file; tar -cf; tar -c; make an archive; pack a directory; tar create
`-c` writes a new archive. `-f` names it. Directories are stored recursively, under their own name, including files that start with a dot. With no `-v`, success prints nothing. The files you named stay on disk. This command does not gzip the result.

```sh
tar -cf archive.tar -- file
```

People treat a silent exit as "nothing happened." Check with `tar -tf archive.tar`. A missing source is fatal, exit 2, and the archive from that run should not be trusted. `tar -cf archive.tar` with no paths refuses to write an empty archive. The next word after `-f` is the archive name, so `tar -cf file dir` archives `dir` into an archive named `file`.

## Extract an archive
Also asked as: how to extract a tar file; tar -xf; tar -x; untar; unpack an archive; tar extract overwrite
`-x` writes the members into the current directory, creating directories the stored paths need. `-f` names the archive. With no `-v`, success prints nothing. Existing files are overwritten. Files that are not in the archive are left alone. This is not a mirror of the tree.

```sh
tar -xf archive.tar
```

People add `-z` from habit. On GNU tar, extract detects gzip, bzip2, and xz by itself, and the wrong letter makes the child compressor fail with exit 2. A tarball of loose files spills into the current directory. Stored paths are used as written, so the names you get are the names `tar -tf` prints, not a flat list of basenames.

## List what is in an archive
Also asked as: list tar contents; tar -tf; tar -t; see files without extracting; names inside a tarball; tar tf
`-t` prints every member name, one per line, and writes nothing to disk. `-f` names the archive. GNU tar detects a compressed stream here, so a `.tar.gz` does not need `-z`. A missing archive prints an error on stderr and exits 2.

```sh
tar -tf archive.tar
```

People expect sizes and permissions. This listing is names only. They also pass `-z` to "be safe" on a plain `.tar` or an xz archive. Forcing gzip on a stream that is not gzip fails even though plain `-tf` would have worked. The names are the exact strings you must pass back to extract one member.

## Show size, owner, and mode
Also asked as: tar -tvf; long listing; permissions inside a tar; size of members; tar list like ls; verbose table of contents
`-t` lists, `-v` adds a long line per member, and `-f` names the archive. The line looks like `ls -l`: mode, `owner/group`, size in bytes, modification time, then the member name. A symlink shows `->` and its target. Nothing is extracted.

```sh
tar -tvf archive.tar
```

People read the size column as the compressed size of the `.tar.gz`. It is the size of that member inside the tar stream, before the outer compression. The size of the archive file itself is what `ls -l` says about `archive.tar`, not what `-tvf` says about each line. Owner names come from the archive. They are not recomputed for your machine.

## Create a gzip archive
Also asked as: tar -czf; make a tar.gz; tar cvzf; tgz versus tar.gz; gzip a directory; macOS tar.gz without -z
`-c` creates, `-z` filters the whole archive through gzip, and `-f` names the file. `.tar.gz` and `.tgz` are the same layout. `-v` prints the member names it stores. On GNU tar, `-z` is what actually compresses. The suffix does not.

```sh
tar -czf archive.tar.gz -- dir
```

`tar -cf archive.tar.gz -- dir` succeeds and writes an uncompressed tar that is merely named like gzip. Tools that require a gzip header will reject it. macOS bsdtar often compresses from the `.gz` suffix anyway, so a recipe that works on a Mac can produce a different file on GNU tar. Gzip is one stream around the whole archive, not a separate compression of each file.

## Create a bzip2 or xz archive
Also asked as: tar -cjf; tar -cJf; tar.bz2; tar.xz; tar -j; tar -J; bzip2 versus xz
`-j` filters through bzip2. `-J` filters through xz. Both are create-time choices, used with `-c` and `-f` the same way `-z` is. The capital letter is the only difference between gzip's cousin flags, and it is easy to swap. Neither flag changes how paths are stored.

```sh
tar -cjf archive.tar.bz2 -- dir
```

```sh
tar -cJf archive.tar.xz -- dir
```

Use `-j` when you need `.tar.bz2`. Use `-J` when you need `.tar.xz`. They do not detect each other. Feeding `-j` a tree you meant to store as xz just makes a bzip2 file with the wrong name if you also picked the wrong suffix. On extract, GNU tar does not need either flag if you leave the compressor unspecified.

## Create a zstd archive
Also asked as: tar --zstd; tar.zst; zstd a directory; there is no -zstd; GNU tar zstd
`--zstd` filters the archive through zstd. There is no short letter. `-z` is gzip. Clustered `-zstd` is not this option. GNU tar has `--zstd`. BusyBox has it only when that build was compiled with zstd. Pair it with `-c` and `-f`.

```sh
tar --zstd -cf archive.tar.zst -- dir
```

People write `tar -czstd` or `tar -c -zstd` and get a gzip attempt or an illegal option. The long option has to stand on its own. As with gzip, the `.zst` suffix does not turn compression on by itself. A wrong compressor flag on extract fails. Plain `-xf` lets GNU tar detect the stream.

## Choose compression from the suffix
Also asked as: tar -a; tar --auto-compress; tar.gz that is not gzip; suffix picks the compressor; GNU -a
`-a` makes GNU tar pick the compressor from the archive name when creating. `.tar.gz` and `.tgz` mean gzip, `.tar.bz2` means bzip2, `.tar.xz` and `.txz` mean xz, `.tar.zst` means zstd. A plain `.tar` stays uncompressed. `-a` is GNU. It does not replace `-c` or `-f`.

```sh
tar -caf archive.tar.gz -- dir
```

People think every tar looks at the suffix. GNU tar does not, unless you pass `-a` or an explicit compressor. macOS bsdtar often does it without `-a`, which hides the bug until the archive is moved to Linux. `-a` on extract just means "detect," which GNU `-xf` already does. A suffix that tar does not know, compressed with `-a`, stays an uncompressed tar under a hopeful name.

## Extract a compressed archive
Also asked as: extract tar.gz without -z; tar -xf archive.tar.gz; tar -xzf failed; auto detect compression; wrong compressor flag; untar tar.xz
GNU tar reads the compression from the bytes when you extract or list. `-x` extracts, `-f` names the archive, and no `-z`, `-j`, or `-J` is required. The same command works for gzip, bzip2, and xz. Success with no `-v` prints nothing and overwrites files already on disk.

```sh
tar -xf archive.tar.gz
```

```sh
tar -xzf archive.tar.gz
```

Use the first form. Use the second only when you know the stream is gzip and you want to force that. `-z` on an xz or bzip2 archive, or on a plain tar, makes gzip reject the input and tar exit 2, even though omitting `-z` would have worked. BusyBox builds are the case that sometimes still wants the letter. If plain `-xf` cannot open it, then pass the letter that matches the file you actually have.

## Extract into a chosen directory
Also asked as: tar -C; extract into a directory; tar -xf -C; unpack into a folder; change to directory before extract
`-C` changes into a directory before tar writes members. `-x` extracts and `-f` names the archive. Paths from the archive are created under that directory. `-C` does not create its own argument. Make the directory first. It does not strip a prefix. The stored relative paths are still appended underneath.

```sh
tar -xf archive.tar -C dir
```

The gotcha is argument order. `-f` consumes the next word, so `tar -xf -C dir archive.tar` tries to open an archive named `-C`. Put the archive name immediately after `-f`, and put `-C dir` where it cannot be eaten. `-C` applies to the files that follow it. It is not a global rename of members. A member stored as `dir/file` still comes out as `dir/file` under the directory you named.

## Extract one member
Also asked as: extract one file from a tar; tar -xf a single path; Not found in archive; extract one directory; member name must match
Name the member after the archive. `-x` extracts only that member and the parents it needs, and `-f` names the archive. The string has to match a line from `tar -tf`, including directory prefixes. A basename is not enough when the archive stored `dir/file`.

```sh
tar -xf archive.tar -- 'path'
```

People copy a path from their tree instead of from the archive, and tar exits 2 with "Not found in archive." Quotes matter once the name contains spaces or globs. Without `--wildcards`, a `*` is a literal character in a member name, not a pattern. Extracting a directory member extracts that prefix, not every file whose basename happens to match.

## Extract members that match a glob
Also asked as: tar --wildcards; extract all files matching a pattern; Pattern matching characters used in file names; tar --no-wildcards; glob inside an archive
GNU tar does not glob member names unless you pass `--wildcards`. The pattern is then a shell-style glob matched against the stored names. Quote it, or the shell expands it against the current directory first and tar never sees the star. `--` keeps the pattern from being read as an option.

```sh
tar --wildcards -xf archive.tar -- '*.txt'
```

Without `--wildcards`, tar warns "Pattern matching characters used in file names" and looks for a member whose name is literally `*.txt`. That member is not there, so it exits 2. This flag is GNU. The glob matches the whole stored path, so `*.txt` can match `dir/notes.txt`. It is not a search of file contents. `--no-wildcards` forces a literal name when a real filename contains a star or a bracket.

## Choose the prefix stored in the archive
Also asked as: archive without the parent directory; tar -C to store contents; paths inside the tar; leading ./ in member names; tar the folder or its contents
`tar -cf archive.tar -- dir` stores every path with the `dir/` prefix, and a later extract recreates `dir`. To store the contents as seen from inside that directory, `-C dir` changes there first and `.` is the tree you pack. `.` is a real member prefix. Listing then shows `./file`, not `file`.

```sh
tar -cf archive.tar -- dir
```

```sh
tar -C dir -cf archive.tar .
```

Use the first form when the directory name should be part of the archive. Use the second when you want what is inside `dir` and you can tolerate `./`. The gotcha is combining them wrong: `-f` eats `-C` if you write `tar -cf -C dir archive.tar .`. A trailing slash on `dir/` does not drop the prefix on GNU tar. Shell `*` inside the directory also drops dotfiles, which naming `dir` or `.` does not.

## Leave out matching paths
Also asked as: tar --exclude; exclude a directory; exclude a glob; leave out node_modules; quote exclude patterns; tar -X; tar --exclude-from
`--exclude` takes a shell glob, not a regular expression, and skips matching paths while creating, listing, or extracting. A pattern with no slash matches that name in any directory, so `*.o` drops object files everywhere and also drops a directory whose own name matches. Excluding a directory drops everything under it. Quote the pattern.

```sh
tar --exclude='pattern' -cf archive.tar -- dir
```

```sh
tar -X exclude-file -cf archive.tar -- dir
```

Use `--exclude` for one or two globs. Use `-X` (the same as `--exclude-from`) when the globs are listed one per line in a file. Those lines are not expanded by the shell either. The gotcha is an unquoted `*`. The shell replaces it with names in the current directory, tar excludes those specific names, and the rest of the tree is stored. Exclude patterns do not apply only to the first path component unless you anchor them on purpose.

## Leave version control out
Also asked as: tar --exclude-vcs; exclude .git; do not pack git; tar --exclude-vcs-ignores; skip .svn and CVS
`--exclude-vcs` drops the metadata directories of the usual version-control systems, including `.git`, `.svn`, and `CVS`, and then archives the rest. It is a GNU option. It does not read `.gitignore`. It does not skip files you ignored. It only skips those VCS directories.

```sh
tar --exclude-vcs -cf archive.tar -- dir
```

```sh
tar --exclude-vcs-ignores -cf archive.tar -- dir
```

Use `--exclude-vcs` when the goal is "no repository metadata." Use `--exclude-vcs-ignores` when you also want the ignore rules from the tree. Both are GNU, and neither is implied by `--exclude='.git'` written against the wrong relative path. A hand-written `--exclude='.git'` does work for a directory of that name at any level, because exclude patterns are unanchored, but it will not catch every VCS name `--exclude-vcs` knows.

## Do not overwrite existing files
Also asked as: tar -k; tar --keep-old-files; tar --skip-old-files; tar --keep-newer-files; extract without clobbering; File exists
Extract overwrites by default and does not ask. `-k` (`--keep-old-files`) refuses to replace a file that is already there, prints "File exists" on stderr, leaves the old bytes, and exits 2. `--skip-old-files` is the quiet version: it skips existing paths and still exits 0. `--keep-newer-files` replaces a file only when the archive copy is newer, prints a note when it skips, and exits 0.

```sh
tar -k -xf archive.tar
```

```sh
tar --skip-old-files -xf archive.tar
```

Use `-k` when an existing file should fail the run. Use `--skip-old-files` when you want whatever is already on disk to win, with a successful status. `--keep-newer-files` is the GNU choice when time should decide. People pass `-o` for "overwrite." On extract, `-o` means `--no-same-owner`. Overwrite was already the default, and `-o` does not turn it off.

## Strip leading directories while extracting
Also asked as: tar --strip-components; strip-components=1; remove the top directory; drop a path prefix; short names were skipped
`--strip-components` takes a count of leading path parts to drop from each member as it is extracted. `--strip-components=1` turns `dir/sub/file` into `sub/file`. It does not look up a directory by name. It always removes that many slash-separated pieces. Pair it with `-xf` and the archive name.

```sh
tar -xf archive.tar --strip-components=1
```

A member that does not have that many components is skipped, with no error and exit 0. Stripping 1 from an archive that also contains a top-level `README` drops `README` and keeps only what lived under the first directory. People use this to mean "extract the one top folder's contents" and then lose every file stored beside that folder. The count has to match the paths `tar -tf` shows, including a `./` prefix if that is what was stored.

## Store the file a symlink points at
Also asked as: tar -h; tar --dereference; follow symlinks; archive the link target; macOS tar -L; GNU -L is tape length
By default tar stores a symlink as a symlink. The archive holds the link text, not the target's bytes. `-h` (`--dereference`) reads the target and stores a regular file under the link's name. Directory symlinks are walked, so the walk can leave the tree you named. GNU `-h` is this switch. It is not help.

```sh
tar -chf archive.tar -- dir
```

On macOS, following links is `-L`, not `-h`. On GNU tar, `-L` is `--tape-length` and takes a number. Copying a macOS `-L` into GNU tar does not follow links. It sets a tape size and then misreads the next argument. A broken symlink fails under `-h` because there is no target to read. Without `-h` it is stored as a link and extract recreates the link, not the file it used to point at.

## Send an archive through a pipe
Also asked as: tar -f -; tar to stdout; tar from stdin; pipe tar to tar; tar -c without -f waits; tar pipe gzip
`-f -` means stdout when you create and stdin when you extract or list. The archive bytes are the stream. Other commands can compress or move them. When the archive is the pipe, `-v` prints names on stderr so it does not corrupt the bytes. GNU tar also treats a missing `-f` as `-f -`.

```sh
tar -cf - -- dir | tar -xf - -C dir
```

`tar -x` or `tar -t` with no `-f` and no pipe sits waiting for an archive on the terminal. It is not listing the current directory. People add `-v` on a pipe and then, on a file archive, wonder why names and progress mixed. Names go to stdout only when the archive itself is a real file. Prefer an explicit `-f -` so the next reader can see where the bytes are supposed to come from.

## Add files to an archive that already exists
Also asked as: tar -r; tar --append; tar -u; tar --update; Cannot update compressed archives; add a file to a tar; member stored twice
`-r` appends the named paths to the end of an existing uncompressed archive. `-u` appends a path only when the disk copy is newer than the member already stored. Neither rewrites earlier members. `tar -tf` shows the name twice, and a later extract uses the last copy. Both refuse a compressed archive with "Cannot update compressed archives" and exit 2.

```sh
tar -rf archive.tar -- file
```

```sh
tar -uf archive.tar -- file
```

Use `-r` to force another copy in. Use `-u` to add only newer files. For a `.tar.gz`, `.tar.bz2`, or `.tar.xz`, recreate the archive with `-c` and the compressor. There is no in-place append once the stream is compressed. BusyBox hits the same wall. `-u` does not delete the stale member it replaced in spirit. The archive only grows.

## Delete a member from an archive
Also asked as: tar --delete; remove a file from a tar; delete a member; edit a tarball; cannot delete from tar.gz
`--delete` removes a member from an uncompressed archive in place. `-f` names the archive, and the member name must match a stored path. GNU tar can do this on a seekable file. It will not do it on a compressed archive. It reports that it cannot update compressed archives and exits 2. This option is GNU. It is not a tape operation.

```sh
tar --delete -f archive.tar -- 'path'
```

People run it against `archive.tar.gz` and assume the failure means the path was wrong. The path can be right and the format still rejects the edit. Recreate a compressed archive without that path, using `--exclude` or by extracting and packing again. `--delete` does not take a glob unless you also ask for wildcards the way extract does. A wrong member name leaves the archive unchanged and fails.

## Compare an archive with the files on disk
Also asked as: tar -d; tar --compare; tar --diff; archive differs from disk; tar exit status 1; check members against the tree
`-d` reads the archive and compares each member with the path of the same name in the current directory. Matching files print nothing and the status is 0. A difference prints a line such as `file: Size differs` and the status is 1. That 1 does not mean the archive file is corrupt. It means disk and archive disagree. `-f` names the archive.

```sh
tar -df archive.tar
```

The comparison uses stored names, so you run it from the directory where `dir/file` still means what it meant when you created the archive. A missing disk file is a difference. `-d` does not extract, and it does not checksum the gzip wrapper. To see whether a compressed stream can be read at all, `tar -tf archive.tar.gz` has to walk it without error. Exit 2 on that run is a broken archive. Exit 1 on `-d` is a content mismatch.

## Handle "file changed as we read it"
Also asked as: file changed as we read it; tar exit code 1 while creating; tar --warning=no-file-changed; file changed during backup; live file was archived
While `-c` is reading a file, tar notices if the file's size or modification time changes. It finishes the archive, prints `file changed as we read it` on stderr, and exits 1. The member can be a torn snapshot of a file that was being written, not a clean copy of either version. Log files and databases do this. Exit 1 here is not exit 2. The archive exists, and that member is the part you should not trust.

```sh
tar --warning=no-file-changed -cf archive.tar -- dir
```

`--warning=no-file-changed` hides the message on GNU tar. On this tar it does not turn the status back into 0. Scripts that treat any non-zero status as failure still fail. People delete the warning and ship the archive. Copy the data to a snapshot or stop the writer first if that member has to be consistent. The same exit 1 applies to `-r` and `-u` when a file changes during the read.

## Archive hidden files
Also asked as: tar dotfiles; include hidden files; star skips names starting with a dot; tar -cf . ; file is the archive not dumped
`*` is expanded by the shell, and it does not include names that start with a dot. `tar -cf archive.tar *` packs the visible names in the current directory and silently omits `.hidden`. Naming a directory does include dotfiles underneath it, because tar itself walks the directory and does not use the shell glob.

```sh
tar -cf archive.tar -- dir
```

`tar -cf archive.tar .` also includes dotfiles, and it stores a `./` prefix. If the archive is being created inside that same directory, tar prints `file is the archive; not dumped` and skips the archive file so it does not pack itself. People add `--exclude='.*'` thinking it means "hidden files only at the top" and instead drop every dotfile at every level. Quote any glob you pass. Prefer naming `dir` when the dotfiles that matter live inside it.

## Keep absolute paths out of the archive
Also asked as: Removing leading '/' from member names; tar -P; tar --absolute-names; strip leading slash; absolute paths in a tarball
If you name `/etc/hostname` as a source, GNU tar stores `etc/hostname` and prints `Removing leading '/' from member names` on stderr. Extract then writes a relative path, not `/etc/hostname`. That is the safe default. `-P` (`--absolute-names`) keeps the leading slash in the member name.

```sh
tar -cf archive.tar -- file
```

```sh
tar -P -cf archive.tar -- /absolute/path
```

Use the first form. Use `-P` only when extract must recreate those absolute paths on purpose. Extracting an archive built with `-P` writes to those absolute locations, which as root overwrites whatever is already there. People see the "Removing leading" line and think the run failed. It is a notice. The status is still 0 if nothing else went wrong. The stored name is the relative one `tar -tf` prints.

## Preserve permissions and owners
Also asked as: tar -p; preserve permissions; tar --same-owner; extract as yourself; umask on extract; tar -o is not overwrite
Tar records the mode, owner, and group it sees at create time. On extract, root restores owner and mode by default. A normal user owns the new files and, unless you pass `-p`, the mode is masked by umask. `-p` (`--preserve-permissions`, also `--same-permissions`) asks for the mode bits from the archive. `--same-owner` asks for the stored owner, which a normal user cannot grant, so the files still end up owned by you.

```sh
tar -xp -f archive.tar
```

People use `-o` to mean overwrite or "preserve everything." On extract, `-o` means `--no-same-owner`. On create, `-o` means the ancient V7 archive format. Neither is `-p`. Permissions on extract also do not change the fact that an existing file's bytes are replaced by default. Mode and owner are a separate question from whether the file is overwritten.

## Put the archive name right after -f
Also asked as: tar -f argument order; tar cfz versus czf; -f ate the flags; tar -cfz wrong name; clustered short options; tar czf syntax
In a short cluster, `-f` consumes the rest of the cluster as its argument if anything follows it, otherwise the next word. `tar -czf archive.tar.gz -- dir` is right because `f` is last and `archive.tar.gz` is the next word. `tar -cfz archive.tar.gz -- dir` creates an archive named `z` and then tries to store a file named `archive.tar.gz`. The same rule applies to the old spelling without a dash.

```sh
tar -czf archive.tar.gz -- dir
```

```sh
tar -c -z -f archive.tar.gz -- dir
```

Use the cluster when `f` is the last letter. Use the split form whenever you are no longer sure which letter takes an argument. `-C` has the same appetite. `tar -xf -C dir archive.tar` opens an archive called `-C`. Flags that do not take arguments (`c`, `x`, `t`, `v`, `z`, `j`, `J`, `p`, `h`, `k`) can sit in any order inside the cluster. `f` cannot.

## Extract into a fresh directory
Also asked as: tar --one-top-level; tarball of loose files; do not spill into the current directory; GNU one-top-level; extract under a new folder
`--one-top-level` is GNU. On extract it creates one new directory and puts every member under it, so a tarball of loose files does not fill the current directory. With no name, the directory is taken from the archive name (`a.tar` becomes `a`). `--one-top-level=dir` uses `dir` instead. Combine it with `-xf`.

```sh
tar -xf archive.tar --one-top-level=dir
```

It always adds that directory, even when the archive already has a single top-level folder. You then get `dir/src/file` rather than `src/file`. It does not strip components. It does not refuse absolute members by itself. Those are still stripped unless the archive was built with `-P`. BusyBox and traditional tar do not have this flag. The portable version of the same idea is to `mkdir dir` and extract with `-C dir`.

