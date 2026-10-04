# diff

`diff` compares two files line by line, or two directories by name, and writes the differences to standard output. It does not merge files, it does not edit either input, and it does not search for a pattern inside one file. The first operand is the old file. The second is the new file. Lines that exist only in the first are removals. Lines that exist only in the second are additions. With no difference, `diff` prints nothing. A directory operand is compared as a set of names, not as a dump of every byte, and it is not walked into subdirectories unless you pass `-r`.

Basic form: diff file file

Operands are paths. `-` means standard input, and only one side can be `-` in a normal run. A name that starts with `-` is a flag unless it comes after `--`. Quote a pattern you pass to `-x` or `-I`, or the shell expands it. GNU diff from diffutils on Linux is assumed here. macOS diff understands the common short flags `-u`, `-c`, `-r`, `-q`, `-b`, `-w`, `-i`, and `-N`. It often has no `--color` and no `--strip-trailing-cr`. BusyBox diff is smaller. Unified output and a recursive compare are often there. The long options and the ignore-regex flag often are not. Exit status is 0 when the inputs are the same, 1 when they differ, and 2 when something went wrong, such as a missing file or an illegal flag. A difference is not a failure of the program. A missing path is.

## Compare two files
Also asked as: diff two files; how to compare files; normal diff; diff old new; what diff prints
With no format flag, `diff` prints a normal diff. A change is a line such as `2,3c2,3`, then the old lines prefixed with `<`, a `---` separator, and the new lines prefixed with `>`. `a` is an addition and `d` is a deletion. Identical files print nothing and exit 0. Different files exit 1, even when the report is empty of anything but the hunks.

```sh
diff -- file file
```

People read `<` as "less than" and `>` as shell redirection. They are labels. The left file is the first argument, the right file is the second, and swapping them swaps every hunk. `diff` does not follow a third file. It does not know which one is "correct." A binary file does not get this line format. GNU diff says the files differ and stops the line listing unless you force text with `-a`.

## Show a unified diff
Also asked as: diff -u; unified diff; patch format; plus and minus lines; diff -U; git style diff
`-u` prints a unified diff, the form `patch` expects. The old file is marked `---`, the new file is marked `+++`, and each hunk starts with `@@`. A space in the first column is context that appears in both files. `-` is a line only in the first file. `+` is a line only in the second file. The default context is three lines. Identical files still print nothing.

```sh
diff -u -- file file
```

The timestamps on the `---` and `+++` lines are modification times, not the time you ran `diff`. A pipe or a process substitution may show a useless time. People count the `+` and `-` lines as the whole change and miss a line that was replaced: that is one `-` and one `+`, not an in-place edit. `-u` is not a word diff. To ignore a change in spaces you still need `-b` or `-w`. The exit status stays 1 when any hunk is printed.

## Show context around each change
Also asked as: diff -c; context diff; diff -C; copied context; bang lines in a diff
`-c` prints a context diff. Each file gets its own block. The old block starts with `***` and the new block with `---`. A space is a line that is unchanged. `!` marks a line that changed. `+` and `-` still mean added and removed. The default is three lines of context. `-C` takes a count, the same way `-U` does for unified output.

```sh
diff -c -- file file
```

People mix `-c` with `-u` and then try to read `!` as a unified minus. A context diff is a different format. `patch` accepts both, but a viewer that understands only unified hunks will not apply a `-c` file cleanly. Use `-u` unless you specifically need the older context form. `-c` does not mean "brief" and it does not mean "ignore case." Brief is `-q`. Ignore case is `-i`.

## Say only whether the files differ
Also asked as: diff -q; brief diff; files differ; do not show the hunks; quiet compare
`-q` prints one line when the files differ and nothing when they are the same. The line is `Files X and Y differ`. Directories get `Only in` lines as well. The exit status is still 0 for identical, 1 for different, and 2 for trouble. No hunks are printed, so you cannot see what changed.

```sh
diff -q -- file file
```

People use `-q` and then look only at the output. An identical pair is silent, which looks like a command that did nothing. Check the status if you need a yes or no. `-q` is not `-s`. `-s` is the opposite announcement: it prints a line when the files are the same. `-q` also does not suppress a binary-file message. Two different binary files still get a "differ" line and exit 1.

## Report files that are the same
Also asked as: diff -s; report identical files; files are identical; show matches not only changes
`-s` prints `Files X and Y are identical` when the two inputs have no differences. Without `-s`, a match is silent. Differences are still printed in the normal way, or as the brief line if you also passed `-q`. The status does not change. Identical is 0, different is 1.

```sh
diff -sq -- file file
```

People pass `-s` expecting it to mean "silent" or "suppress." It adds a report. It does not hide differences. On a directory compare it reports identical files as well as `Only in` and `differ`, so a large tree becomes much noisier. Use it when a script must print a positive match. Use `-q` alone when you only want the mismatches.

## Ignore letter case
Also asked as: diff -i; ignore case; case insensitive diff; diff capital letters; compare ignoring case
`-i` treats uppercase and lowercase letters as the same while comparing. A line that differs only by case produces no hunk. The files still differ if any other character differs. The output, when there is a real change, shows the original case from each file. `-i` does not fold the names of files in a directory unless you also pass `--ignore-file-name-case`, which is GNU.

```sh
diff -i -- file file
```

Case folding follows the locale. It is not a promise about every Unicode pair. A binary file is still binary. `-i` does not ignore spaces, and it does not ignore a blank line. Those are `-w`, `-b`, and `-B`. People combine "ignore case" with a belief that the exit status will be 0 whenever a human would call the text the same. A reordered paragraph is still a difference.

## Ignore changes in whitespace
Also asked as: diff -w; diff -b; ignore spaces; ignore all whitespace; ignore amount of space; trailing space
`-w` ignores all white space when comparing lines. `a b` and `ab` can compare equal. `-b` ignores only a change in the amount of white space, so `a b` and `a  b` compare equal, while `a b` and `ab` do not. `-Z` ignores white space at the end of the line only. None of these flags delete the spaces from the output. A hunk that is printed still shows the lines as they are in the files.

```sh
diff -w -- file file
```

```sh
diff -b -- file file
```

Use `-w` when every space and tab should be ignored. Use `-b` when you only want to ignore indentation or a double space, and a missing space should still count. Neither flag ignores a blank line. A line that is empty in one file and missing in the other is still a change unless you pass `-B`. Tabs and spaces are both white space. A non-breaking space is not. Carriage return is not white space to these flags. That is `--strip-trailing-cr`.

## Ignore blank lines
Also asked as: diff -B; ignore blank lines; empty lines differ; skip blank lines; ignore changes whose lines are blank
`-B` ignores a change whose lines are all blank. An inserted or deleted empty line produces no hunk. A line that contains spaces is not blank. A change that mixes a blank line with a changed word is not ignored, because not every line of that change is blank.

```sh
diff -B -- file file
```

People pass `-B` to mean "ignore white space." It does not. `a b` and `ab` still differ. Pair it with `-w` or `-b` when both empty lines and spacing should be ignored. `-B` also does not strip a trailing carriage return. Two files that differ only by blank lines exit 0 with `-B`. Two files that differ by a blank line and one other character still exit 1.

## Ignore changes that match a regex
Also asked as: diff -I; ignore matching lines; skip lines matching a pattern; ignore a header line; GNU ignore-matching-lines
`-I` takes a regular expression. A change is ignored only when every inserted and deleted line matches that expression. Context lines are not the test. One non-matching line in the hunk keeps the whole change. Quote the pattern. This flag is GNU. The regex is a basic regular expression unless you are sure of the build. Keep the pattern simple.

```sh
diff -I 'pattern' -- file file
```

People write `-I` to drop a timestamp line and still see the hunk, because a neighboring real edit was grouped into the same change. `diff` ignores the change only if all of its added and removed lines match. A pattern that matches the timestamp but not the edited line does nothing visible. `-I` is not exclude-a-file. Excluding a name in a directory compare is `-x`.

## Compare two directories
Also asked as: diff directories; diff two folders; Only in; files differ; compare folder names; diff dir dir
Given two directories, `diff` compares the names in each. A name in both is compared as a file. A name in only one is reported as `Only in left: name`. It does not enter subdirectories unless you pass `-r`. The first directory is the left side. The second is the right side. Identical names that are both files and have the same contents produce no line.

```sh
diff -- dir dir
```

People expect a recursive tree compare from plain `diff`. They get only the top level, and a subdirectory that exists on both sides is not opened. They also expect `diff` to match files that were renamed. It matches names, not contents. Two files with different names are `Only in`, even when the bytes are the same. The status is 1 if any name is missing or any common file differs. It is 2 if a directory cannot be read.

## Walk subdirectories
Also asked as: diff -r; recursive diff; diff -rq; compare directory trees; diff folders recursively
`-r` compares names in subdirectories as well. Each pair of directories is walked. Output is still `Only in` and `Files X and Y differ`, plus hunks unless you passed `-q`. A subdirectory that exists on only one side is reported as `Only in`. It is not descended on the other side, because there is nothing to pair it with.

```sh
diff -rq -- dir dir
```

`-rq` is the form that scales. Without `-q`, a recursive diff prints a full diff of every changed file, which is what you want for a patch and not what you want for a summary. `-r` follows symlinks to directories unless you pass `--no-dereference`, which is GNU. A symlink and a regular file with the same name are a difference. Permission errors on a subdirectory are trouble for that path and can make the status 2.

## Treat a missing file as empty
Also asked as: diff -N; new file; absent file as empty; Only in versus a real diff; unidirectional new file
`-N` treats a file that exists on only one side as an empty file on the other side. Instead of `Only in`, you get a real diff that adds or removes every line. That is the form you want when the result must be a patch that creates the missing file. It applies to directory compares. It is not a search for similar names.

```sh
diff -ruN -- dir dir
```

Without `-N`, a file that was added is only a note, and `patch` cannot create it from that note. With `-N`, the missing side is empty, so every line is an addition or a deletion. `--unidirectional-new-file` is the GNU variant that treats an absent first file as empty and still reports a file that exists only in the first directory as `Only in`. Use `-N` when either side may have new files. Use the unidirectional flag when you only want to turn "added on the right" into hunks.

## Skip names while comparing directories
Also asked as: diff -x; exclude a file; diff --exclude; skip a directory; diff -X; ignore a glob in a tree
`-x` takes a shell glob, not a regular expression, and skips matching names during a directory compare. The pattern is matched against the base name. Quote it, or the shell expands it before `diff` sees it. `-X` reads those globs from a file, one per line. Both apply to the names `diff` is about to compare, not to lines inside a file.

```sh
diff -rx 'pattern' -- dir dir
```

People use `-x` to ignore a line in a file. It does not. Line filtering is `-I`. A pattern of `*.o` skips object files at every level of an `-r` walk. It does not skip a directory's contents unless the directory name itself matches. Excluding `dir` skips that directory and, because it is not entered, everything under it. `-x` does not change a two-file compare. There are no names to exclude.

## Show the files side by side
Also asked as: diff -y; side by side; two columns; diff -y --suppress-common-lines; compare columns
`-y` prints the two files in columns. The left column is the first file. The right column is the second. A `|` between them marks a changed line. `<` marks a line only on the left. `>` marks a line only on the right. The default width is 130 columns. `-W` sets that width. Common lines are printed unless you pass `--suppress-common-lines`.

```sh
diff -y -- file file
```

```sh
diff -y --suppress-common-lines -- file file
```

Use plain `-y` when you want to read both files in place. Use `--suppress-common-lines` when you only want the rows that differ. Long lines are truncated to the width. They are not wrapped, so a difference past column 130 can sit in the cut-off part. `-y` is a display format. `patch` does not apply it. Use `-u` when the output has to be a patch.

## Choose how much context to print
Also asked as: diff -U; diff -U0; lines of context; unified context count; diff -C 0; no context
`-U` sets how many unchanged lines a unified hunk keeps. `-U 3` is the default `-u`. `-U 0` prints only the changed lines, which is easier to read and worse as a patch, because the context is what lets `patch` find the right place. `-C` is the same count for a context diff. The number is attached or passed as the next argument.

```sh
diff -u -U 0 -- file file
```

People write `-u0` and get a bad option, or they write `-U0` and it works on GNU diff because the number may be glued to the short flag. Prefer `-U 0` with a space so the count cannot be eaten by the cluster. Zero context does not mean the files were almost the same. It means the unchanged neighbors were hidden. The exit status is still 1 if any line differs.

## Ignore a Windows line ending
Also asked as: diff strip trailing cr; carriage return; CRLF versus LF; files differ only by line ending; --strip-trailing-cr
`--strip-trailing-cr` removes a trailing carriage return from each input line before comparing. A file written with CRLF and a file written with LF then compare equal if the text is otherwise the same. The flag is GNU. The output lines, if a real difference remains, are the stripped form in the comparison, and the report is still a normal diff of the text.

```sh
diff --strip-trailing-cr -- file file
```

Without the flag, every line can differ by a hidden `\r`, and `diff` prints what looks like identical lines. `-w` does not fix that. Carriage return is not white space to `-w` or `-b`. People also try `dos2unix` in a pipeline. That works, but it is a second copy of the file. The flag is the direct compare. It does not change the files on disk. A CR in the middle of a line is left alone.

## Compare files diff thinks are binary
Also asked as: diff binary files; Binary files differ; diff -a; treat as text; force a text diff
If a file contains a NUL byte in the portion `diff` inspects, GNU diff calls it binary. It prints `Binary files X and Y differ` and does not print hunks. The status is 1 if they differ and 0 if they are identical. `-a` forces a text compare. Every byte is then a character, and NULs can show up inside a hunk.

```sh
diff -a -- file file
```

People pass `-a` to a pair of images or object files and try to read the hunks. The hunks are not a useful edit script. They are a byte compare dressed as lines, split on newlines that may not be there. Use `-q` when you only need to know that two binary files differ. Use `cmp` when you need the first byte offset. `-a` is for a text file that was misdetected, not for a format `diff` understands.

## Read one side from standard input
Also asked as: diff stdin; diff -; diff a pipe; compare command output; one file is standard input
`-` as a file operand reads that side from standard input. The other operand is a path. The unified header shows `-` as the name of the piped side. `diff` reads the pipe until end of file, then compares. You cannot usefully put `-` on both sides of one `diff`. There is only one standard input.

```sh
diff -u -- file -
```

People redirect both ways or run `diff` with no operands and it fails with a usage error and exit 2. A directory cannot be read from stdin. The `-` has to be a file side. `--label` replaces the name printed in the header, which is GNU, and it does not change the comparison. A pipe that fails still looks like an empty file if you ignore the producer's status. `diff` only sees the bytes it was given.

## Read the exit status
Also asked as: diff exit code; diff exit status; diff returns 1; different is not an error; diff status in a script
`diff` exits 0 when it finds no differences, 1 when it finds some, and 2 when it cannot do the comparison. A missing file, a directory it cannot read, and a bad flag are 2. A one-line change is 1. In a shell with `set -e`, a different pair aborts the script unless you accounted for status 1. The printed hunks are not the status.

```sh
diff -q -- file file
```

People test the output for emptiness and ignore the status, or the reverse. Identical files with no `-s` print nothing and exit 0. A trouble case can also print nothing useful on stdout and exit 2, with the reason on stderr. `-q` does not change the codes. `-c` and `-C` do not either. `-C` here would be context. The quiet check flag of `sort` is not a `diff` flag. Do not read exit 1 as "diff crashed."

## Color the hunks
Also asked as: diff --color; diff --color=always; color a diff; color in a pipeline; GNU diff color
`--color=auto` colors the hunk markers when standard output is a terminal. `--color=always` writes the color codes even into a pipe. `--color=never` turns them off. Plain `--color` means auto. The flag is GNU. The colors mark added, removed, and changed text. They do not change which lines are differences.

```sh
diff --color=auto -u -- file file
```

People pass `--color=always` into `patch` or into a file they will apply later. The escape bytes become part of the diff and the patch no longer applies. Use `auto` for a person, and do not use `always` when the next tool must read the diff. macOS diff often has no `--color`. A pager that does not interpret color shows the raw escapes. The exit status is unchanged.

## See which function a change is in
Also asked as: diff -p; show C function; diff function header; which function changed; diff -F
`-p` looks backward from each change for a C function header and prints that name in the hunk. It is a hint for a reader, based on a simple pattern, not a parser. `-F` takes a regular expression and uses the most recent matching line as that label instead. Both are GNU-friendly options on diffutils. They do not limit the compare to that function.

```sh
diff -up -- file file
```

People expect `-p` to mean "patch" or "brief." It only decorates the hunk header. A macro, a nested function, or a language that is not C produces a useless label or none. The compare is still the whole file. `-p` does not ignore comments and it does not require the files to compile. Quote the regex if you use `-F`. An unquoted pattern is expanded by the shell.
