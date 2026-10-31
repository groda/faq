# xargs

`xargs` reads names from standard input and runs a command with those names as arguments. It does not search. `find` searches. It splits the input into items, then packs as many as fit onto each command. With no command, it runs `echo`. GNU xargs from findutils on Linux is assumed here. An empty input still runs the command once, with no extra arguments, unless you pass `-r`. macOS xargs does the opposite. An empty input runs nothing, and there is no `-r`. BusyBox xargs is thinner and its `-r` and `-P` vary. Exit status is 0 when every command exited 0. It is 123 when a command exited 1 to 125. It is 124 when the command exited 255, which makes xargs stop. A later command is not run after 255.

Basic form: xargs command --

The command and its fixed arguments come after the options. Names from the input are appended. `--` belongs on that command if a name may start with a dash. The input is split on blanks and newlines, and quotes and backslashes are special, unless you pass `-0` or `-d`. Quote a pattern you pass to the command. `xargs` does not quote the items it appends. They are already separate arguments.

## Turn a list into arguments
Also asked as: xargs; stdin to arguments; xargs command; run a command on each name; pack arguments
`xargs` reads standard input, splits it into items, and appends them to the command. It runs the command as many times as it must so the argument list stays under the limit. A short list is one run. A long list is several. With no command, GNU xargs uses `echo`. The items are arguments, not standard input of the command. The command's stdin is `/dev/null` unless you pass `-o`.

```sh
printf '%s\n' file file | xargs command --
```

People pipe a list to `command` and also expect `xargs` to put that list on the command's stdin. The list was consumed as arguments. A command that reads names from stdin sees nothing. `-o` reopens the child's stdin on the terminal, which is what an editor needs. A blank line is not an item. Quotes in the input are processed, so a stray apostrophe makes xargs wait for a close quote and then fail.

## Do not split names on spaces
Also asked as: xargs -0; null separated; find -print0; filenames with spaces; xargs --null
`-0` splits the input on NUL bytes and turns off quote processing. A name with a space or a newline stays one argument. `find -print0` writes that form. So does `printf '%s\0'`. Without `-0`, a space splits the name and `rm` sees two paths. This is the safe pair. A line-oriented list is not safe if names can contain newlines.

```sh
find . -type f -name 'pattern' -print0 | xargs -0 command --
```

People use `-0` on newline-separated input. There is no NUL, so the whole file is one argument. Match the producer. `-d '\n'` splits on newlines only and also turns off quotes. It still breaks on a newline in a name. `-0` does not imply `-n 1`. Many names are still packed onto one command. A command that accepts a single file needs `-n 1` or `-I`.

## Run once per name
Also asked as: xargs -n 1; one argument at a time; xargs -I; replace string; once per line
`-n 1` puts at most one input item on each command. The command still gets its fixed arguments. `-I {}` is the other form. It replaces `{}` in the command with one item, and it runs once per item. `-I` splits on newlines, not on blanks, and it does not pack. Use `-n` when the name is just another argument. Use `-I` when the name must sit in the middle of the command.

```sh
printf '%s\n' file file | xargs -n 1 command --
```

```sh
printf '%s\n' file | xargs -I{} command -- {} file
```

`-I` is required for `cp -- {} {}.bak`, because the second use is not a plain appended argument. `-n 1` cannot build `{}.bak`. `-I` does not pair with `-0` on older GNU xargs the way people hope. Generate newline-safe input, or use `-n` with `-0` when the name is only appended. A `{}` the shell sees is expanded by the shell. Quote the command script, or use a form the shell does not touch.

## Do not run when the list is empty
Also asked as: xargs -r; no run if empty; empty stdin; xargs runs echo once; GNU no-run-if-empty
GNU xargs runs the command once when the input has no items. `xargs rm` on an empty list becomes `rm` with no operands, which is an error. `-r` skips the run when there are no items. The status is then 0. macOS xargs already skips an empty list, and it rejects `-r`. A script that must run on both should not rely on `-r` without a check.

```sh
find . -name 'pattern' -print0 | xargs -0 -r command --
```

People omit `-r` and an empty `find` invokes `rm` or `mv` with only the fixed arguments. `mv -- target` with no sources fails. `command` with no file may read stdin and hang, except xargs already pointed stdin at `/dev/null`. The hang is less common than the wrong-operand error. `-r` does not change a list that contains an empty string item. An empty line is not an item. A quoted empty argument in the input can be.

## Ask before each run
Also asked as: xargs -p; interactive xargs; prompt before command; confirm xargs; dry run is not -p
`-p` prints the command it is about to run and reads a yes or no from the terminal. It does not run the command on a no. It is the confirmation flag. It is not a dry run you can redirect. The question goes to the terminal. `-t` prints the command and does not ask. `-t` is the trace. `-p` is the prompt.

```sh
printf '%s\n' file | xargs -p command --
```

People pass `-p` in a script and the script stops on the question, or the question fails because there is no terminal. `-t` is the script-friendly trace. A no does not stop the rest of the list. xargs asks again for the next batch. `-p` with a packed list confirms the whole batch, not each file, unless you also passed `-n 1`. Look at the printed command. That is the unit you are confirming.

## Show the command it runs
Also asked as: xargs -t; verbose xargs; print the command; trace xargs; see arguments
`-t` prints each command on standard error before it runs it. The line is the built command, not a shell script. Arguments with spaces are shown in a form you can read. `-t` does not ask, and it does not skip the run. It is how you see whether the names were split.

```sh
printf '%s\n' file | xargs -t command --
```

People turn on `-t` and think a bad split was fixed. It was only printed. A name split into two arguments shows as two words on that line. Fix the split with `-0`, then run again. `-t` output is not stable for parsing. It is a diagnostic. The command's own output is still on standard output. They interleave if you merge the streams.

## Stop if the command fails
Also asked as: xargs exit status; xargs 123; command exits 255; xargs keeps going; stop on error
xargs keeps going when the command exits 1. The overall status becomes 123 if any command exited in the range 1 to 125. A command that exits 255 stops xargs immediately, and the status is 124. There is no `--halt` on every build. GNU xargs has `-P` and the exit rule above. A script that must stop on the first failure should make the command exit 255, or run one item and check.

```sh
printf '%s\n' file file | xargs -n 1 command --
```

People see status 123 and think xargs itself crashed. The command failed, and xargs ran the rest. People `set -e` and a pipeline hides the 123 unless `pipefail` is on. The status of a pipeline is the last stage. `xargs` is usually the last stage. A 255 exit is the convention to abort the batch. A normal `grep` miss is 1, so xargs continues. That is the documented rule, not a retry.

## Limit how many run at once
Also asked as: xargs -P; parallel xargs; max procs; run in parallel; xargs jobs
`-P` runs up to that many commands at once. It only helps if each command is a separate run, so combine it with `-n` or `-I`. `-P 4` is four processes. The output of those processes is interleaved. The order is not the input order. A failure still counts toward the exit status. GNU xargs supports `-P`. Some BusyBox builds do not.

```sh
printf '%s\n' file file | xargs -n 1 -P 4 command --
```

People add `-P` without `-n` and get one command, because every name fit on one line. Parallel applies to command invocations, not to arguments inside one invocation. Two runs that write the same file will corrupt it. `-P` does not add a lock. A command that reads stdin will not see the list. xargs still owns the list. `-o` is the terminal reopen, and it is a bad fit with `-P`.

## Put the name in the middle of the command
Also asked as: xargs -I {}; replace; mv with xargs; cp to a suffix; not only appended
`-I {}` replaces every `{}` in the fixed arguments with one input line. The command runs once per line. This is how you build `mv -- {} dir/` or `command -- {} file` where the name is not the last argument. The replacement is not a shell expansion. A `{}` inside single quotes in a shell command you asked xargs to run is still replaced by xargs before the shell starts, if it is in the argv xargs sees.

```sh
printf '%s\n' file | xargs -I{} mv -- {} dir/
```

People write `xargs -I{} sh -c 'mv -- {} dir/'` and the quotes hide `{}` from xargs, so the shell sees the braces. Let xargs replace, or pass the name as a positional parameter: `sh -c 'mv -- "$1" dir/' sh {}`. The second form is the one that survives spaces only if `-I` or `-0` kept the name together. `-I` already splits on lines. Do not add a shell unless you need one.

## Read the list from a file
Also asked as: xargs -a; arg file; xargs --arg-file; list in a file; do not use stdin
`-a` reads the items from a file instead of standard input. The command can then use stdin itself. This is GNU. A `-` file name is not standard input for `-a` on every version. Use `-a file`. The split rules are the same. `-0` still means NUL-separated records in that file.

```sh
xargs -a file -n 1 command --
```

People redirect stdin and also use `-a`, and wonder which wins. `-a` wins. The redirect is unused by xargs. A command that needs the terminal still wants `-o` if something else holds stdin. `-a` does not imply `-r`. An empty file on GNU xargs still runs the command once unless `-r` is set. The same empty-list bug, from a file.

## Split the input on a chosen character
Also asked as: xargs -d; delimiter; split on comma; newline only; xargs --delimiter
`-d` sets the separator and turns off quote processing. `-d '\n'` is one item per line, including lines with spaces. `-d ','` splits on commas. The delimiter is one character. It is not a regular expression. `-0` is the NUL form of the same idea. `-d` is GNU. macOS xargs has `-0` and may not have `-d`.

```sh
printf '%s\n' 'file name' | xargs -d '\n' command --
```

People use `-d` and still pass shell quotes in the file. Quote processing is off, so the quotes are part of the name. `-d '\n'` does not strip a trailing `\r`. A CRLF file gives items that end with a carriage return, and the command looks for the wrong path. Strip the CR in the producer. A trailing delimiter does not create an extra empty item the way some split tools do. Check `-t` if a run looks one short.

## Avoid a command line that is too long
Also asked as: argument list too long; xargs splits; max chars; xargs -s; why not a glob
A glob that expands to too many names fails in the shell with "argument list too long." `xargs` is the usual fix. It packs arguments up to the system limit, then runs the command again. You do not have to set the limit. `-s` lowers it. `-n` limits the count instead of the bytes. Both force more runs.

```sh
find . -name 'pattern' -print0 | xargs -0 command --
```

People still write `command -- *` and hit the limit. The shell expands the glob before `xargs` exists. The list has to arrive on stdin, from `find` or from `printf`. `-n 100` is a count cap. It does not raise the kernel limit. A single argument longer than the limit cannot be run. xargs reports it and, with `-x`, exits. A name that long is rare and usually a bad split that glued the whole file into one item.

## What a failed xargs means
Also asked as: xargs exit code; xargs 123; xargs 124; unmatched quote; xargs failed; no such file
Status 0 means every invocation exited 0. Status 123 means at least one invocation failed with 1 to 125, and xargs kept going. Status 124 means a command exited 255 and xargs stopped. An unmatched quote in the input is a non-zero exit and a message on stderr, and the command may not have run. `-t` shows the runs that did happen.

```sh
printf '%s\n' file | xargs -r -t command --
```

People read 123 as "the list was wrong." The list was used. One command rejected an item. The earlier items may have been processed. xargs does not roll them back. A missing file is the command's error, printed by the command, and 123 from xargs. Fix the item, or use `-n 1` so you can see which run failed. Under `pipefail`, a failing producer fails the pipeline even when xargs would have exited 0. Check both statuses if the producer matters.
