# more

`more` shows a file or a pipe one screen at a time. It does not edit the file. It is a pager that mostly moves forward. Going back is limited, and on a pipe it is not really available. `less` is the pager that scrolls backward. With no file, `more` reads standard input. util-linux `more` on Linux is assumed here. macOS `more` is a BSD pager with the same idea and different flags. BusyBox `more` is thinner. On some systems `more` is `less` in disguise. `more -V` or `more --version` says which you have. Exit status is 0 when you quit or the file ends. It is non-zero when a named file cannot be read.

Basic form: more -- file

Operands are files. A name that starts with `-` is a flag unless it comes after `--`. `+10` starts at line 10. `+/pattern` starts at the first match. Quote a pattern. Space advances a screen. Enter advances a line. `q` quits. A short file may print and exit without a prompt, which is the behavior people wanted from `cat`.

## Page through a file
Also asked as: more a file; page a file; read a long file; more instead of cat; open in more
`more -- file` shows the first screen and waits. Space or `f` is the next screen. Enter is the next line. `q` quits. A file shorter than the screen is printed and `more` exits. You never get a prompt. The file is not changed. This is the forward pager.

```sh
more -- file
```

People press `b` expecting to go back, as in `less`. util-linux `more` can go back in a file it can rewind. A pipe cannot be rewound. People also wait for a prompt on a short file and the command has already returned. That is success. Use `less` if you need to scroll both ways or keep a short file on a prompt. `more` is the simple forward read.

## Quit
Also asked as: more quit; q in more; how to exit more; leave more; stop paging
`q` quits. On util-linux `more`, `Q` quits too. There is no save. Nothing was edited. If `more` is the pager for `man` or another command, `q` returns you to the shell. A short file never waits, so there is nothing to quit.

```sh
more -- file
```

People try `:q` because that is vim and `less`. In `more`, `q` is the key. A `!` is a shell escape on some builds, not a force-quit. People also close the terminal. That works. `q` is the clean exit, and the status is 0. A `more` that seems stuck at the end of a file is waiting. `q` or Space, depending on the build, finishes it. `-e` makes end of file exit on its own.

## Move forward a line or a screen
Also asked as: more next page; space in more; enter in more; scroll more; f key
Space moves forward a screen. Enter moves forward one line. `d` moves half a screen on util-linux `more`. A number before Space moves that many lines. There is no "page up" you can rely on. The direction of this pager is forward. `less` is the one with `b` as a habit.

```sh
more -- file
```

People hold Enter and expect a smooth scroll. Each Enter is one line and a new prompt. Space is the page. People also press the arrow keys. A terminal may pass them through, and `more` may treat them as ordinary input or ignore them. Space and Enter are the keys that work. The prompt often shows a percentage. It is of the file, and it stays blank on a pipe whose length is unknown.

## Search
Also asked as: more search; /pattern in more; find in more; slash search; n next
`/` searches forward from the current position. The pattern is a regular expression. `n` repeats the last search. A match moves you to that line. A miss rings the bell or prints a message and stays. There is no backward search in the usual `more`. `less` has `?`.

```sh
more -- file
```

Type `/pattern` at the `--More--` prompt, not in the shell. People type `/pattern` on the command line and the shell looks for a file. A regex dot matches any character. Quote nothing inside the prompt. The shell is not involved there. A search on a pipe only sees what has been read and what follows. It cannot jump backward through discarded input. `+/pattern` on the command line starts at the first match, which is the non-interactive form.

## Start at a line or a match
Also asked as: more +10; more +/pattern; start at line; open at a match; skip the top
`+10` begins at line 10. `+/pattern` begins at the first line that matches. The pattern is a regex. Quote it. Line numbers count from 1. A number past the end starts at the end. This is how you skip a header you already read.

```sh
more +/pattern -- file
```

People put the pattern after the filename and `more` treats it as a second file. The `+` argument comes before the files. A pattern that is not found starts at the top and says so. It is not a usage error. `+1` is the start. A `+` glued to a filename can be read as this option. Use `--` before a strange name. `+/pattern` does not filter. It only moves the start. You still see the following lines.

## Exit at the end of the file
Also asked as: more -e; exit on eof; do not prompt at end; --exit-on-eof; more quit at end
`-e` exits when `more` reaches end of file, instead of prompting one last time. A short file already exits. `-e` matters for a file that took several screens. You press Space through it and the last screen returns you to the shell. Without `-e`, the last prompt waits for `q`.

```sh
more -e -- file
```

People pass `-e` and think it changes a short file. The short file already exited. People also want `-e` to follow a growing file. It does not. End of file is the end. A growing log needs `tail -f` or `less +F`. `-e` is a convenience on the last page. It does not skip the last page. You still see it.

## Squeeze blank lines
Also asked as: more -s; squeeze blank lines; collapse empty lines; --squeeze; multiple blanks
`-s` shows a run of blank lines as one blank line. The file is unchanged. A log with large gaps becomes shorter to page. A line of spaces is not blank. A blank line is empty. `-s` does not trim trailing spaces.

```sh
more -s -- file
```

People pass `-s` to edit the file. `more` does not write. The next open without `-s` shows the gaps. People also expect `-s` to join wrapped lines. It does not. Wrapping is the terminal width. `-s` is only empty lines. Search still sees the real file. A match in a squeezed gap is still found, and the display may show a single blank where several were.

## Set the screen size
Also asked as: more -n; lines per screen; more -20; --lines; page length
`-n 20` uses 20 lines per screen instead of the terminal height. The old form `-20` is the same number. Use this when the prompt scrolls off, or when you are capturing a session and want short pages. The number is lines. It is not a percentage.

```sh
more -n 20 -- file
```

People pass `-n` expecting line numbers. Line numbers are not this flag. `-n` is the page length. People also set a length taller than the terminal and the prompt overwrites the text. Use the terminal's height or less. A pipe and a file use the same length. The flag does not change the file. It changes how much `more` prints before waiting.

## Show a prompt instead of ringing the bell
Also asked as: more -d; silent more; help prompt; --silent; do not ring
`-d` prints a short hint, such as which key continues, instead of ringing the bell when you press a key `more` does not use. It is the quieter prompt. It does not change paging. It does not disable the end-of-file prompt. `-e` is the one that exits at the end.

```sh
more -d -- file
```

People pass `-d` and expect no prompt at all. The hint is still a prompt. People also expect `-d` to suppress a missing-file error. It does not. An unreadable file is still an error on stderr and a non-zero status. `-d` is about the interactive bell. A redirected `more` is not interactive, and `-d` does nothing useful there.

## Plain text, without bold
Also asked as: more -u; suppress underline; plain more; --plain; no backspace bold
`-u` suppresses underlining and bold. Old `more` showed bold with backspace sequences. `-u` prints plain text instead. util-linux `more` also has this flag. It does not strip ANSI color from a modern command. It is the old overstrike filter. A color log still has its escapes.

```sh
more -u -- file
```

People pass `-u` to make a colored diff readable and still see `^[[` sequences. That is ANSI color, not overstrike. `less -R` is the pager that keeps color on purpose. `more` is a poor color pager. `-u` does not change search or paging. A manual page that looks faint may be using overstrike. `-u` makes it plain. The file is unchanged.

## Do not pause on a form feed
Also asked as: more -l; form feed; no pause; --no-pause; Ctrl-L in more
`-l` does not treat a form feed as a page break. Without it, a form feed can make `more` pause as if it filled a screen. `-l` is the flag for a file that uses form feeds as decoration. The pauses you still get are the ordinary screenfuls.

```sh
more -l -- file
```

People pass `-l` expecting `more` not to page at all. It still pages on screenfuls. A form feed is the only extra pause it removes. People also want a follow mode. `more` does not have one. A log that grows needs another tool. `-l` does not change `-e` or the search. It is a single display rule.

## Page a pipeline
Also asked as: command | more; pipe to more; more stdin; page command output; no file more
`command | more` pages that command's standard output. You move forward only. The producer runs as you read, or it fills the pipe and blocks. `q` stops `more` and the producer gets a broken pipe if it is still writing. There is no file name and no percentage, because the length is unknown until the producer exits.

```sh
command -- file | more
```

People pipe into `more` and press `b`. The pipe cannot be reread. Use `less` if you need to go back. People also pipe a command that asks questions. The keyboard belongs to `more`. The command's stdin is not your terminal unless you arranged that. A producer that never closes makes `more` wait at the end forever. `q` is the way out. The producer's status is lost unless the shell has `pipefail`.

## Open several files
Also asked as: more multiple files; next file in more; more file file; switch file; several files
`more -- file file` opens each file in order. At the end of one it moves to the next. A prompt between files says which is next. `:n` skips to the next file on util-linux `more`. A missing file in the list is an error for that name. The others still open.

```sh
more -- file file
```

People expect one continuous scroll with no boundary. The files are separate, and `more` announces the next one. People also pass a glob that matches nothing. The shell leaves the literal star, and `more` says the file does not exist. A directory is an error. `more` will not list it. Use a glob of real files, and `--` before a name that starts with a dash.

## Start in the middle of a long file
Also asked as: more skip ahead; more +number; begin at line; jump forward; do not start at the top
A `+` and a number is the start line. `more +100 -- file` skips the first 99 lines and shows line 100. It still cannot jump there in a pipe without reading and discarding. In a file it seeks. This is the command-line form of "I already saw the top."

```sh
more +100 -- file
```

People use `tail` and `more` together and lose the ability to see the middle. `+100` keeps the rest of the file. A number that is not a line, such as a byte offset, is wrong. `more` counts lines. A `+` command applies to each file in a list. `:n` to the next file starts that file at its `+` line too. Check one file first if the number is large.

## See why more is not less
Also asked as: more versus less; cannot go back; more or less; pager difference; why less
`more` pages forward. `less` pages forward and backward, searches both ways, and follows a file. On a pipe, `less` can scroll back through what it has read. `more` generally cannot. `more` exits on a short file. `less` waits unless you passed `-F`. For a quick look at a short file, `more` is enough. For a log you will search, use `less`.

```sh
less -- file
```

People learn Space and `q` in `more` and those same keys work in `less`. The habit that does not transfer is going back. If `b` does nothing, you are in `more` or on a pipe. `man` often uses `less` even when you asked for a manual. The pager is `PAGER` or `MANPAGER`. Set that to `less` if you want one behavior everywhere. `more` remains the small tool.

## Use it when the file is short
Also asked as: more on a short file; more exits immediately; no prompt; cat versus more; one screen
If the file fits on one screen, `more` prints it and exits. There is no prompt and no `q`. That is intentional. A script that pipes a short command through `more` looks like `cat`. A long command pages. You do not have to measure the file first.

```sh
more -- file
```

People think a missing prompt means `more` failed. Check the status and the text. Empty output and status 0 is an empty file. A usage error is non-zero and prints on stderr. People also want the short-file exit and the long-file pager from `less`. That is `less -F`. `more` already exits on a short file. You do not pass a flag for it.

## Open a name that starts with a dash
Also asked as: more -- -file; more dash file; file named -n; double dash more; operand
A file named `-n` looks like a flag. `more` then complains or waits on stdin. `--` ends options. The next word is a file. `./-n` is the other form. This is the same rule as the other utilities.

```sh
more -- -file
```

People `more -file` and get a usage line. The dash was a flag. `--` is the fix. A `+` at the start of a name can be read as a start-line command. Put unusual names after `--`. A directory after `--` is still an error. `more` will not list it. Pass the files, not the folder.

## What a failed more means
Also asked as: more exit status; more failed; no such file more; cannot open; more error
A missing file exits non-zero after a message on stderr. A directory exits non-zero. A search that finds nothing does not exit. You are still at the prompt. A pipe whose producer fails still shows whatever arrived, and the pipeline status is the last command unless `pipefail` is set. `q` is status 0.

```sh
more -- file
```

People treat a short silent run as failure. A short file exits 0 after printing. People also redirect `more` to a file and get the whole input with no paging, because there is no terminal to page on. `more` is the viewer. The file or the pipe is the input. Check the producer's status separately if a pipeline looked empty. An empty page and a live producer means it has not printed yet, not that `more` failed.
