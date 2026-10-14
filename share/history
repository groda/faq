# history

`history` is a Bash builtin. It prints and edits the list of commands this shell remembers. It is not a log of every process on the machine, and it is not `~/.bash_history` until a shell writes that file. With no flags it prints the in-memory list, numbered. GNU Bash 5 on Linux is assumed here. macOS `/bin/bash` is often 3.2 and still has `history`, with the same common flags. Dash and BusyBox ash have no `history` and no `!!`. Zsh has a different `history` and a different expansion syntax. Exit status is 0 unless the option is illegal or the history file cannot be read or written. A command that failed is still stored. The history status is not the status of the old command.

Basic form: history

Flags are Bash options, not options of an external program. `history 20` is a count, not a filename. A filename belongs with `-w`, `-r`, `-a`, or `-n`. Quote a pattern you pass to `grep`. History expansion (`!!`, `!n`, `!$`) runs before anything else, in an interactive shell, and it does not run in a script. A leading space can keep a command out of the list if `HISTCONTROL` says so. That is the shell's rule, not a flag on this command.

## Show the commands you have run
Also asked as: history; show command history; list previous commands; bash history; what did I run
With no arguments, `history` prints the in-memory list for this shell. Each line has a number and the command. The number is the history index, not a pid. A `*` before an entry means it was modified. Commands from other terminals are absent until they have been written and this shell has read them. The list is newest at the bottom.

```sh
history
```

People open `~/.bash_history` and expect it to match. That file is what a previous shell wrote. This shell may not have flushed yet. People also expect failed commands to be omitted. They are stored. `HISTCONTROL` can skip duplicates or commands that start with a space. It does not skip failures. A login shell and a shell started with `--norc` can have different lists. The list you see is the one this process holds.

## Show only the last few commands
Also asked as: history 20; last n commands; recent history; history n; only the end of the list
A number alone prints that many of the newest entries. `history 20` is the last 20. The numbers stay the real history indexes, so the first line of this view is not 1 unless you only have one command. There is no flag for "first 20." The head of the list is the oldest. Pipe to `head` if you want those.

```sh
history 20
```

People pass `-20` and Bash treats it as an option. The count is a bare number. People also run `history 20` in a script and get nothing useful, because a non-interactive shell has no history unless `HISTFILE` was arranged and history was turned on. The count does not search. A command from yesterday is in the last 20 only if you have typed fewer than 20 since. Otherwise search the full list.

## Search the list
Also asked as: history grep; search bash history; find an old command; history | grep; which command contained
`history` has no search flag. The list is text. `grep` on that text is the search. Quote the pattern. A match prints the history number and the command, so you can rerun it with `!n`. `Ctrl-R` in an interactive Bash is a different search. It walks the same list backward and does not print it. It is readline, not `history`.

```sh
history | grep -- 'pattern'
```

People `grep` `~/.bash_history` and miss the commands still only in memory. Grep `history` for this shell, and append to the file first if another terminal must see them. A pattern with a space needs quotes. A pattern that is a flag to `grep` needs `--`. History expansion can fire on `!` inside a double-quoted pattern in an interactive shell. Single quotes stop that. `grep` exits 1 when nothing matches. That is not a broken history.

## Rerun the last command
Also asked as: !!; bang bang; repeat last command; run previous command; history expansion last
`!!` expands to the previous command and runs it. Bash prints the expansion before it runs, unless you turned that off. It is history expansion, and it happens only in an interactive shell. A script that contains `!!` does not rerun anything. `!!` is the last entry, including a command that failed. It is not the last successful command.

```sh
!!
```

People press Enter on `!!` and are surprised the old command runs with today's shell state. The working directory and the variables are now, not then. People also try `!!` in `sh` or in a non-interactive Bash. Expansion is off there. `set -H` turns it on. `set +H` turns it off. A space before `!!` does not make it literal while expansion is on. Quotes do. `'!!'` is two characters.

## Rerun a command by its number
Also asked as: !123; bang number; rerun history entry; history number; run event
`!123` expands to history entry 123 and runs it. The number is the one `history` printed, not a count from the end. A negative form, `!-2`, counts backward. `!-1` is the same as `!!`. Expansion prints the command, then runs it. A number that is not in the list is an event-not-found error, and nothing runs.

```sh
!123
```

People read a line from `~/.bash_history`, count down, and use that count as `!n`. The file has no numbers. The index is whatever this shell assigned. People also use `!123` when they meant argument 123. The event is the whole command. Word designators come after a colon. `!123:0` is the command name only. Expansion runs before aliases and before the command exists. A stale number after you deleted an entry is a different command. Print `history` again before you trust a number.

## Rerun a command by its prefix
Also asked as: !ssh; bang prefix; last command starting with; history by name; !string
`!ssh` expands to the most recent command that started with `ssh`, and runs it. The match is a prefix of the command line, not a search of the arguments. `!s` can match `sudo` if that is newer. `!?pattern` searches anywhere in the line, not only the start. Both are interactive expansion. Both run the command they found.

```sh
!pattern
```

People type `!rm` expecting a search result and instead delete something. The expansion runs. Bash prints it first, which is the moment to hit `Ctrl-C`. `shopt -s histverify` makes expansion fill the line and wait for Enter. That is the safer habit. A prefix of `!` inside double quotes can still expand. Single quotes are literal. An event-not-found error means no line started that way. It does not mean the command was never installed.

## Reuse the last argument
Also asked as: !$; last argument; bang dollar; previous file argument; alt dot
`!$` is the last word of the previous command. `!*` is every argument, without the command name. `:n` picks a word by position, starting at 0 for the command name. `!:2` is the second argument of the last command. These expand in an interactive shell and then run as part of the new command. They do not run the old command again.

```sh
printf '%s\n' !$
```

People use `!$` after a pipeline and get the last word of the whole line, which may be a redirect or a flag. The last word is text, not "the file I meant." `!$` after `sudo cmd file` is `file`. `!:0` is `sudo`. A quoted `!$` is literal. Expansion happens before the command runs, so a new file created by the previous command is a fine target. A variable in the old command was already expanded when it was stored. You get the text that was stored.

## Fix a typo in the last command
Also asked as: ^old^new; caret replace; quick substitution; fix last command; history replace
`^old^new` reruns the previous command with the first `old` replaced by `new`. It is a shorthand for `!!:s/old/new/`. Only the first match changes. A second typo needs another caret, or the full substitute form `!!:gs/old/new/` for every match. This is interactive expansion. It runs the new line.

```sh
^pattern^pattern
```

People expect every occurrence to change. Only the first does, unless you used `:gs`. The separator is `^`, and a `^` inside the text breaks it. Use the `:s/old/new/` form when the text contains a caret. A failed match is an error and the old command does not run. `histverify` is worth turning on before you use this on anything that deletes. The substitution is text. It does not know which word was the filename.

## Delete one entry
Also asked as: history -d; delete a history line; remove one command from history; history delete offset; forget a command
`history -d` removes one entry by its number. A negative offset counts from the end. `-d -1` drops the newest. The later numbers shift down. The file on disk is unchanged until you write it. Anyone with a copy of the old list still has the line. This does not scrub a shell log or another terminal.

```sh
history -d 123
```

People `-d` a password command and then find it in `~/.bash_history`, because this shell had already appended, or another shell wrote the file. Write the file after the delete, and know that a shell which already read the line keeps it. People also delete the wrong index after a previous delete. Print `history` again. `-d` is not `-c`. One entry is `-d`. The whole list is `-c`.

## Clear the in-memory list
Also asked as: history -c; clear history; wipe bash history; history clear; forget all commands
`history -c` deletes every entry in this shell's list. It does not empty `~/.bash_history` by itself. A later `history -w` writes the empty list over the file. A later `history -a` appends nothing useful and leaves the old file. Another terminal keeps its own list. Closing this shell without writing can let the old file survive, or overwrite it, depending on `histappend`.

```sh
history -c
```

People `-c` and reopen a terminal and the old commands are back. The file was never rewritten. `history -c && history -w` clears the memory and then the file this shell is using. It does not clear a file another user copied, and it does not clear the scrollback. A command typed after `-c` is entry 1. `HISTSIZE` is unchanged. You have not disabled history. New commands are stored again.

## Write the list to the history file
Also asked as: history -w; save history; flush history to disk; overwrite bash_history; write HISTFILE
`history -w` writes this shell's list to the history file, replacing the file. The default file is `$HISTFILE`, or `~/.bash_history`. `-w file` writes somewhere else. This is a snapshot of memory, not an append. Commands that were in the file and are not in this shell's list are gone from the file. `history -a` is the append.

```sh
history -w
```

People `-w` from a short-lived shell and truncate a long history file. Use `-a` unless you mean to replace the file. A permission error on the file exits non-zero and the list stays in memory. The file is plain text. Times are written only if `HISTTIMEFORMAT` was set when the commands were stored. Writing does not merge another terminal's commands. It dumps this list.

## Append this session to the file
Also asked as: history -a; append history; share history between terminals; flush new commands; multi session history
`history -a` appends the lines from this session that have not yet been written. It does not remove lines, and it does not pull the other terminal's lines in. The pair people want is `-a` in the shell that just ran something, and `history -n` in the shell that should see it. `shopt -s histappend` makes a normal exit append instead of overwrite.

```sh
history -a
```

People open two terminals, run a command in one, and expect the other to show it. Nothing syncs until a shell writes and the other reads. `-a` is the write. `-n` is the read of lines not already read. A shell that exits without `histappend` can rewrite the file from its own shorter list and drop the other session. Turn `histappend` on if you use more than one terminal.

## Read the history file into this shell
Also asked as: history -r; history -n; reload history; load bash_history; import history
`history -r` appends the history file to the current list. `history -n` appends only lines not already read from that file. `-n` is the one to use after another terminal did `-a`. `-r` can duplicate lines you already have. Neither deletes in-memory lines. A named file, `history -r file`, reads that file instead of `HISTFILE`.

```sh
history -n
```

People `-r` in a loop and the list grows copies. Use `-n` for a sync. A file with timestamps in `HISTTIMEFORMAT` form is read back only if the format matches what Bash wrote. A hand-edited file of plain commands still loads. A file another user cannot read fails, and the status is non-zero. Reading does not make this shell the owner of the other session's numbers. Indexes are local.

## Choose the history file
Also asked as: HISTFILE; bash_history location; change history file; HISTFILE=/dev/null; where is history stored
`HISTFILE` is the path this shell reads at start and writes at exit. Unset it, or set it to `/dev/null`, and this shell does not keep a file. The in-memory list still works. A different file per project is a different `HISTFILE` before the shell starts, or a `history -w` to a path you pass. The variable is not retroactive. The file already read stays loaded.

```sh
HISTFILE=/dev/null
```

People export `HISTFILE` in the middle of a session and expect the old file to be abandoned. Exit still writes wherever `HISTFILE` points at exit. Point it somewhere private before you start a shell that will see secrets, or disable history. A shared `HISTFILE` between users is a race and a permission problem. Bash does not lock it for you. Two shells writing one file can interleave lines.

## Cap how much is remembered
Also asked as: HISTSIZE; HISTFILESIZE; history limit; how many commands; truncate history
`HISTSIZE` is how many commands this shell keeps in memory. `HISTFILESIZE` is how many lines the file may grow to. On exit, Bash trims the file to `HISTFILESIZE`. Setting `HISTSIZE` to 0 and `HISTFILESIZE` to 0 is the usual "turn it off" pair, set before commands run. A huge `HISTSIZE` makes `Ctrl-R` slower and the file larger. The defaults are finite.

```sh
HISTSIZE=10000
```

People set `HISTSIZE` and not `HISTFILESIZE`. The shell remembers more, and the file still truncates. People set them in a script after the commands they wanted to forget. The commands are already stored. Set the sizes in `~/.bashrc` for an interactive shell. A non-interactive Bash does not read `.bashrc` unless you arranged it. `HISTFILESIZE` is not a byte count. It is a line count.

## Skip duplicates and commands that start with a space
Also asked as: HISTCONTROL; ignorespace; ignoredups; ignoreboth; do not save a command; space prefix history
`HISTCONTROL` filters what gets stored. `ignorespace` drops commands that start with a space. `ignoredups` drops a command that matches the previous one. `ignoreboth` is both. `erasedups` removes older copies of a command when it is stored again. The filter runs as the command is stored. It does not rewrite a file that already has the line.

```sh
HISTCONTROL=ignoreboth
```

People prefix a password command with a space and still find it in the file. `HISTCONTROL` was unset, or another shell without that setting ran it. The space has to be the first character, and the option has to be set in that shell. `ignoredups` only compares with the immediately previous entry. An older duplicate stays. `erasedups` is the one that sweeps the list. Neither filter stops `!!` from expanding a command that was stored.

## Stamp each entry with the time
Also asked as: HISTTIMEFORMAT; history timestamps; when did I run that; history date; show time in history
`HISTTIMEFORMAT` is a `strftime` string. When it is set, `history` prints a time before each command. Bash also writes comment lines with an epoch time into the history file, so a later shell can show times. Set the variable before you care about the stamps. Old entries stored without it have no time. The format is per shell. The epoch in the file is the durable part.

```sh
HISTTIMEFORMAT='%F %T '
```

People set the variable and expect yesterday's commands to grow timestamps. Only commands stored while some format was set have times. People also confuse this with `history -w`. Writing does not invent times. A format with a space needs to be quoted in an assignment if you set it from the command line. The printed time is local. The stored value is an epoch. Two shells with different formats still share the epoch.

## Ignore specific commands
Also asked as: HISTIGNORE; do not record ls; history ignore pattern; skip commands; HISTIGNORE pattern
`HISTIGNORE` is a colon-separated list of patterns. A command matching a pattern is not stored. The pattern is a shell pattern, not a regular expression. `ls` matches only `ls`. `ls *` matches `ls` with arguments. This does not delete old entries. It does not apply to a shell that did not set it. A match is silent.

```sh
HISTIGNORE='ls:cd:exit'
```

People put a space-separated list and only the first token works. The separator is a colon. A pattern that is too broad, `*`, stores nothing. `HISTIGNORE` does not replace `HISTCONTROL`. A command can be dropped by either. The ignored command is also a poor target for `!!`, because it was never stored. The previous stored command is what `!!` will run.

## Turn history expansion off
Also asked as: set +H; disable bang history; histexpand; stop !! expansion; literal exclamation
`set +H` turns history expansion off. `set -H` turns it on. With it off, `!!` and `!123` are ordinary words. The `history` command still prints the list. Expansion is the `!` syntax, not the list. Interactive Bash has it on by default. Scripts have it off. `shopt -u histexpand` is the same switch.

```sh
set +H
```

People disable expansion and think history stopped being recorded. Recording is `HISTSIZE` and `HISTFILE`. Expansion is only the `!` rewrite. People also escape a single bang as `\!` and still get expansion in some quotes. Single quotes are the reliable literal. A `!` in a double-quoted string can expand if it looks like an event. Turn expansion off for a session of commit messages and URLs, then turn it back on.

## Make expansion wait for confirmation
Also asked as: histverify; shopt histverify; confirm history expansion; do not run bang immediately; preview !!
`shopt -s histverify` makes `!!`, `!n`, and caret substitution fill the next prompt instead of running. You can edit the line, then press Enter. Without it, the expansion runs as soon as you press Enter the first time. This is the switch to turn on before you use `!rm`. It does not affect the `history` builtin. It affects expansion.

```sh
shopt -s histverify
```

People expect `!!` to always preview. It runs immediately unless `histverify` is set. The preview is the expanded text, not a dry run of side effects. Enter still runs it. `Ctrl-C` discards the filled line. `histverify` does nothing in a script, where expansion is already off. It also does not verify `history -d` or `history -c`. Those are builtins and they run when you call them.

## Append on exit instead of overwriting
Also asked as: histappend; shopt histappend; do not overwrite bash_history; concurrent shells; exit appends
`shopt -s histappend` makes this shell append its lines to `HISTFILE` on exit. Without it, a shell can rewrite the file from its own in-memory list and drop lines another shell had written. This is the setting to put in `~/.bashrc` if you use more than one terminal. It does not sync shells while they are still open. That is still `-a` and `-n`.

```sh
shopt -s histappend
```

People turn it on and expect a second terminal to update live. Exit is when the append happens. A crash can still skip the write. `history -a` after a command you care about is the flush. `histappend` does not merge duplicate lines. Two shells that both ran `export` can both append it. `HISTCONTROL` in each shell is what suppresses duplicates at store time.

## Use fc to edit and rerun
Also asked as: fc; fix command; edit last command; fc -s; history edit and run
`fc` is the other history builtin. `fc` opens the last command in `$EDITOR`, then runs what you save. `fc -s` reruns the last command with an optional substitution, close to `^old^new`. `fc 123` edits entry 123. This is interactive. A script has no editor prompt worth using. `fc` uses the same list as `history`.

```sh
fc -s pattern=pattern
```

People confuse `fc` with `history -w`. `fc` edits and runs. It does not write the history file. An editor that forks and returns immediately can run a half-written command. Use an editor that blocks. `fc -s` with no arguments reruns the last command, like `!!`. A bad offset is an error and nothing runs. The status you see after `fc` is the status of the command it ran, not of `fc` itself.

## Keep secrets out of the list
Also asked as: password in history; do not store a secret; history leak; ignore space password; private command
A secret typed on a command line is stored like any other command, then written to `HISTFILE` on exit. Prefix it with a space if `HISTCONTROL` includes `ignorespace` or `ignoreboth`. Or set `HISTFILE` to `/dev/null` for that shell before you start. Deleting the line with `history -d` after the fact misses copies already written or already read by another shell.

```sh
HISTCONTROL=ignorespace
```

People rely on the leading space and the variable is unset in that shell. The command is stored. People also put the secret in a `curl` URL and then `history | grep` it into a ticket. The list is readable by anyone who can read the file. Mode `600` on `~/.bash_history` is the normal protection. It does not help against root, backups, or a shell that already printed the line. Prefer a prompt or a file mode the command reads itself.

## What a failed history means
Also asked as: history exit status; history failed; event not found; cannot write history; history error
`history` itself fails on a bad flag, a `-d` offset that does not exist, or a file it cannot read or write. Expansion failure is a different message, `event not found`, and it comes from the shell before a command runs. A full disk on `-w` exits non-zero and the in-memory list is unchanged. The status of `!!` after it runs is the status of the rerun command.

```sh
history
```

People treat `event not found` as a broken history file. The list is fine. The `!` text matched nothing. People also run `history` in `sh` and get "not found," because it is a Bash builtin. Use Bash if you need the list. A non-interactive Bash with history turned off prints nothing and exits 0. That empty success is the gotcha in scripts. Turn history on, or accept that scripts do not have this list.
