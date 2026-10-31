# less

`less` shows a file or a pipe one screen at a time. It does not edit the file. It does not run the command that produced a pipe. You can scroll backward through input it has already read, which is the point of it over `more`. With no file it reads standard input and waits. less 590, the usual pager on Linux, is assumed here. macOS ships an older less with the same common keys. BusyBox less is thinner and lacks some flags. Exit status is 0 when you quit normally. It is non-zero when a named file cannot be read. A search that finds nothing is not an error. You are still in `less`.

Basic form: less -- file

Operands are files. A name that starts with `-` is a flag unless it comes after `--`. Flags start with `-` and can be stacked. `+` followed by a command runs that command at start, so `+/pattern` opens at the first match. Quote a pattern. The display is the terminal. A pipe into `less` has no file name, and `less` cannot go back to a producer that has exited except through what it buffered.

## Page through a file
Also asked as: less a file; open in less; pager; read a long file; less instead of cat
`less -- file` opens the file and shows the first screen. Space or `f` goes forward a screen. `b` goes back a screen. `j` and `k` move a line, or the arrow keys. `g` goes to the start. `G` goes to the end. `q` quits. `h` is the help screen, and `q` leaves that too. The file is not changed.

```sh
less -- file
```

People press `q` and expect the file to close on disk. Only the pager exits. People also press Enter expecting a line, and Enter moves one line down. `j` is the same move. A file shorter than the screen still waits, unless you passed `-F`. You are looking at a copy of the bytes in the terminal. Writing the file from another program does not move your view until you reopen or follow.

## Quit
Also asked as: less quit; q in less; how to exit less; :q; ZZ
`q` quits. `:q` quits. `ZZ` quits. Control-C quits a search or a follow. It does not always quit `less` itself if `less` is waiting at the prompt. There is no save step. Nothing was edited. If `less` is the pager for another command, quitting returns you to the shell.

```sh
less -- file
```

People reach for `:q!` because that is vim. `q` is enough. A `!` in `less` is a shell escape, not a force flag. People also close the terminal. That works and is ruder. `q` is the clean exit. Status is 0. A stuck `less` at the end of a short file is still `less`. `-F` would have exited on its own. You are not in a shell until you quit.

## Search forward and backward
Also asked as: less search; slash search; /pattern; ?pattern; find in less; n next match
`/` searches forward. `?` searches backward. Enter runs the search. `n` repeats it in the same direction. `N` repeats it in the other direction. The pattern is a regular expression. A dot matches any character. `*` repeats. The match is highlighted. `&pattern` is a different command. It filters the view to matching lines. `/` moves. `&` hides.

```sh
less -- file
```

Type `/pattern` at the prompt inside `less`, not on the command line. People type `/pattern` in the shell and the shell tries to open a file or start a search of its own. A pattern with a slash does not need escaping inside `/`. A pattern with a special regex character does. `less` uses its built-in regex. A search that finds nothing rings the bell and stays put. It is not a failed process. `-i` makes the search ignore case unless the pattern has an uppercase letter. `-I` ignores case always.

## Jump to a line, or to the end
Also asked as: less go to line; g and G; less line number; jump to end; 50g
A number and `g` jumps to that line. `50g` is line 50. `g` alone is the start. `G` is the end. `:n` is another way to ask for a line number. `less` counts lines from 1. A number past the end goes to the end. For a pipe, the end is however far the producer has got, until you have read it all.

```sh
less -- file
```

People type `:50` because that is vim. In `less` the count comes first, then `g`. People also `G` on a pipe that is still running and `less` waits while it reads to the current end. A very large file is fine. `less` does not load it all at once for a normal forward read. Jumping to the end of a huge file does read through it. `-N` shows line numbers in the margin so the jump has a landmark.

## Show line numbers
Also asked as: less -N; less line numbers; --LINE-NUMBERS; number the lines; less -N file
`-N` prints a line number in front of each line. The number is the line in the file, not the screen row. Wrapped lines share the file line's number. `-N` does not change search. It does not change `g`. It costs a little width. The flag is on the command line, or you can type `-N` inside `less` to toggle it.

```sh
less -N -- file
```

People expect `-n` to number lines. `-n` suppresses the line-number calculation in the prompt, which is a different flag and a speed hack on a slow file. `-N` is the visible numbers. Inside `less`, typing `-N` and Enter toggles the option for this session. It does not rewrite the file. A pipe's line numbers count what has been read. They are stable once read.

## Follow a growing file
Also asked as: less +F; less follow; like tail -f; Shift-F; watch a log; less growing file
`+F` starts in follow mode. `less` jumps to the end and reads new bytes as they arrive, like `tail -f`. Shift-F does the same from inside `less`. Control-C leaves follow mode and returns to normal paging, at the end. `F` again resumes. Follow works on a file. On a pipe it reads until the producer closes, then it has nothing more to follow.

```sh
less +F -- file
```

People `+F` a file that is rotated. `less` keeps the inode it opened. A new file with the same name is invisible until you reopen. `tail -F` follows the name. `less` does not. People also expect to search while following. Leave follow with Control-C, search, then `F` again. New lines are highlighted only in the sense that they appear. Follow is not a filter.

## Keep color from a command
Also asked as: less -R; less colors; raw control chars; git diff less; ANSI color; ls --color | less
`-R` lets through ANSI color sequences and otherwise shows control characters safely. A command that prints color, piped to `less`, looks colored. Without `-R`, you see the escape bytes or nothing useful. `-r` is the older, less safe form that passes controls through raw. Prefer `-R`. `LESS=-R` makes it the default for every `less`.

```sh
command -- file | less -R
```

People pipe `ls --color` or a test runner into `less` and the color disappears. The producer saw a pipe and may also have disabled color itself. Force color on the producer if it has a flag, and pass `-R` to `less`. `-R` does not invent color. It only preserves sequences that are already in the stream. A binary file with `-R` can still look like noise. `less` is a text pager.

## Chop long lines instead of wrapping
Also asked as: less -S; chop long lines; horizontal scroll; do not wrap; less --chop-long-lines
`-S` cuts lines at the screen edge instead of wrapping them. You scroll right with the right arrow, or left with the left arrow. Wrapped lines are the default, and a long JSON line becomes many screen rows. `-S` keeps one file line on one screen line. Inside `less`, `-S` and Enter toggles it.

```sh
less -S -- file
```

People lose the end of a line and think the file is short. It is chopped. Scroll right. People also search and the match is off screen to the right. `less` moves to the line. It may not scroll horizontally to the match. The right arrow is the rest of the way. `-S` does not change the file. A copy you redirect is still the full line.

## Quit if the file fits on one screen
Also asked as: less -F; quit if one screen; less cat short files; --quit-if-one-screen; less -FX
`-F` exits immediately if the file fits on one screen. You never see the prompt. It is the flag that makes `less` behave like `cat` for short input and like a pager for long input. Combined with `-X`, it avoids clearing the screen, so short output stays in the scrollback. `LESS=-FX` is a common default.

```sh
less -FX -- file
```

People pass `-F` and a short file seems to do nothing, because `less` printed it and exited. That is the flag working. People also want `-F` on a pipe. `less` has to read the whole pipe to know it is short, so the producer runs to completion. A producer that never closes makes `-F` wait. `-F` does not follow. `+F` does. They are different flags.

## Start at a pattern
Also asked as: less +/pattern; open at a match; less -p; start at search; jump to pattern on open
`+/pattern` runs a forward search as soon as the file opens, so the first screen is the first match. `-p pattern` is the same idea as a flag. The pattern is a regex. Quote it so the shell does not eat it. `+` commands run on each file in a multi-file list. This is the way to open a log at an error string.

```sh
less +/pattern -- file
```

People put the pattern after the filename and `less` treats it as a second file. The `+` command belongs with the options, before the file. A pattern that is not found opens at the top and the bell rings. It is not a usage error. A slash in the pattern is fine inside the quotes. A single quote in the pattern needs a different quoting. The search is the same one `/` would run.

## Ignore case in a search
Also asked as: less -i; less -I; case insensitive search; ignore case; smart case less
`-i` ignores case in searches unless the pattern contains an uppercase letter. That is the usual "smart case." `-I` ignores case always. Both apply to `/`, `?`, and `+/`. Inside `less` you can toggle `-i` by typing it. A pattern of `[A-Z]` is still a character class. The flag does not rewrite your regex.

```sh
less -i -- file
```

People expect the search to ignore case by default. It does not. `Foo` does not match `foo` unless `-i` or `-I` is on. With `-i`, a pattern of `Foo` is case sensitive, because of the uppercase letter. Type `foo` if you wanted the insensitive search. `-I` is the blunt one. Neither flag changes `&` filtering in a surprising extra way. The filter uses the same pattern rules.

## Filter the view to matching lines
Also asked as: less filter; ampersand pattern; &pattern; only matching lines; less grep
`&pattern` shows only lines that match. It is a display filter. The file is unchanged. `&` alone, or `&` with an empty pattern, clears the filter. The pattern is a regex, with the same case rules as `/`. You can still scroll. You are scrolling the matches. This is the interactive form of a grep you can walk.

```sh
less -- file
```

Type `&pattern` at the `less` prompt. People prefix it on the command line and the shell backgrounds the command. Inside `less`, `&` is the filter. A filter that matches nothing shows an empty file until you clear it. `n` still searches. The filter is not a search highlight. `/` highlights and moves. `&` hides non-matches. Clear it before you decide the file is short.

## Open several files
Also asked as: less multiple files; :n next file; less file file; switch files; :p previous
`less -- file file` opens a list. `:n` goes to the next file. `:p` goes to the previous. `:x` is the first. The prompt shows the file name when you have more than one, or with `-M`. A search does not cross files unless you started it with a form that says so. Each file starts at the top, or at a `+` command.

```sh
less -- file file
```

People expect a single scroll across all files. The list is separate files. `:n` is the step. People also pass a glob that matches nothing. The shell leaves the literal, and `less` reports no such file. A directory operand is an error. `less` does not list a directory. Use a glob of files. `:d` removes the current file from the list. It does not delete it on disk.

## See the file name and the position
Also asked as: less -M; long prompt; where am I in the file; less status line; percent through file
`-M` uses a long prompt. It shows the file name, the line number, and the percentage through the file. `-m` is the shorter form. The default prompt is easy to miss. `-M` is the one that answers "where am I." Inside `less`, `-M` toggles it. The percentage is of bytes, and it can sit still across many short lines.

```sh
less -M -- file
```

People read the percentage as a line percentage. It is of the file size. A few huge lines move it a lot. People also want the prompt on a pipe. Until the producer finishes, `less` does not know the length, so the percentage is a question mark. The line number still advances. `-N` is the margin numbers. `-M` is the prompt. They are independent.

## Squeeze blank lines
Also asked as: less -s; squeeze blank lines; collapse empty lines; less multiple blanks; --squeeze-blank-lines
`-s` shows a run of blank lines as one blank line. The file is unchanged. A log with page-long gaps becomes readable. A form feed is a different character. `-s` is about empty lines. Inside `less` it can be toggled. Copying from the display copies what you see. A redirect of the file still has the blank lines.

```sh
less -s -- file
```

People pass `-s` to delete blank lines from a file they are editing. `less` does not edit. The next open without `-s` shows the gaps again. People also expect `-s` to trim trailing spaces. It does not. A line of spaces is not blank. A blank line is empty. `-s` does not change search. A search for a line in a squeezed gap still finds the real line.

## Do not clear the screen on exit
Also asked as: less -X; no init; leave output on screen; less clear screen; --no-init
`-X` skips the terminal init and deinit strings. On exit, the last screen stays in the scrollback instead of clearing. Combined with `-F`, short output behaves like `cat` and remains visible. Without `-X`, some terminals clear the pager's screen as `less` exits, and the text you just read is gone.

```sh
less -X -- file
```

People quit `less` and the page vanishes. `-X` is the usual fix. People also set `LESS=-X` and then a full-screen `less` misbehaves on a terminal that needed the init string. The trade is real. `-X` is right for short command output. A long editing session in `less` can prefer the default. The flag does not change keys or search.

## Page a pipeline
Also asked as: command | less; pipe to less; less stdin; pager for a command; no file
`command | less` pages that command's standard output. There is no file name. You can scroll back through what `less` has read. You cannot scroll back through a producer that `less` has not read yet, and you cannot restart the producer from inside `less`. `q` stops `less` and the producer gets a broken pipe if it is still writing.

```sh
command -- file | less
```

People pipe a command that asks questions. The command's stdin is the pipe's stdin, not your keyboard, unless you arranged that. `less` has the keyboard. A producer that detects a pipe and turns color off needs its color forced, plus `less -R`. A producer that never exits makes `G` and `-F` wait. Control-C during follow, or `q`, is the way out. The producer's status is lost unless the shell has `pipefail`.

## Search only matching lines, from the command line
Also asked as: less and grep; grep | less; page matches; less a filtered file; pattern then page
`less` does not take a search flag that filters, except the `+/pattern` jump. To see only matches, filter first. `grep -- pattern file | less` is the pipeline. `grep -n` keeps the line numbers. Inside `less`, `&pattern` is the interactive version of the same idea, on a file you already opened. The pipeline is the one to use from a script or a command line.

```sh
grep -n -- 'pattern' file | less
```

People write `less /pattern file` and `less` looks for a file named `/pattern`. The slash search is an inside command. People also `grep` a binary and `less` shows noise. `grep -a` or a text file is the fix. `-R` still helps if the match lines have color from `grep --color`. `grep --color=always` forces the color into the pipe. Without `always`, grep turns color off when it sees a pipe.

## Use it as the default pager
Also asked as: PAGER=less; LESS variable; man less; default pager; git pager
`PAGER` is the program other tools run when they have more than a screen of output. `export PAGER=less` makes that `less`. `LESS` holds default flags, such as `-R -F -X`. `man` uses a pager. `git` uses one for diffs. Setting `LESS` once is quieter than passing flags on every command.

```sh
export LESS='-R -F -X'
```

People export `PAGER='less -R'` and a tool that appends its own flags can break the parse. Put flags in `LESS`. Put the program name in `PAGER`. People also set `-F` globally and a short `man` page flashes and exits. That is `-F`. Drop it if you want every manual to wait. `LESS` is read by `less`, not by `more`. A system whose `PAGER` is `more` never sees it.

## Open a file whose name starts with a dash
Also asked as: less -- -file; less dash file; file named -N; double dash less; operand
A file named `-N` looks like the line-number flag. `less` then waits on stdin, or errors, and the file is unopened. `--` ends options. The next word is a file. `./-N` is the other form. This is the same rule as the rest of the core utilities.

```sh
less -- -file
```

People `less -file` and get a usage complaint or a session that ignores the file. The dash was a flag. `--` is the fix. A `+` in a filename can also be read as a command if it sits where a `+` command belongs. Put file names after `--` when they are unusual. A directory is still an error after `--`. `less` will not list it.

## What a failed less means
Also asked as: less exit status; less failed; no such file less; less error; cannot open
A missing file exits non-zero after a message. A directory exits non-zero. A search that finds nothing does not exit at all. You are still in `less`. A pipe whose producer fails still lets `less` show whatever arrived, and the pipeline status is the last command unless `pipefail` is set. `q` is status 0.

```sh
less -- file
```

People treat an empty `less` as a broken file. An empty file opens and shows nothing. That is a successful open. People also background `less` and it errors because it has no terminal. The pager needs a tty. Redirecting `less` itself to a file is the wrong shape. `less` is the viewer. The file or the pipe is the input. Check the producer's status separately if the page was short and you expected more.
