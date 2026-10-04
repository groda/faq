# sed

`sed` reads lines, runs a script on each one, and writes the result to standard output. It does not edit the file unless you pass `-i`. The default script action is to print the line after other commands run, so an unchanged line still appears. With no file, `sed` reads standard input and waits. Several files are one stream unless you pass `-s`. GNU sed on Linux is assumed here. The default regular expression is basic, not extended. `+`, `?`, `|`, and parentheses are literal unless you escape them or pass `-E`. macOS sed is BSD sed. `-i` there requires a backup suffix argument, even an empty one, and it has no `-z`. BusyBox sed has `s`, `d`, and `-n`, and its `-i` and `-E` vary by build. Exit status is 0 when the script ran, including when nothing matched. It is non-zero when the script is illegal or a named file cannot be read. No match is not an error.

Basic form: sed 's/pattern/pattern/' -- file

The first non-option argument is the script, unless you used `-e` or `-f`. Later arguments are files. Single-quote the script or the shell eats backslashes and expands `$`. `--` before a file stops a leading dash from looking like a flag. `/` is the usual `s` delimiter. Use another character when the pattern contains slashes. A line is the unit. A match does not cross a newline unless you build a multi-line pattern space yourself.

## Replace the first match on each line
Also asked as: sed substitute; sed s///; replace text; first occurrence; sed search and replace
`s/pattern/replacement/` replaces the first match on each line and prints every line. Lines that do not match are printed unchanged. The pattern is a basic regular expression. A dot matches any character. `*` repeats the previous atom. The replacement is text, not a regex. `&` in the replacement means the whole match.

```sh
sed 's/pattern/pattern/' -- file
```

People expect every match on the line to change. Only the first does, unless you add `g`. People also expect the file to change. Standard output changes. The file does not, until `-i`. A pattern with a slash ends the command early. `s#pattern#pattern#` is the same command with a different delimiter. Quote the script. `sed s/a/b/ file` is split by the shell and is not one script.

## Replace every match on the line
Also asked as: sed g flag; replace all occurrences; global substitute; sed s///g; every match on a line
The `g` flag after the closing delimiter repeats the substitution on the rest of the line. It does not mean "every line." Every line is already processed. It means every non-overlapping match on that line. The rest of the line is still printed, and lines with no match are still printed.

```sh
sed 's/pattern/pattern/g' -- file
```

A match that can be empty, or a `.*` that eats the rest of the line, makes `g` look like it did nothing further. `.*` is greedy. It runs to the end of the line, then gives back only what it must. To replace a literal dot, write `\.`. To replace a literal star, write `\*`. `g` does not cross newlines. A match split over two lines is two lines, and neither is fully matched.

## Print only lines that matched
Also asked as: sed -n; sed -n p; quiet sed; print only matches; suppress automatic printing
`-n` turns off the automatic print. `p` prints the pattern space. `sed -n '/pattern/p'` prints only lines that match. `sed -n 's/pattern/pattern/p'` prints only lines where the substitution happened, and it prints the line after the change. Without `-n`, `p` prints a matching line twice, because the automatic print still happens.

```sh
sed -n 's/pattern/pattern/p' -- file
```

People add `p` to see the result and get duplicates. The automatic print is the reason. `-n` is the off switch. `-n` without a `p`, or without another command that prints, produces nothing and exits 0. That looks like a bad pattern. It is a script that never asked to print. No match is also empty output and status 0. A missing file is the non-zero case.

## Delete lines
Also asked as: sed d; delete matching lines; sed '/pattern/d'; remove lines; print all but
`d` deletes the pattern space and skips the rest of the script for that line, including the automatic print. `sed '/pattern/d'` prints every line that does not match. The match is a basic regular expression. A line that matches is gone from the output, not gone from the file.

```sh
sed '/pattern/d' -- file
```

People invert the test and delete the lines they meant to keep. `d` on a match removes it. To keep only matches, use `-n` and `p`. An empty pattern is not "blank lines." A blank line is `/^$/d`. A line of spaces is not empty. `d` does not squeeze a file in place. Without `-i` you are printing a filtered copy. The status is still 0 when every line was deleted.

## Edit a file in place
Also asked as: sed -i; in place edit; sed -i.bak; edit the file not stdout; GNU sed -i; macOS sed -i
`-i` writes the result back to the named file. GNU sed may take an optional backup suffix glued to the flag. `-i.bak` keeps `file.bak`. `-i` alone does not keep a backup. There is no second copy if the script was wrong. macOS sed requires the suffix argument. An empty suffix there is `sed -i ''`. GNU sed reads `-i ''` differently. Do not mix them.

```sh
sed -i 's/pattern/pattern/' -- file
```

```sh
sed -i.bak 's/pattern/pattern/' -- file
```

Use a backup suffix until the script is right. Use bare `-i` only when you can lose the original. `-i` refuses to read standard input as the file to edit. It needs a named file. A symlink is edited in place as the symlink's target only if you pass `--follow-symlinks`. Otherwise GNU sed may replace the symlink with a new regular file. That flag is GNU.

## Use extended regular expressions
Also asked as: sed -E; sed -r; extended regex; unescaped parentheses; alternation in sed; POSIX -E
`-E` selects extended regular expressions. `+`, `?`, `|`, and `()` then have their special meaning and do not need backslashes. `-r` is the older GNU spelling of the same switch. Prefer `-E` if you want the portable flag. macOS sed accepts `-E` and does not accept `-r`. Without `-E`, a `+` is a literal plus, and a group is `\(` `\)`.

```sh
sed -E 's/pattern|pattern/pattern/' -- file
```

People copy a `grep -E` pattern into `sed` and it matches the plus signs literally. The dialect has to be selected on `sed` as well. Extended mode does not turn on `g`, does not turn on `-i`, and does not change `.` so that it matches a newline. It still does not. Alternation is leftmost first. `a|ab` matches `a` at the start of `ab` and does not try `ab` unless the rest of the pattern forces it.

## Address a range of lines
Also asked as: sed line numbers; sed 1,10; sed address; only line 1; sed last line; sed $
A number selects a line. `$` is the last line. `1,10` is the inclusive range. The command after the address runs only on those lines. Other lines pass through and are printed. `1d` deletes the first line. `$d` deletes the last. A range that never occurs prints the file unchanged and exits 0.

```sh
sed '1,10s/pattern/pattern/' -- file
```

Line numbers count from 1, across the whole input, unless you pass `-s`. With several files and no `-s`, `$` is the last line of the last file, and line 1 is the first line of the first file. People write `sed 1,10` with no command and get a script error. The address only selects. The command is still required. `10q` prints through line 10 and stops reading. That is the early exit, not a range.

## Substitute in a range, or only the Nth match
Also asked as: sed nth occurrence; s///2; replace second match; first match on a line number; occurrence flag
A number after the closing delimiter picks which match on the line to replace. `s/pattern/pattern/2` replaces the second match and leaves the first. It is not a line number. The line number goes in front of the command. `g` replaces all of them. You cannot combine a specific occurrence with `g` in a useful way. The occurrence form is the one that means "the second one on this line."

```sh
sed 's/pattern/pattern/2' -- file
```

People write `s/pattern/pattern/2` expecting line 2. Line 2 is `2s/pattern/pattern/`. A line with only one match is unchanged when you asked for the second. No error is printed. The occurrence counts non-overlapping matches left to right. It restarts on the next line. It does not count across the file.

## Keep part of the match
Also asked as: sed backreference; sed \1; capture group; ampersand whole match; keep the matched text
In the replacement, `&` is the whole match. `\1` is the first group. In a basic expression the group is `\(` `\)`. In an extended expression it is `(` `)`. The backslash is still how you write the group in the replacement, in both dialects. A literal ampersand in the replacement is `\&`.

```sh
sed -E 's/(pattern)/x\1x/' -- file
```

People write `$1` because that is what the shell uses. In the replacement it is `\1`, and the script must be single-quoted so the shell does not eat the backslash. Groups are numbered from the left parenthesis. A group that did not participate is empty. `&` includes text the group also captured. You do not need both unless you want them.

## Insert or append a line
Also asked as: sed i insert; sed a append; add a line; insert before a match; sed append after
`i` inserts text before the selected line. `a` appends text after it. In GNU sed the text may follow the command. A backslash-newline is the portable way to separate the command from the text. The inserted text is not scanned for the next command. Without an address, the insert happens for every line.

```sh
sed '/pattern/a pattern' -- file
```

People use `s` to add a line and instead change the matching line. `a` and `i` add a new line. They do not replace. BSD sed is stricter about the backslash form. A script that works on GNU sed with `a text` can fail on macOS unless the text is on the next line. `i` and `a` write to standard output with the rest of the stream. They do not edit the file unless `-i` is also there.

## Change a whole line
Also asked as: sed c; change line; replace entire line; sed c command; swap a matching line
`c` replaces the selected line with the text you give. The rest of the line is discarded, not reused. Without an address it replaces every line, which is rarely what you want. With an address it replaces each matching line. The replacement is literal text, not a substitution of groups.

```sh
sed '/pattern/c pattern' -- file
```

People reach for `c` when `s` would keep the unmatched part of the line. `s/^.*$/pattern/` is the substitution form of "replace the whole line," and it can contain `&`. `c` cannot. As with `a` and `i`, the portable spelling puts the text after a backslash-newline. GNU sed accepts the one-line form. A range given to `c` replaces the whole range with one copy of the text, not one copy per line.

## Run more than one edit
Also asked as: sed -e; several expressions; multiple sed commands; chain substitutions; sed semicolon
`-e` adds a command. You can repeat it. Commands run in order on each line. A semicolon separates commands in one script. `-f` reads the script from a file, one command sequence per line, which is the way out of a quoting mess. Later commands see the line as earlier commands left it.

```sh
sed -e 's/pattern/pattern/' -e '/pattern/d' -- file
```

People pipe `sed` into `sed` and pay for two processes and two copies of every line. One process with two `-e` arguments is the same edit. A `d` in an earlier command skips the later commands for that line. Order matters. A substitution that creates the text a later command deletes will delete it. `-e` is also how you keep a script that starts with a dash from being read as a flag.

## Read the script from a file
Also asked as: sed -f; sed script file; commands in a file; sed -f script; avoid quoting
`-f` reads the sed script from a file. The file is not the input. The input is still the file operands, or standard input. One command per line is the readable layout. You do not single-quote the commands inside that file. The shell never sees them. `-f` can be combined with `-e`.

```sh
sed -f file -- file
```

The two `file` arguments are the bug people ship. The first, attached to `-f`, is the script. The second is the data. `sed -f file` with only one name reads the script from `file` and the data from standard input, then waits if stdin is a terminal. Comments in a script file start with `#`. A comment is not available in the middle of a command in a portable script.

## Follow a symlink while editing in place
Also asked as: sed --follow-symlinks; sed -i symlink; replace symlink with a file; in place on a link
GNU `sed -i` on a symlink can replace the symlink with a new regular file that has the output, leaving the original target unchanged. `--follow-symlinks` makes `-i` edit the target and leave the link in place. The flag is GNU, and it only matters with `-i`. It does not make a normal stdout run follow links in a special way. Opening the path already follows links.

```sh
sed -i --follow-symlinks 's/pattern/pattern/' -- file
```

People edit what they think is a config file and create a new regular file where a symlink used to be. The package's file, the target, did not change. The next update restores the link and the edit vanishes. Use the flag when the path must stay a symlink. A backup suffix still names the backup next to the path you gave, which is another surprise on a symlink. Check the path with `ls -l` after the first try.

## Separate files instead of one stream
Also asked as: sed -s; separate files; line numbers restart; sed per file; GNU sed -s
By default several named files are one stream. Line 1 is the first line of the first file, and `$` is the last line of the last file. `-s` restarts line numbers and `$` for each file. It does not start a new output file. It does not imply `-i`. It is GNU.

```sh
sed -s '1d' -- file file
```

People `sed -i '1d'` on many files and only the first file loses its first line, because the files were one stream. `-s` is the fix when the address should apply per file. `-i` still writes each input file. Without `-s`, a `1d` in a multi-file in-place run deletes one line total, not one line per file. Addresses are stream addresses unless you separate the files.

## Use a different delimiter
Also asked as: sed delimiter; s###; replace a path; slash in the pattern; sed s| |
The character after `s` is the delimiter. It does not have to be `/`. `s#/old/#/new/#` replaces a path without a row of escaped slashes. The same character ends the pattern and the replacement. A `#` inside the pattern must still be escaped if `#` is the delimiter. The flag, such as `g`, comes after the closing delimiter.

```sh
sed 's#pattern#pattern#g' -- file
```

People escape every slash and then miss one, and the script becomes "unterminated substitute." Changing the delimiter is the fix. The delimiter is not a quote. The shell still needs the script quoted. A backslash in a path is a backslash escape to `sed` as well as to the shell. Single quotes let you write the backslashes `sed` requires, and no more.

## Split on NUL records
Also asked as: sed -z; null separated; sed filenames; newline in the record; GNU sed -z
`-z` makes the record separator a NUL byte instead of a newline. A newline inside a record is ordinary text. The output records are also NUL-separated. This is GNU. macOS sed and BusyBox sed do not have it. It pairs with `find -print0` and other NUL streams.

```sh
sed -z 's/pattern/pattern/' -- file
```

Without `-z`, a newline ends the record, so a substitution cannot see both sides of it and a filename that contains a newline is two records. `-z` does not imply `-n` or `-i`. A missing final NUL is still a record, the way a missing final newline is still a line. Do not mix `-z` output with a line-oriented tool. The NULs are the separators.

## What a failed sed means
Also asked as: sed exit status; sed exit code; unterminated s; sed failed; no match exit status
An illegal script, such as an unterminated `s`, exits non-zero and prints the error on stderr. A missing file exits non-zero. A script that matches nothing exits 0 and, without `-n`, prints every input line unchanged. `sed` does not have a "whether it substituted" status. The output is the only report.

```sh
sed 's/pattern/pattern/' -- file
```

People test `$?` to see if the pattern occurred. It does not say that. `grep -q` is the test for a match. `sed` is the editor of the stream. Under `set -e`, a bad script aborts, and a clean script that changed nothing does not. `-n` with no `p` is a successful empty result, which is easy to misread as a broken pattern. Print one line, or drop `-n`, before trusting an empty pipe.
