# grep

`grep` prints every line that matches a pattern. Give it files, or give it no file and it reads standard input. It does not list files by name. It looks inside them, and only if you ask with `-l` does it answer "which files" instead of "which lines".

Basic form: grep [options] PATTERN [FILE...]

The pattern is a basic regular expression, not a fixed sentence, unless you pass `-F`. In that default dialect `.` matches any one character, `*` repeats the previous item, and `^` and `$` anchor the ends of a line. A plus, a question mark, a vertical bar, and parentheses are ordinary characters until you escape them or switch dialect with `-E`. Most "the pattern did nothing" surprises are that rule, or the shell having already eaten the pattern before `grep` saw it.

Quote patterns in single quotes. Double quotes still expand `$`, backticks, and sometimes `!`. A pattern that starts with `-` looks like an option; put `--` before the pattern so `grep` stops reading options.

Exit status is 0 when some line matched, 1 when none did, and 2 when a file could not be read or the invocation itself was wrong. Scripts should trust that status. `grep -q` asks the question and prints nothing.

No file means "read standard input", and `grep` will sit there until you type, or until the pipe in front of it produces lines. A directory given without `-r` is not searched. GNU `grep` reports `Is a directory` and exits 2.

These recipes assume GNU grep, which is the `grep` on Linux. macOS ships a different grep. Everyday flags (`-n`, `-i`, `-r`, `-v`, `-F`, `-E`, `-A`, `-B`, `-C`) exist on both. A recipe that needs a GNU-only flag says so.

Each question below stands on its own. The line under the question uses the other wordings people reach for, so a search can land here even when it is not phrased like the heading.

## Find all files containing a specific text

Also asked as: find all files containing a specific string on Linux; which files contain this text; filenames only; search a tree for a word; grep -l; grep -r files containing

`-l` turns the answer from lines into paths. `-r` is what walks into directories. Without `-l` you get every matching line, which is a different question.

```sh
grep -r -l -- 'some text' .
```

The path at the end matters. `.` means "start here". Hidden files and hidden directories are included. So is `.git`, if you have one. `--exclude-dir` is the usual way to drop those, and it has its own question below.

`-l` prints each path once, even if the file matches on many lines. The pattern is still a basic regular expression, so a dot in `file.txt` matches any character. Add `-F` when the text is literal.

```sh
grep -r -l -F -- 'file.txt' .
```

This does not find files by name. `find . -name '*.txt'` is that other job.

## Show the matching lines themselves

Also asked as: print the lines that contain a string; not just the filename; show me the hit; grep a file for a word; where does this text occur

The default already prints matching lines. `-n` adds the line number, which you almost always want when the file is long. `-H` forces `filename:` onto each line. GNU grep already does that when you name more than one file, and omits it when you name exactly one.

```sh
grep -n -H -- 'some text' file
```

Across a tree, drop `-l` and keep `-r`:

```sh
grep -r -n -- 'some text' .
```

Each hit is `path:line:text`. A colon inside a filename is ambiguous in that format. It is rare, and `-z` is the escape hatch, not the everyday form.

## Find all lines that do not contain a word

Also asked as: lines that do not contain a word; exclude lines; invert match; filter out; grep -v; lines without this word; opposite of grep

`-v` keeps the lines that failed the pattern. The rest of the flags still describe what a "match" would have been. `-w` means a whole word, not a stretch of characters in the middle of a larger one.

```sh
grep -v -w -- 'word' file
```

`-w` is the difference between "the word pass" and "the letters pass". Without `-w`, `-v -F -- 'pass'` also throws away `password` and `compass`. With `-w`, those lines stay, because the letters are not sitting between non-word characters. Word characters are letters, digits, and underscore. A hyphen is not one, so `pass` still matches inside `pass-word`.

To drop lines that contain the letters anywhere, even inside a longer word, use a fixed string and no `-w`:

```sh
grep -v -F -- 'word' file
```

The whole line goes. `grep` cannot delete just the word and print the rest. That is `sed`, or cut the column some other way.

## Show lines surrounding each match

Also asked as: show lines surrounding each match; context; lines around a match; a few lines before and after; neighboring lines; grep -C; grep -A -B; window around the hit

`-C 2` prints two lines before the match, the match itself, and two lines after. The match is inside the window, not extra to it.

```sh
grep -n -C 2 -- 'pattern' file
```

`-B` is only the lines before. `-A` is only the lines after. `-C` is both, and `-C 2` is the same request as `-A 2 -B 2`.

```sh
grep -n -A 5 -- 'pattern' file
```

Separate groups of context are split by a line that contains only `--`. That marker is not part of the file. `-n` numbers the context lines as well as the match, so you can see how far from the hit you are. Context does not wrap from the end of the file back to the start.

## Recursively grep all directories and subdirectories

Also asked as: how to recursively grep all directories and subdirectories; search every folder under this one; grep a whole tree; grep -r; descend into subdirs; recursive search

`-r` reads every file under each directory you name. It does not follow a symbolic link that points at a directory. `-R` does follow those links, which can visit the same file twice and can loop. Use `-r` unless you mean the links.

```sh
grep -r -n -- 'pattern' .
```

The `.` is not decoration. With no file at all, `grep` reads standard input even if `-r` is set, and it looks hung. `-r` applies to directory arguments. It does not mean "guess that I meant the current directory".

Hidden directories are searched. Binary files are not printed as text; grep says `Binary file … matches` and moves on. Both of those have their own questions.

## Search only files with a certain name

Also asked as: only python files; only a filename pattern; grep *.log; certain extensions; --include; skip everything except these names

`--include` keeps files whose base name matches a glob. The directory walk still enters other directories. Quote the glob, or the shell expands it to whatever happens to be in the current directory before `grep` starts.

```sh
grep -r -n --include='*.py' -- 'pattern' .
```

Repeat the flag to allow more than one shape. They combine as "any of these names".

```sh
grep -r -n --include='*.py' --include='*.pyi' -- 'pattern' .
```

The glob is only the file's own name, not its path. `*.py` matches `src/app.py`. It does not match a file simply because a parent directory contains `py`.

## Skip .git, node_modules, and other directories

Also asked as: exclude a directory; exclude-dir; ignore vendor; skip node_modules; don't search .git; --exclude-dir; grep is too slow on a repo; leave out a folder

`--exclude-dir` matches the directory's own name, anywhere under the start path. A directory named `.git` is skipped no matter how deep it sits.

```sh
grep -r -n --exclude-dir='.git' --exclude-dir='node_modules' -- 'pattern' .
```

Do not write `--exclude-dir={.git,node_modules}`. Bash expands the braces into two arguments before `grep` sees them, and the result is not what you meant. Repeat the flag.

`--exclude` is the same idea for file names rather than directories, for example `--exclude='*.min.js'`. It is also a base-name glob, and it also wants quotes.

## Search for a fixed string, not a regular expression

Also asked as: literal string; not a regex; exact text; the dot should be a dot; grep -F; fgrep; search for a sentence; characters are special and I do not want that

`-F` makes every character in the pattern ordinary. A dot is a dot, a star is a star, a bracket is a bracket.

```sh
grep -F -n -- 'config.json' file
```

Use this whenever the thing you have is data, not a pattern: a filename, an error message, a URL, a sentence. You can combine it with anything else. `-r -l -F` is "files that contain this literal text".

`fgrep` is the old name for the same mode. Write `-F`.

The pattern is still matched as a substring. `-F` does not mean "the whole line". That is `-x`, and it is the next question but one.

## Search without caring about case

Also asked as: case insensitive; ignore case; grep -i; ERROR and error; capital letters do not matter

`-i` folds case using the current locale. In a typical UTF-8 locale, `i` and `I` match, and the same for letters that have a case pair.

```sh
grep -i -n -- 'error' log
```

It does not mean "close enough". `error` does not match `err`. Combine it with `-F` when the rest of the text is literal, and with `-w` when you still want a whole word.

## Match a whole word, not a substring

Also asked as: whole word only; not part of another word; word boundary; grep -w; match port but not portrait; exact word

`-w` requires a non-word character (or the edge of the line) on both sides of the match. Word characters are letters, digits, and underscore.

```sh
grep -w -n -- 'port' file
```

`port` matches `port 80` and does not match `portrait`. It does match `pass` inside `pass-word` if you searched for `pass`, because the hyphen is not a word character. If that hyphen should glue the word together, `-w` is the wrong tool and you want a tighter pattern, or `-x` if the whole line is the word.

## Match the whole line and nothing else

Also asked as: whole line; exact line; line is exactly this; no extra text on the line; grep -x; full line match

`-x` anchors the pattern to the entire line. Nothing may sit before or after it. `-F` keeps the text literal, which is what you want for a status word or a fixed record.

```sh
grep -x -F -- 'OK' file
```

Without `-F`, the pattern is still a regular expression, only now it has to cover the line. `grep -x -- 'a.*'` matches a line that is `a` followed by anything, and it matches nothing else.

## Show line numbers and the file name

Also asked as: line numbers; print the filename; file and line; grep -n; grep -H; where in the file; prefix every hit

`-n` puts the line number after the file name. `-H` prints the file name even when you only named one file. With two or more files, GNU grep prints the name whether or not you passed `-H`.

```sh
grep -n -H -- 'pattern' file
```

`-h` does the opposite: never print the file name. That is what you want when several files are just one stream of lines to you, for example a glob of logs you are about to cut apart.

```sh
grep -h -n -- 'pattern' *.log
```

The shell expands `*.log`. If nothing matches the glob, bash passes the literal `*.log` unless `nullglob` is set, and grep then looks for a file with that name.

## Count matching lines, or count matches

Also asked as: how many lines; count occurrences; count occurrences not lines; matches versus lines; how many times; grep -c; number of hits; wc

`-c` counts lines, not hits. A line that contains the pattern three times counts as one. Every input file is reported, including a file whose count is zero.

```sh
grep -c -- 'pattern' file
```

With more than one file the lines look like `file:3`. The exit status is still 0 if any file had a match and 1 if none did. The zero on a particular file is not an error.

To count occurrences, including several on one line, print only the matches and count those lines:

```sh
grep -o -F -- 'pattern' file | wc -l
```

`-o` is doing the real work. `wc -l` only counts the lines `-o` produced. A binary file can still short-circuit this; see the binary-file question.

## Print only the part of the line that matched

Also asked as: only the match; extract the text; not the whole line; grep -o; pull out the numbers; show just the hit

`-o` prints each match on its own line and throws the rest of the line away. Two matches on one line become two output lines.

```sh
grep -o -E -- '[0-9]+' file
```

`[0-9]+` needs `-E`. In the default dialect `+` is a plus sign, not "one or more". That dialect trap has its own question. Add `-n` if you still need to know which line the fragment came from. The number is the line in the original file, and it repeats when one line produced several fragments.

## Search for any of several patterns

Also asked as: this or that; either pattern; multiple patterns; OR; grep -e; egrep; one of these words; several strings

Repeat `-e` once per pattern. Each pattern is still a basic regular expression. Add `-F` when they are all literal text.

```sh
grep -n -e 'warn' -e 'error' -- log
```

```sh
grep -F -n -e 'warn' -e 'error' -- log
```

A single pattern with a vertical bar is the other shape, and the bar is only special under `-E`:

```sh
grep -E -n -- 'warn|error' log
```

In the default dialect you would have to write `warn\|error`. Prefer `-E` or repeated `-e`. Don't mix them up: `|` inside a `-F` pattern is a literal bar character, so it will not mean "or".

## Keep lines that contain all of these words

Also asked as: AND; both words; both words on a line; line contains two strings; all of these; not or; must have both; grep and grep

`grep` has no AND operator. A line contains both words when it survives two filters in a row. The second `grep` reads the lines the first one printed.

```sh
grep -w -F -- 'timeout' log | grep -w -F -- 'redis'
```

Put `-i` on both if case does not matter, and `-w` on both if you mean words. The order of the two filters does not change which lines survive. This is "both somewhere on the line", not "this phrase". A phrase with a space is one fixed pattern: `grep -F -- 'connection reset' log`.

## Read the patterns from a file

Also asked as: patterns from a file; many patterns in a list; grep -f; a file of needles; search for each line in this list

`-f` reads patterns from a file, one per line. `-F` makes those lines literal. You almost always want that, because the file is data someone listed, not a file of regular expressions.

```sh
grep -F -f patterns.txt -- log
```

A blank line in `patterns.txt` is an empty pattern, and an empty pattern matches every line. The search then appears to return the whole log. Delete blank lines. A line in the pattern file is not a shell command and is not quoted; what you see in the file is the pattern, including any spaces.

## Test whether a match exists, in a script

Also asked as: if grep; quiet; exit code; did it match; grep -q; test in a shell script; boolean; found or not

`-q` prints nothing and stops at the first match. The `if` looks at the exit status: 0 means something matched, 1 means nothing did.

```sh
if grep -q -F -- 'ready' status.txt; then
  echo found
fi
```

A missing file is exit status 2, and `if` treats that as false as well. "No match" and "no such file" are the same branch unless you distinguish them. Check that the file exists first when the difference matters, or inspect `$?` without an `if`.

`set -e` does not abort the script on the failing `grep` inside `if`. That is the normal, safe form. Prefer `-q` over redirecting stdout to `/dev/null`, which still scans to the end and still prints errors.

## Use a pattern that starts with a dash

Also asked as: pattern starts with a hyphen; starts with -; option looking pattern; grep -- ; argument that looks like a flag; -v as text

`grep` reads anything that starts with `-` as an option, until it sees `--` or a non-option. A pattern of `-failed` is understood as flags, not as text, and the command errors or searches for the wrong thing.

```sh
grep -n -- '-failed' file
```

`--` ends option parsing. The next argument is the pattern even though it starts with a dash. Quoting does not do this job. Quotes hide the pattern from the shell, not from `grep`'s own option parser.

`-e` is the other way to mark "this argument is a pattern":

```sh
grep -n -e '-failed' -- file
```

Use `--` as the habit. It is the same spelling you want in front of ordinary patterns too.

## Match a literal dot, star, or bracket

Also asked as: literal dot; escape special characters; star is not a star; brackets; file.txt matches fileXtxt; regex metacharacters; quote a dot

In the default dialect `.` matches any character, so `file.txt` matches `file.txt` and also `fileXtxt`. When the whole pattern is literal, `-F` is the right fix, not a row of backslashes.

```sh
grep -F -n -- 'file.txt' list
```

When part of the pattern really is a regular expression and one character must be literal, escape that character and do not use `-F`:

```sh
grep -n -- 'file\.txt$' list
```

The backslash is inside single quotes so the shell passes it through. `\.` is "a dot". `$` is "end of the line", which is why this one is a pattern and not a `-F` string.

## Why a+ does not mean one or more

Also asked as: plus does not repeat; a+ matches nothing; plus not working; regex plus not working; why does + not match; basic regex; extended regex; grep -E; egrep; quantifier; question mark and bar and parentheses are literal

GNU grep's default dialect is basic regular expressions. `+`, `?`, `|`, `{`, `}`, `(`, and `)` are ordinary characters there. The pattern `a+` matches the two characters `a+`. It does not match `a` or `aaa`.

```sh
grep -E -n -- 'a+' file
```

`-E` switches to extended regular expressions, where `+` means "one or more", `?` means "optional", and `|` means "or". Parentheses group, without backslashes.

The same pattern in the default dialect escapes the plus:

```sh
grep -n -- 'a\+' file
```

Remember one of the two, not both. `-E` is the one to reach for when you are writing a pattern on purpose. `-F` is the one to reach for when you are not.

`egrep` is the old spelling of `grep -E`. Write `-E`.

## Search the output of another command

Also asked as: grep a pipe; from stdin; filter command output; no file; search ls; search logs from a program; standard input; grep hung waiting; grep waits; looks hung; waiting for input; forgot the pipe

With no file name, `grep` reads standard input. A pipe is standard input.

```sh
ls -l | grep -F -- ' -> '
```

That keeps the long-listing lines that contain the arrow `ls` uses for symbolic links. There is no file argument. If you leave the pipe out, `grep` waits for you to type, which looks like a hang. Ctrl-D is end of input.

Don't pipe `cat` into `grep` just to read a file. `grep -F -- '404' access.log` is the whole command. Use the pipe when the lines are produced by something else.

`-r` does not change the "no file means stdin" rule. `grep -r pattern` with no path reads the terminal.

## Stop a pipeline grep from matching its own process

Also asked as: grep finds itself; ps aux grep itself; ps aux | grep; the grep line shows up; match its own command; pgrep; bracket trick; process list

`ps` prints every process, including the `grep` you just started. That `grep`'s command line contains the word you searched for, so it matches.

```sh
ps aux | grep '[s]sh'
```

`[s]sh` is a pattern that matches the text `ssh`. The `grep` process is listed with the argument `[s]sh`, which does not match the pattern `[s]sh`. The real `ssh` processes do.

`pgrep` exists so you do not have to do this. `pgrep -a ssh` prints matching processes and not itself. Use that when the question is "is this process running" rather than "filter this particular text".

## Skip binary files, or search them as text

Also asked as: Binary file matches; skip binary; grep -I; grep -a; don't search binaries; force text; garbage output

GNU grep looks at the start of a file. If it decides the file is binary, it does not print the matching line. It prints `Binary file foo matches` once and continues. That message is easy to miss in a recursive search, and the line you wanted is not on the screen.

`-I` tells grep to treat a binary file as if it did not match. No message, no hit. This is the right flag for "search my source tree".

```sh
grep -r -I -l -- 'some text' .
```

`-a` does the opposite. The file is text, whatever the bytes are. You will see unreadable bytes when the match sits in a compressed or compiled file. That is what you asked for.

```sh
grep -a -n -- 'some text' file.bin
```

`-I` and `-a` are GNU behavior. If the grep on another system rejects them, search only the files you already know are text, with `--include` or with `find`.

## Stop after a few hits

Also asked as: first match only; stop early; limit output; head; grep -m; only a few lines; don't flood the terminal

`-m 5` stops a file after five matching lines. It is a per-file cap, not a cap on the whole run. In a recursive search you get up to five hits from each file.

```sh
grep -m 5 -n -- 'pattern' file
```

The exit status is 0 if it found those lines, including when it stopped early because of `-m`. It is 1 only when nothing matched.

If you want five lines from an entire tree and you do not care which file they came from, `-m` will not do that. Cut the output down afterwards:

```sh
grep -r -n -- 'pattern' . | head -n 5
```

`head` closes the pipe early. `grep` may report a broken pipe. The five lines are still the five lines.

## Follow directory symlinks, or don't

Also asked as: -r versus -R; symbolic links; symlink to a directory; dereference; grep does not enter a link; follow links

`-r` descends into real directories. A symbolic link to a directory is printed or skipped as the link itself, not walked. `-R` walks those links too.

```sh
grep -r -n -- 'pattern' .
```

```sh
grep -R -n -- 'pattern' .
```

Prefer `-r`. Following links revisits files that are linked from more than one place, and a link back to a parent walks forever. Use `-R` only when the tree is deliberately built out of links and you know it does not cycle.

A symbolic link to a regular file is a separate choice, controlled by whether the link itself is the thing you opened. For a link named on the command line, grep reads the file it points at. You do not need `-R` for that.

## A form that does not depend on GNU grep

Also asked as: portable grep; portable macOS; macOS; busybox; POSIX; no --exclude-dir; find and xargs; works everywhere; BSD grep

Simple recursion exists on current macOS too: `grep -r -n -- 'pattern' .` is fine there. Reach for `find` when you need to skip directories and you cannot depend on `--exclude-dir`, or when the grep is a small one that does not have it.

```sh
find . -type f \
  ! -path '*/.git/*' \
  ! -path '*/node_modules/*' \
  -print0 |
  xargs -0 grep -n -H -e 'pattern'
```

`-print0` and `xargs -0` split names on a null byte, so spaces and newlines in file names do not break the command. `xargs` may run `grep` several times if the list is long. A match is still a match. The exit status of the pipeline is the status of `xargs`, not the simple 0-or-1 of a single `grep`, so don't use this form inside `if` when you only need a yes or no. Use `grep -q` for that.

`-path` is matched against the path `find` is printing, with the leading `./`. The `*/.git/*` shape is what skips entries inside those directories.
