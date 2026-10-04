# find

`find` walks a directory tree and prints the paths that match tests you give it, or runs an action on those paths. It does not search inside file contents (that is `grep`), it does not sort results, and it does not start at your home directory unless you say so. With no path it starts at `.` and prints every entry it can read, including directories. The expression is a list of tests and actions combined with implicit AND, so later tests only see paths that already matched.

Basic form: find . -name 'pattern'

Paths come first. Anything that looks like a test (`-name`, `-type`) is part of the expression, not a path. The shell expands `*`, `?`, and braces before `find` sees them, so quote every pattern and quote the `{}` and `\;` of `-exec`. A path that starts with `-` or is meant to be a path after options needs a `--` only in the rare case you pass a leading-dash path; tests themselves are options and must not follow `--`. GNU find (findutils on Linux) is assumed here. macOS and other BSD find lack `-printf`, `-regextype`, and some `-exec` forms. BusyBox find is a smaller subset: it often has `-name`, `-type`, `-mtime`, and `-exec`, and often lacks `-delete`, `-printf`, and `-regextype`. Exit status is 0 if the walk finished, even when nothing matched. It is greater than 0 if a start path was missing, a directory could not be read, or an `-exec` command failed in a way find treats as an error. No match is not an error.

## Find files by name
Also asked as: find file by name; find -name; search for a filename; find files named; match a basename
`find` compares `-name` to the last component of the path, not the full path, and the pattern is a shell glob (`*`, `?`, `[]`), not a regular expression. The match is case-sensitive. Quote the pattern or the shell expands it in the current directory and `find` never sees the wildcard.

```sh
find . -name 'pattern'
```

People confuse this with matching a path fragment. A file `dir/pattern` matches `-name 'pattern'`. It does not match `-name 'dir/pattern'`. Use `-path` or `-wholename` when the directory part matters. Hidden names are included. A leading `.` in the pattern is not special to `find`.

## Find files by extension
Also asked as: find all files with extension; find *.log; find files ending in; find -name '*.txt'; files of a type by suffix
The extension is just part of the basename glob. `-name '*.log'` matches `a.log` anywhere under the start path and also matches a directory named `a.log` unless you add `-type f`. The shell must not see the `*`.

```sh
find . -type f -name '*.log'
```

A second suffix needs its own pattern. `*.tar.gz` is one glob, not two extensions. `-name '*.gz'` also matches `notes.gz.bak` only if the name actually ends in `.gz`. It does not. Names like `.log` with nothing before the dot still match `*.log`.

## Find only regular files
Also asked as: find -type f; only files not directories; skip directories; find files not folders
`-type f` keeps regular files. Directories are `d`, symbolic links are `l`, sockets are `s`, named pipes are `p`. Without a type test, `find` prints directories and files. The type is the type of the directory entry itself. A symlink to a file is `l`, not `f`, unless you add `-xtype` or `-follow`.

```sh
find . -type f
```

`-type f` does not mean "file I can read" and does not follow symlinks. Broken symlinks are still type `l`. Device nodes and sockets are not files. Combine with `-name` when you want both.

## Find only directories
Also asked as: find directories named; find -type d; list folders matching; find empty dirs later
`-type d` selects directories. The start path is tested too, so `find . -type d` prints `.` as well as its subdirectories. `-name` still applies only to the basename.

```sh
find . -type d -name 'pattern'
```

People expect this to skip the start directory. It does not. Add `-mindepth 1` on GNU and BSD find when `.` itself is noise. A symlink to a directory is type `l`, not `d`, unless you follow links.

## Limit how deep find walks
Also asked as: find -maxdepth; only this directory; do not recurse; find -mindepth; search one level
`-maxdepth 1` tests the start path and the entries directly inside it, and does not open subdirectories. `-maxdepth 0` tests only the start path. `-mindepth 1` skips the start path itself. Both are GNU and BSD. Place them before tests you want to short-circuit. They are global to the walk, not a per-directory filter.

```sh
find . -maxdepth 1 -type f -name 'pattern'
```

People put `-maxdepth` after `-name` and think it failed. It still works, but the walk is planned from the options as a whole. Omitting `-maxdepth` always recurses. BusyBox find usually has `-maxdepth`.

## Case-insensitive name search
Also asked as: find -iname; find ignore case; case insensitive filename; find -name case insensitive
`-iname` is the case-insensitive form of `-name`. The pattern is still a glob. GNU find and macOS find both have `-iname`. It does not fold locale-specific case for every Unicode edge case the way people expect. It folds ASCII case.

```sh
find . -iname 'pattern'
```

`-name` will not match `Readme` if you wrote `readme`. Do not use `-iregex` for a simple name. That tests the whole path and uses a different syntax. Quote the pattern. The shell's nocaseglob is irrelevant. `find` does the match.

## Match a path, not just the filename
Also asked as: find -path; find -wholename; match directory and name; find files under a folder name
`-path` (same as `-wholename`) matches the path `find` would print, using glob rules, against a pattern that can contain slashes. The string does not automatically get a leading `./` stripped in the comparison relative to how you started. Start at `.` and the path looks like `./dir/file`.

```sh
find . -path '*/dir/pattern'
```

`-name 'dir/pattern'` never matches, because the basename has no slash. `-path` is not a regular expression. `*` does not cross nothing special. It can cross slashes. Anchor with care. A pattern without a leading `*` fails to match `./dir/pattern` if you started at `.` and wrote `dir/pattern`.

## Find files modified in the last N days
Also asked as: find -mtime; files changed recently; modified in the last 7 days; find files newer than n days
`-mtime -7` means the file's modification time is less than 7 whole days ago. The unit is 24-hour periods, rounded. Negative means "more recent than", positive means "older than", and a bare number means "exactly that many days, rounded down". It is not calendar days and not hours.

```sh
find . -type f -mtime -7
```

People use `-mtime 7` and get only files in a single day-wide bucket about a week ago. Use `-mmin -30` for minutes. Content edits update mtime. `touch` does too. Access time is `-atime`, and on Linux it is often unreliable because of relatime or noatime.

## Find files larger or smaller than a size
Also asked as: find -size; files bigger than; find files over 100M; find -size +1G; smaller than
`-size +100M` is strictly larger than 100 mebibytes (units are 1024-based: `c` bytes, `k` kibibytes, `M` mebibytes, `G` gibibytes). A leading `+` is greater than, `-` is less than, and no sign is exactly that many units, rounded up. GNU find accepts this. BSD find uses the same idea with slightly different suffix letters. Prefer `M` on GNU.

```sh
find . -type f -size +100M
```

The gotcha is rounding. `-size 1M` is not "about a megabyte". It is one size unit after rounding. Sparse files report the apparent size, not disk blocks, unless you use `-size` with no unit tricks. For blocks, GNU documents `b`. Do not confuse `-size` with `du`.

## Find files newer or older than another file
Also asked as: find -newer; files modified after a file; newer than timestamp; find -newermt
`-newer file` is true if the current path was modified more recently than `file`. The reference can be any path. GNU find also has `-newermt '2024-01-01'` to compare against a date string. BSD find has `-newermt` on macOS as well. The reference file must exist for `-newer`.

```sh
find . -type f -newer file
```

People pass a date to `-newer` and get an error because `-newer` wants a file. Use `-newermt` for a timestamp. `-newer` compares mtime only. There is `-anewer` and `-cnewer` for access and status-change time. Status change is not the same as content edit.

## Find empty files and empty directories
Also asked as: find -empty; zero byte files; empty folders; find empty directories
`-empty` is true for zero-length regular files and for directories with no entries other than `.` and `..`. GNU and BSD find both have it. BusyBox often does too. It does not mean "directory whose files are all empty".

```sh
find . -type d -empty
```

A directory that contains only an empty subdirectory is not empty. A symlink to an empty file is not `-empty` unless you follow it. Use `-type f -empty` when you only want zero-byte files. `-size 0` is the file-only equivalent and does not select empty directories.

## Find files owned by a user or group
Also asked as: find -user; find -group; files owned by; find by uid; find -uid
`-user` takes a name or a numeric id. `-group` does the same for the group. `-uid` and `-gid` take numbers only. The comparison is the owner stored in the inode, not who can read the file.

```sh
find . -user 'user'
```

A deleted account still owns files. Matching the name fails if the name is gone, so use `-uid` with the number. `find` does not check your current permissions beyond being able to stat the entry. NFS root squashing and id maps can make the printed owner differ from the server.

## Find files by permission bits
Also asked as: find -perm; world writable files; find mode 644; find -perm -111; executable files by mode
`-perm 644` matches that mode exactly, including no extra bits. `-perm -644` means "all of these bits are set" and allows extra bits. `-perm /644` on GNU means "any of these bits are set". The leading slash form is GNU. BSD uses `-perm +644` for the any-bits form, which GNU still accepts but may warn about.

```sh
find . -type f -perm -644
```

People write `-perm 644` and miss files that are `664` or `755`. Exact match is rarely what a search for "writable" wants. Setuid is `-perm -4000`. The mode test does not mean "I can execute it". Access also depends on owner and path permissions. GNU `-executable` is a different test and is not portable.

## Skip a directory
Also asked as: find -prune; exclude directory; do not search node_modules; find exclude folder; skip .git
`-prune` is the action that stops `find` descending into the directory it just matched. You must place it so the directory matches before the rest of the expression, and you must OR it so the pruned directory is not also required to match the later tests. Without `-prune`, `-path` filters names after the walk has already entered.

```sh
find . -path './skip' -prune -o -type f -name 'pattern' -print
```

The gotcha is operator precedence and the missing `-print`. Implicit AND binds tighter than `-o`. If you omit `-print` on the right side, GNU find's default print does not apply to an all-OR expression, and you get no output. Quote or write the path the way `find` prints it (`./skip`, not `skip`, when you started at `.`). `-prune` does not delete anything.

## Do not follow symbolic links, or do follow them
Also asked as: find -follow; find -L; do not follow symlinks; find -P; stay on this filesystem
The default (`-P`) does not follow symlinks. A symlink is visited as itself. `-L` (or the older `-follow`) follows symlinks to directories and can loop. GNU find detects loops and warns. `-mount` or GNU `-xdev` stays on the same filesystem as the start path and will not cross into other mounts.

```sh
find . -xdev -type f -name 'pattern'
```

People use `-L` to find a target name and then get duplicate paths and surprising `-type f` hits through links. `-type l` finds the links themselves. Following links also changes which permission errors you see. BusyBox supports `-follow` more reliably than `-L`.

## Find symbolic links and broken links
Also asked as: find -type l; broken symlinks; dangling links; find -xtype; dead links
`-type l` prints symbolic links, broken or not. A broken link is a symlink whose target does not exist. GNU find can select those with `-xtype l`: the entry is a symlink, and the type of the followed target is still `l` because the follow failed. macOS find also has `-type l`. Confirm `-xtype` before relying on it on BusyBox.

```sh
find . -type l
```

```sh
find . -xtype l
```

Use the first form for every symlink. Use the second on GNU find for dangling ones. People test `-type l` and then `! -exec test -e {} \;`, which works but is slower and treats a link to an empty path oddly. A link to a file is not broken just because you cannot read the target.

## Run a command on each match
Also asked as: find -exec; find exec; run command on each file; find {} \; ; find -exec batch
`-exec` replaces `{}` with the path and runs the command from the start directory's process, not through a shell, until `\;`. You must quote `{}` and `\;` so the shell does not eat them. The command runs once per file. If the command returns non-zero, find records an error but keeps going.

```sh
find . -type f -name 'pattern' -exec command -- '{}' \;
```

The gotcha is the escaped semicolon. Without `\;` or `+`, find reports a missing argument. `{}` must be a separate argument. Embedding it in a shell string does nothing unless you actually invoke `sh -c`. Paths with spaces are safe here because there is no shell. They are not safe if you pipe to `xargs` without `-0`.

## Run a command on many files at once
Also asked as: find -exec +; find plus form; batch exec; xargs alternative; find {} +
The `+` form of `-exec` appends as many paths as fit on the command line and runs the command fewer times. `{}` must be the last argument before `+`. This is the boring replacement for `xargs` when you do not need extra filtering. GNU, BSD, and current BusyBox support it.

```sh
find . -type f -name 'pattern' -exec command -- '{}' +
```

Use `\;` when the command must see exactly one file, or when `{}` is not the final argument. Use `+` to delete or checksum in batches. `+` does not invoke a shell. A command that only accepts one file will misread the extra arguments. There is no automatic `--` inside the command. Add it if the command needs it.

## Delete the files that matched
Also asked as: find -delete; find and remove; delete found files; find -exec rm; remove matching files
GNU find's `-delete` removes files and empty directories after it has finished reading a directory. It implies `-depth`, so contents go before the directory. It refuses to delete `.` itself. It does not spawn `rm`. macOS find also has `-delete`. Older BusyBox often does not. If you are not sure, use `-exec rm`.

```sh
find . -type f -name 'pattern' -delete
```

```sh
find . -type f -name 'pattern' -exec rm -f -- '{}' +
```

Use `-delete` on GNU and macOS when the test is simple. Use `rm` when you need `-i`, or when the find is BusyBox. The gotcha is testing the expression with `-print` first. `-delete` is an action that succeeds as a test. Combined carelessly with `-o`, it can delete more than the names you printed. Never add `-delete` on the same run you are still editing the tests.

## Print null-terminated names for xargs
Also asked as: find -print0; xargs -0; filenames with spaces; find print0; null delimited
`-print0` prints each path followed by a NUL byte instead of a newline. Pair it with `xargs -0`. Newlines, spaces, and quotes in names are then safe. GNU and BSD find have `-print0`. It is an action, so it counts as "you printed something" and suppresses the default print.

```sh
find . -type f -name 'pattern' -print0
```

People pipe `find` to `xargs` without `-0` and then watch a file named `a b` split into two arguments. `-print` is the newline form and is the default when the expression has no action. Prefer `-exec cmd {} +` when you do not need a pipeline. `-print0` is not useful in a terminal you are reading by eye.

## Print a custom format
Also asked as: find -printf; print size and path; find format output; custom find output
GNU `-printf` prints a format instead of the bare path. `%p` is the path, `%s` is the size in bytes, `%TY-%Tm-%Td` is a modification date, and `\n` is a newline you must include yourself. This is GNU-only. macOS and BusyBox do not have `-printf`. Do not invent a long option for it.

```sh
find . -type f -name 'pattern' -printf '%s %p\n'
```

The gotcha is the missing newline. Without `\n`, every record sticks together. `%p` does not escape newlines in names. If a name can contain a newline, `-printf` is the wrong tool. Use `-print0`. For a portable size listing, use `-exec ls -l {} +` and accept that `ls` output is harder to parse.

## Ignore unreadable directories
Also asked as: find permission denied; hide find errors; find 2>/dev/null; cannot read directory; stderr noise
`find` writes "Permission denied" to stderr and continues with the rest of the tree. The exit status is non-zero if any directory could not be read. Redirecting stderr hides the message and does not change the walk. It also hides real errors, such as a bad start path.

```sh
find . -type f -name 'pattern' 2>/dev/null
```

People think `2>/dev/null` makes find skip those directories faster or succeed. It only drops the text. The exit status is still non-zero, which breaks `set -e` and pipelines that check status. There is no portable "skip quietly and exit 0" flag. Start from a directory you can read if the status matters.

## Combine tests with AND and OR
Also asked as: find -o; find OR; find AND; two name patterns; either extension; find \( \)
Adjacent tests are ANDed. `-o` is OR and has lower precedence, so you almost always need `\( \)` which the shell must not eat. A parenthesized OR that contains no action does not get the default `-print`. Add `-print` yourself.

```sh
find . -type f \( -name '*.log' -o -name '*.txt' \) -print
```

The gotcha is precedence. `find . -name '*.log' -o -name '*.txt' -type f` applies `-type f` only to the second branch, and default printing only happens for the branch that has no action, which is not the branch you think. Quote the parentheses. BusyBox supports `-o` and grouping.

## Exclude a name
Also asked as: find -not; find ! -name; not this name; exclude pattern; find negation
`!` and `-not` negate the next test. `-not` is GNU and BSD. `!` is the portable operator. Negation binds tightly. Quote `!` because history expansion in interactive bash can eat it. Exclude a directory with prune if you want to avoid walking it. Negation alone still walks it.

```sh
find . -type f ! -name 'pattern'
```

`! -name '*.o'` still prints directories unless you also asked for `-type f`. `! -path` does not stop the descent. People negate `-name '*pattern*'` and then wonder why `./pattern/file` still appears. `-name` never saw the directory part.

## Regular expression on the whole path
Also asked as: find -regex; find -regextype; posix regex path; find -iregex; emacs regex default
`-regex` matches the whole path, not the basename, and the default syntax on GNU find is Emacs regular expressions, not POSIX and not a glob. `-regextype posix-extended` switches syntax and is GNU-only. The path includes the `./` prefix when you started at `.`. A pattern that does not match the entire string fails.

```sh
find . -regextype posix-extended -regex '.*pattern'
```

People write `-regex 'pattern'` and get nothing, because the path is `./dir/pattern` and the match is not unanchored the way `grep` is. It is anchored to the whole path. macOS find uses a different default regex and has no `-regextype`. Use `-name` or `-path` unless you actually need alternation or character classes a glob cannot express.

## Count matching files
Also asked as: count files found; find | wc -l; how many files; count find results; number of matches
`find` does not have a count action. It prints paths. `wc -l` counts newlines, so a filename that contains a newline is counted as more than one, and a missing trailing newline is not an issue because find adds one. No matches print nothing, and `wc -l` prints 0. The exit status of find is not the count.

```sh
find . -type f -name 'pattern' -print | wc -l
```

The pipeline's status is the status of `wc`, not find, unless the shell has `pipefail`. Permission errors still land on stderr and do not increment the count. For names with newlines, count NULs with `-print0` and a tool that counts them, or use `-exec` and a counter. Do not use `grep -c` on find output. That counts matching lines inside the path text.

## Find files and search their contents
Also asked as: find and grep; find -exec grep; search inside found files; grep files by name; find then grep
`find` selects paths. `grep` searches contents. Combine them with `-exec` so names with spaces survive. GNU grep's `-r` already walks trees. Use find when the selection is a test grep does not have, such as size, mtime, or a pruned directory. `--` stops grep from treating a filename as an option.

```sh
find . -type f -name 'pattern' -exec grep -H -n -e 'pattern' -- '{}' +
```

People pipe find to `xargs grep` without `-0` and split names. `grep` with no file argument reads stdin, so a failed find that prints nothing makes grep wait for the keyboard if you connected them wrong. `-H` forces a filename even when only one file is passed. Binary files may print "Binary file matches" instead of lines. That is grep, not find.

## Stay out of other filesystems
Also asked as: find -xdev; find -mount; do not cross mounts; same filesystem; skip other disks
`-xdev` and `-mount` mean the same thing on GNU find: do not descend into a directory that is on a different filesystem than the path you started from. This skips `/proc`, `/sys`, and other mounts when you start at `/`, but only at the boundary. macOS has `-xdev`. BusyBox usually has `-xdev`.

```sh
find / -xdev -type f -name 'pattern'
```

Starting at `/` without `-xdev` walks every mounted disk and virtual filesystem, which is slow and prints permission errors. `-xdev` does not skip a directory that is merely a different folder on the same disk. Bind mounts can still surprise you because they may be a separate filesystem id.

## Find files changed in the last few minutes
Also asked as: find -mmin; modified in the last hour; minutes since modification; find mmin; recent files by minutes
`-mmin -60` selects files whose mtime is less than 60 minutes ago. The sign works like `-mtime`. Fractional rounding means a file touched 60 minutes and a few seconds ago may fall outside `-60`. GNU and BSD both have `-mmin`. BusyBox often has it.

```sh
find . -type f -mmin -60
```

People multiply days by 24 and use `-mtime` for a two-hour window. `-mtime` cannot express that. Access time is `-amin` and is often not updated. Status-change time is `-cmin`. A file that was only renamed may have a new ctime and an old mtime.

## Depth-first walk
Also asked as: find -depth; process files before directory; depth first find; delete directory tree order
`-depth` visits a directory's contents before the directory itself. `-delete` turns this on implicitly. You want it when a command must empty a directory before removing the directory. You do not want it when you prune, because prune decisions happen at the directory, and `-depth` has already entered it.

```sh
find . -depth -type d -name 'pattern' -print
```

The gotcha is combining `-depth` with `-prune`. Prune cannot stop a descent that already happened. Default order is pre-order: directory first, then contents. `-depth` is not a depth limit. That is `-maxdepth`.

## Start from more than one path
Also asked as: find multiple directories; two start paths; find path1 path2; search several roots
Every argument before the first expression is a start path. `find` walks each in order and applies the same expression. If one start path does not exist, find warns and still walks the others, then exits non-zero. There is no default path if you pass only an expression. A leading `-` looks like an expression.

```sh
find dir1 dir2 -type f -name 'pattern'
```

People write `find -name 'pattern' dir` and find treats `dir` as a path in the wrong place, or as a non-option it rejects, depending on version. Put paths first. A glob of start directories is expanded by the shell, so `find * -name 'pattern'` does not search hidden directories and does not use `*` as a pattern.n.
