# ls

`ls` prints the names of files in a directory, or the names you pass it. It does not read file contents, it does not search inside files, and it does not enter subdirectories unless you pass `-R`. With no operand it lists the current directory. A directory operand is opened and its contents are listed. A file operand is listed as itself. Names that start with `.` are hidden unless you ask for them. On a terminal, GNU ls lays names out in columns and quotes odd characters. Into a pipe or a file, it prints one raw name per line, in locale alphabetical order, not in version order and not newest first.

Basic form: ls dir

Operands are paths, not a pattern language. The shell expands `*` before `ls` runs, so an unquoted star never reaches `ls`, and it does not match a leading dot. The only patterns `ls` itself matches are the ones you pass to `-I` or `--hide`, and those must be quoted or the shell eats them. Put a path you will replace after `--`, so a name that starts with a dash is still a path. GNU ls from coreutils on Linux is assumed here. macOS ships BSD ls. It shares `-l`, `-a`, `-A`, `-h`, `-R`, `-t`, and `-S`, but color is `-G`, a long timestamp is `-T`, and it has no `--group-directories-first`, no `-v`, no `-I`, and no `--zero`. On GNU ls those letters mean something else: `-G` drops the group column, and `-T` sets the tab width. BusyBox ls is smaller again. Long listings, hidden names, time and size sorts, and human sizes are usually there. The GNU long options are not. Exit status is 0 when every named path was listed. An empty directory is success. It is 2 when a command-line path is missing, cannot be opened, or the flags are illegal. It is 1 for a lesser walk problem, such as a subdirectory `-R` is not allowed to enter.

## List the names in a directory
Also asked as: list files in a directory; what does ls print; ls a folder; ls of several paths; why did ls print a header; locale sort order
With a directory operand, `ls` prints the names inside it, not the directory's own name. With a file operand, it prints that file. With no operand, it lists `.`. Several directories produce a blank line and a `dir:` header before each group. One directory does not get a header. Files mixed with directories are printed first, then each directory is opened under its own header.

```sh
ls -- dir
```

People expect a full path. `ls` prints the name inside the directory it opened, so `dir/file` is listed as `file` when you asked for `dir`. The order is alphabetical for the current locale, not byte order and not numeric order. `_`, case, and non-ASCII names move when `LC_COLLATE` changes. `LC_ALL=C ls -- dir` is the plain byte sort. A glob such as `*` is expanded by the shell first, and that expansion skips names that start with `.`.

## Print one name per line
Also asked as: ls -1; one file per line; ls in a script; ls columns; ls -C; output is wrapped
`-1` forces a single column, one name per line, and nothing else. No size, no header, no `total`. Use it when the next program reads names line by line. On a terminal, plain `ls` uses columns instead, so one row holds several names. Into a pipe, GNU ls already switches to one name per line, but `-1` does not depend on that.

```sh
ls -1 -- dir
```

`-C` forces columns even when stdout is not a terminal. Those lines then split on spaces if a name contains one. People omit `-1` in a script because "the pipe makes it one column here" and then an alias or a later `-C` wraps the names again. `-1` still hides dotfiles, and a name that itself contains a newline still breaks the line unless you change the quoting or use a NUL separator.

## Show a long listing
Also asked as: ls -l; long format; permissions owner size date; what is the total line; ls -s allocated size
`-l` prints one line per entry: file type and mode, link count, owner, group, size in bytes, modification time, then the name. A symlink adds ` -> ` and the target text. The byte count of a directory is the size of the directory itself, often 4096, not the sum of what is inside it. The byte count of a symlink is the length of its target string, not the size of the file it points at.

```sh
ls -l -- dir
```

The first line inside a directory listing is `total N`. That is allocated disk space in blocks, not the number of files. An empty directory still prints `total 0` and exits 0. `-s` is that allocated size per entry, in blocks, which is why a 3-byte file can show as 4. It is not the byte column. `-h` does not change `-l` unless you add it. Hidden names are still hidden.

## Read the permission string
Also asked as: what does drwxr-xr-x mean; ls -l permissions; file type character; rwx for a directory; setuid sticky bit; ls -Z
The first character of the `-l` mode is the type. `-` is a regular file, `d` a directory, `l` a symlink, `p` a pipe, `s` a socket, `c` a character device, and `b` a block device. The next nine characters are three `rwx` sets: owner, group, everyone else. `r` is read, `w` is write, and a missing bit is `-`. On a file, `x` means it may be executed. On a directory, `x` means it may be entered. Without directory `x`, the names inside cannot be looked up even when `r` is set.

```sh
ls -l -- file
```

An `s` in an execute position is setuid or setgid. A `t` in the last execute position is the sticky bit. The letter is uppercase `S` or `T` when that execute bit is otherwise off. A trailing `.` or `+` is not another permission. GNU ls prints `.` when the file has a security context and `+` when it has an ACL. Neither is decoded in the nine letters. `ls -Z` prints the SELinux context and is GNU. The mode string is not the octal number `chmod` takes. Those are two views of the same bits.

## Show hidden names
Also asked as: ls -a; ls -A; ls -la; show dotfiles; list files starting with a dot; almost all versus all
`-a` lists every name, including `.` and `..` and every dotfile. `-A` lists the dotfiles but not `.` and `..`. Both apply to directory listings. A name is hidden only because it starts with a dot. There is no separate hidden attribute that `ls` is honoring.

```sh
ls -a -- dir
```

```sh
ls -A -- dir
```

Use `-a` when you need the `.` and `..` entries. Use `-A` when you want dotfiles and those two would be noise. `ls -la` is `-l` plus `-a`, so you also get `.` and `..` in the long listing. A shell glob still does not match dotfiles. `ls -d -- dir/.*` is the shell's version of "the hidden names," and it includes `.` and `..` because `.*` matches them.

## List the directory, not what is inside it
Also asked as: ls -d; ls -ld; do not descend; list the folder itself; symlink to a directory shows contents
`-d` lists the operand instead of opening it. On a directory, you get the directory's name, not its children. Combined with `-l`, you get the directory's own mode, owner, and size. Without `-d`, a directory operand is always opened, and a symlink to a directory is opened too.

```sh
ls -ld -- dir
```

People run `ls -l dir` to see whether `dir` is writable and instead get a long listing of whatever is inside. The same surprise happens with a symlink to a directory: plain `ls` prints the target's contents, while `ls -ld` prints the symlink. `-d` does not mean "only directories." If you pass it files and directories together, it lists each operand and opens none of them.

## Print sizes people can read
Also asked as: ls -h; ls -lh; human readable file size; ls --si; size in megabytes; why does ls -h do nothing
`-h` prints sizes in units of 1024, such as `1.5M` or `2G`, instead of a raw byte count. It only affects a size column, so it has to be combined with `-l` or with `-s`. Alone, `ls -h dir` looks like plain `ls`. The numbers are rounded. `1.5M` is not an exact byte count and it is not 1.5 million bytes.

```sh
ls -lh -- file
```

```sh
ls -l --si -- file
```

Use `-h` for the usual KiB and MiB scale. Use `--si` when you want powers of 1000. `--si` is GNU. Both are display formats. The bytes stored in the inode do not change. A directory's human size is still the directory inode, not the total of its children. A symlink's human size is still the length of the link text unless you followed the link.

## Sort by modification time
Also asked as: ls -t; ls -lt; newest files first; sort by date; recently modified; ls -ltr oldest first
`-t` sorts by modification time, newest first. That is the time the contents changed, not the time the name was created and not the time permissions changed. `-l` adds the long columns and, with `-t`, shows that same timestamp. The default date in `-l` is abbreviated. Older files lose the clock time and gain a year, which is a display change, not a different clock.

```sh
ls -lt -- dir
```

```sh
ls -ltr -- dir
```

Use `-lt` for a long listing with the newest entry first. Use `-ltr` for the oldest first. `-r` only reverses whatever sort is in effect. It does not mean recursive. `-t` does not show hidden names, and it does not compare files inside subdirectories unless you also asked for `-R`. On a terminal without `-l` or `-1`, the newest-first names still come out in columns.

## Sort by size
Also asked as: ls -S; largest files first; ls -lS; sort by file size; biggest file in a directory
`-S` sorts by size, largest first. With `-l`, the size you see is the byte size used for that sort. Without a following `-r`, the largest name is first. This is the size `ls` would print for the entry, not a measurement of how much space the directory's children occupy.

```sh
ls -lS -- dir
```

A directory sorts by the directory's own byte size, usually a few kilobytes, so a folder full of large files does not rise to the top. A symlink sorts by the length of its target path, not by the target's size, unless you follow links with `-L`. `-h` changes only the printed unit. The order is still the underlying byte count. Hidden files are omitted, so the largest object in the directory can be absent from the list.

## Reverse the order
Also asked as: ls -r; reverse alphabetical; ls -tr; Z to A; reverse sort; ls -r is not recursive
`-r` reverses the sort that is already selected. With no other sort flag, names come out in reverse locale order. With `-t`, the oldest time is first. With `-S`, the smallest size is first. It does not recurse, it does not change columns versus one-per-line, and it does not reveal dotfiles.

```sh
ls -r -- dir
```

The flag people want for a tree is `-R`. `-r` is only the direction of the sort. Stacking both, as in `ls -lR -r`, reverses the entries inside each directory and still walks the tree. Reverse locale order is not the reverse of byte order unless you are already in the C locale. A terminal will still wrap the reversed names into columns unless you pass `-1` or `-l`.

## List subdirectories too
Also asked as: ls -R; recursive listing; list files in subfolders; ls recursive tree; do not follow symlink directories
`-R` walks subdirectories and prints each one under a `dir:` header. It lists the names it can open. A subdirectory it cannot enter is reported on stderr, that branch is skipped, and the exit status becomes 1 if nothing worse happened. The walk does not follow a symlink to a directory, so a link back to a parent does not loop. The symlink is listed as a name.

```sh
ls -R -- dir
```

People add `-r` and get a reversed single directory. The capital letter is the walk. `-R` still hides dotfiles, so a hidden directory is not entered and not shown. The headers and the blank lines are not filenames. A long listing of a tree is `ls -lR`, and every directory in it has its own `total` line. Output order inside each directory is still the normal sort, not the order of the walk.

## List only directories
Also asked as: list only directories; ls directories only; ls -d */; folders but not files; no flag for directories only
`ls` has no flag that means "directories only." `-d` stops `ls` from opening a directory operand. The shell glob `*/` expands to the directories in the current directory, and `-d` makes `ls` print those names instead of their contents. Do not quote `*/`. If you quote it, the shell does not expand it.

```sh
ls -d -- */
```

`*/` misses directories whose names start with a dot, and it also misses the current directory. If the folder contains no visible directories, bash leaves the literal `*/` in place, `ls` looks for a file of that name, and the status is 2. `ls -l | grep '^d'` is the fragile version. It depends on the English long format, it counts the `total` line as text, and a newline in a name splits a row. `-F` and `-p` mark directories in a full listing. They do not filter it.

## Mark directories and links on the name
Also asked as: ls -F; ls -p; trailing slash on directories; classify file type; asterisk for executable; symlink at sign
`-F` appends a type character to each name. `/` is a directory, `@` is a symlink, `*` is an executable file, `|` is a pipe, and `=` is a socket. `-p` appends `/` to directories and adds nothing else. Neither flag filters the listing. They only decorate the names `ls` was already going to print.

```sh
ls -F -- dir
```

```sh
ls -p -- dir
```

Use `-F` when you want every type mark. Use `-p` when the only mark you want is a slash on real directories. A symlink to a directory is `@` under `-F` and has no slash under `-p`, because the entry itself is a symlink. People expect `dirlink/` and get `dirlink@`. `-L` classifies the target instead. The extra character is not part of the filename. A script that reads `-F` output and then opens `name/` is opening a different path from `name` only when that slash was added.

## Color the names
Also asked as: ls --color; ls --color=auto; ls --color=always; color in a pipeline; macOS ls -G; GNU ls -G no group
`--color=auto` colors names only when stdout is a terminal and `TERM` is not `dumb`. Directories, symlinks, and executables are the usual colored types. Ordinary files are often left plain. `--color=always` writes the color codes even into a pipe or a file. The palette comes from `LS_COLORS`. With no variable set, GNU ls uses a built-in default once color is actually enabled.

```sh
ls --color=auto -- dir
```

```sh
ls -lG -- file
```

Use `--color=auto` for a person at a terminal. Do not use `--color=always` when the next command is `grep` or a script. The escape bytes become part of the text and the name no longer matches. On GNU ls, `-G` does not turn color on. In a long listing it drops the group column. On macOS, `-G` is the color switch, and `--color` is not. Copying either habit onto the other system does the wrong thing. BusyBox often has no color, or only a small `--color`.

## Follow a symlink, or show the link
Also asked as: ls symlink; ls -l shows the arrow; ls -H; ls -L; dereference; broken symlink exit status
A symlink to a file is listed as a name. A symlink to a directory is opened, and `ls` prints the directory's contents. `-l` changes that. `ls -l` on a symlink prints the link itself, with type `l` and ` -> ` plus the target text, and does not follow it. `-d` also prints the symlink rather than the target's contents. A broken symlink is still a successful listing. The link exists even when the target does not.

```sh
ls -ld -- file
```

```sh
ls -ldH -- file
```

Use `-ld` when you want the link. Use `-H` when the operands you typed should be followed, so a symlink to a directory is described as a directory. `-L` follows every symlink `ls` looks at, including ones inside a directory, not only the names on the command line. `-L` on a broken link fails with "No such file or directory" and exits 2. Without `-L`, that same name exits 0. `-H` is "the arguments." `-L` is "all of them."

## Show the full time
Also asked as: ls --full-time; ls --time-style; full timestamp; modification time with seconds; macOS ls -T; GNU ls -T tab size
The clock in a default `-l` line is an abbreviation. Recent files show month, day, and hour-minute. Older files show month, day, and year, and drop the time of day. `--full-time` is the GNU long form of `-l`: date, time, nanoseconds, and timezone. `--time-style=long-iso` is the shorter GNU form, `YYYY-MM-DD HH:MM`, still with `-l`. Neither flag exists on macOS or on BusyBox.

```sh
ls -l --full-time -- file
```

```sh
ls -lT -- file
```

Use `--full-time` on GNU ls. Use `-lT` on macOS, where `-T` means "print the complete time." On GNU ls, `-T` is tab width. `ls -lT file` exits 2 with "invalid tab size" because `file` was read as the number of columns. The time `--full-time` prints is still the modification time unless you also selected access time or change time. It is not the birth time of the file.

## Put directories before files
Also asked as: ls --group-directories-first; directories first; folders at the top; group directories before files
`--group-directories-first` splits the listing into directories and then everything else. Inside each group, the normal sort still applies, so the directories are alphabetical unless you also passed `-t` or `-S`. A symlink to a directory is grouped with the directories. The flag does not hide files and it does not imply `-l`. It is GNU. macOS and BusyBox do not have it.

```sh
ls --group-directories-first -- dir
```

`-U` and `-f` turn the grouping off, because they turn sorting off. People expect this flag to recurse or to list only directories. It does neither. It also does not pull directories out of subdirectories. Each directory `-R` prints is grouped on its own. Hidden directories stay hidden unless `-a` or `-A` is present, so a dot directory will not appear in the directory group.

## Skip names that match a pattern
Also asked as: ls -I; ls --ignore; ls --hide; exclude a glob; do not list a pattern; ignore backup files
`-I` takes a shell glob, not a regular expression, and drops directory entries that match it. The pattern is matched against the whole name, so `*.txt` drops names that end in `.txt` and does not drop `file.txt.bak`. Quote it. `--hide` uses the same kind of pattern. Both are GNU. `-B` is the special case that hides names ending in `~`.

```sh
ls -I 'pattern' -- dir
```

```sh
ls --hide='pattern' -- dir
```

Use `-I` when the names should stay hidden even in a full listing. Use `--hide` when you still want `-a` or `-A` to bring them back. `--hide` is ignored once `-a` or `-A` is set. `-I` keeps filtering. Neither one removes an operand you typed on the command line. `ls -I '*.txt' -- file.txt` still prints `file.txt`, because you named it. The filters apply to names `ls` found by opening a directory, including names found with `-R`.

## Quote names a person can paste
Also asked as: ls quoting; ls -Q; ls -N; names with spaces; newline in a filename; --quoting-style=shell-escape; literal names in a pipe
On a terminal, GNU ls quotes names that the shell would misread. A space becomes `'a b'`. A newline is shown as an escape, not as a line break. In a pipe or a redirect, the default is literal. The space is a real space and a newline is a real newline, so the next line-based tool sees two records. `--quoting-style=shell-escape` forces the terminal style even in a pipe. The result is text you can paste to bash.

```sh
ls -1 --quoting-style=shell-escape -- dir
```

```sh
ls -1N -- dir
```

Use `shell-escape` when a person or a shell will read the names. Use `-N` when you want the raw bytes and you already know no name contains a newline or a leading dash. `-Q` is a third style. It wraps every name in double quotes and backslash escapes, which is not shell syntax. `'a b'` and `"a b"` are not interchangeable once the name contains a quote or a newline. Quoting does not change which names are listed. It changes how they are printed.

## Count the entries
Also asked as: count files in a directory; ls | wc -l; how many files; ls -1 wc; total line counted by mistake
`ls` does not print a count. `wc -l` counts lines, so the listing has to be one name per line first. `-1` forces that. `-A` includes dotfiles and leaves out `.` and `..`, which is usually what "how many files" means. The pipeline's status is the status of `wc` unless the shell has `pipefail`. An empty directory prints no lines, and `wc -l` prints 0.

```sh
ls -1A -- dir | wc -l
```

`ls -l | wc -l` is wrong by at least one, because of the `total` line, and by more when you did not pass `-A`. A name that contains a newline counts as more than one. Column output counts as one line for several names, which is why `-1` is there even though a pipe is often already a single column. The count is names in that directory, not a recursive total. `-R` would also count the `dir:` headers.

## List a name that starts with a dash
Also asked as: ls file starting with a dash; ls -- -file; invalid option; ls ./-file; name looks like a flag
A operand that starts with `-` is parsed as a flag. `ls` then fails with "invalid option" and exits 2, even though a file of that name exists. `--` ends option parsing. The next word is a path no matter what it starts with. `./-file` is the other way to make the same name not look like a flag.

```sh
ls -- -file
```

```sh
ls ./-file
```

Use `--` when you want the name exactly as it is stored. Use `./` when you are already talking about the current directory and the next command in the pipeline does not understand `--`. Do not put `--` between a flag that takes an argument and that argument. It belongs before the paths. The same rule applies to `-I` patterns only in the sense that the pattern is an argument of `-I` and must sit next to it, not after `--`.

## Show inode numbers
Also asked as: ls -i; ls -li; inode number; which files are hard links; index number of a file
`-i` prints the inode number before the name. With `-l`, the number is the first column of the long line. An inode is the filesystem's id for the file data, not a path. Two names with the same inode on the same filesystem are hard links to one file. The number is not unique across disks, and it is reused after the file is removed.

```sh
ls -li -- file
```

People treat the inode as a stable global id or as the position of the name in the directory. It is neither. `ls -i dir` prints the inodes of the names inside `dir`, not the inode of `dir`, unless you also pass `-d`. A symlink has its own inode. Without `-L`, you see the link's number, not the target's. The column is easy to confuse with the link count in `-l`, which is how many directory entries share that inode, not the inode itself.

## Show numeric owners
Also asked as: ls -n; numeric uid gid; owner is a number; ls without passwd names; long listing with ids
`-n` is a long listing that prints numeric user and group ids instead of names. The rest of the `-l` line is unchanged: mode, link count, size, time, and name. Use it when the name lookup is the thing you do not trust, which is common in a container or a chroot whose `/etc/passwd` does not know the id that owns the file.

```sh
ls -n -- file
```

Plain `-l` prints the number too, but only when it cannot resolve the name, so the same file can show a name on one machine and a number on another. `-n` makes that choice explicit. It does not change the owner. It does not show the numeric mode. GNU `-G` and `-o` drop the group name from a long listing. They are not `-n`. On macOS, `-n` also means numeric ids. The GNU long option is `--numeric-uid-gid`.

## Sort numbers inside names
Also asked as: ls -v; natural sort; version sort; a1 a2 a10; numeric order of filenames; GNU version sort
`-v` sorts names by the numbers embedded in them. `a1`, `a2`, and `a10` come out in that order. The default alphabetical sort puts `a10` before `a2`, because `1` is before `2` at the first character that differs. `-v` is GNU. macOS and BusyBox do not have it. It replaces the name sort. It does not replace `-t` or `-S` unless it is the sort you selected.

```sh
ls -v -- dir
```

"Version" here means runs of digits in the filename, not a guarantee about every versioning scheme. Leading zeros and mixed suffixes still need a look. `-v` does not show hidden names and does not walk subdirectories. Locale alphabetical order is the thing people are usually trying to escape. `-v` is the ls switch for that. Piping to `sort -V` is the same idea only when `ls` was already one raw name per line.

## Leave the names unsorted
Also asked as: ls -U; ls -f; directory order; do not sort; unsorted and hidden files; fastest ls of a huge directory
`-U` prints directory entries in the order the directory stores them and does not sort. Dotfiles stay hidden. `-f` is unsorted too, and it also turns on `-a`, so `.`, `..`, and every dotfile appear. The order is whatever the filesystem returns. It can change after files are added or removed. Neither flag is a stable "creation order."

```sh
ls -U -- dir
```

```sh
ls -f -- dir
```

Use `-U` when you only want to skip the sort on a huge directory and you still want the normal hiding rules. Use `-f` when you want that order and the hidden names, because `-f` implies `-a`. `-f` is easy to misread as "files only" or as "full time." It is neither. `--group-directories-first` has no effect together with `-U` or `-f`, since there is no sort left to group.

## Show access time or status-change time
Also asked as: ls -u; ls -c; access time; ctime versus mtime; inode change time; ls --time=birth; creation time
`-l` shows modification time, the time the contents last changed. `-u` selects access time instead. `-c` selects ctime, the time the inode changed. That is chmod, chown, rename, or a link-count change, not the time the file was created. With `-t`, the selected clock is also the sort, newest first. `ls -ltc` sorts by ctime. `ls -l -c` without `-t` shows ctime but still sorts by name.

```sh
ls -ltc -- dir
```

```sh
ls -ltu -- dir
```

Use `-c` when you care that metadata changed. Use `-u` for the last access. On Linux, access time is often left stale by relatime or noatime, so `-u` can show a time from when the file was created or copied. Birth time, when you actually want creation, is GNU `--time=birth` together with `-l`. Filesystems that do not store a birth time show a dash instead of a clock. macOS has no `--time=birth`. ctime is not a substitute for it.

## Tell a missing path from an empty directory
Also asked as: ls exit code; ls exit status; cannot access No such file; empty directory prints nothing; ls failed; broken symlink status
A path that does not exist prints `cannot access` on stderr and exits 2. `ls` does not print a listing for that path. An empty directory exits 0. Without `-l` it prints nothing at all, which is easy to read as a failure. With `-l` it prints `total 0` and still exits 0. A usage error, such as an unknown flag, is also exit 2.

```sh
ls -- file
```

A subdirectory that `-R` cannot enter is the smaller failure. `ls` prints the error, keeps walking, and exits 1 if no command-line path failed. A broken symlink named on the command line exits 0, because the link itself is there. The same name under `-L` exits 2, because the target is not. Scripts that treat empty output as "missing" are wrong. Check the status. Scripts that treat any non-zero status as "the directory is empty" are wrong the other way.

## Separate names with a NUL
Also asked as: ls --zero; ls -0; null separated names; ls and xargs -0; filename with a newline; GNU ls zero
`--zero` ends each name with a NUL byte instead of a newline. Spaces, quotes, and newlines inside a name stay inside that record. Use it with one directory and with a reader that understands NULs, such as `xargs -0`. It is GNU, and only in coreutils 9 and newer. macOS and BusyBox do not have it. There is no `-0` short flag.

```sh
ls -1 --zero -- dir
```

The gotcha is more than one directory operand. `ls` then inserts a `dir:` header, and that header is not a clean extra record, so the NUL stream is no longer a list of names. Quoting flags do not matter once the separator is NUL. The bytes of the name are written as stored. `--zero` still hides dotfiles unless you add `-a` or `-A`, and it still lists the contents of a directory operand rather than the directory name unless you add `-d`.
