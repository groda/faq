# bash recipes

These are the shell patterns people reuse in scripts and one-liners. They are bash, not a separate program. Bash does not have a job called "loop" or "redirect." It parses a script, expands words, runs commands, and keeps an exit status from the last command. A recipe that works in an interactive bash can fail in `sh`, in a cron job, or on macOS `/bin/bash`, which is still bash 3.2 on many machines. Associative arrays, `mapfile`, `${var,,}`, and `|&` need bash 4. BusyBox ash has no `[[ ]]`, no arrays, and no process substitution. GNU userland on Linux is assumed when a recipe calls `find`, `mktemp`, or `date`.

Basic form: bash script

Words split on spaces unless you quote them. A variable that holds a filename must be `"$file"`, not `$file`. Single quotes hide `$` and globs from the shell. Double quotes expand variables and keep the result as one word. `--` before a path stops the next command from reading a leading dash as a flag. The status of a script is the status of the last command, unless you exit yourself. A pipeline's status is the status of the last stage, unless `pipefail` is on. A test that prints nothing can still be success. `0` is success. Anything else is failure.

## Loop through files in a directory
Also asked as: for file in *; loop over files; bash for loop files; iterate filenames; for f in dir
A `for` loop over a glob runs once per name the shell expanded. Quote `"$file"` when you pass it on. An empty directory leaves the literal `*` unless `nullglob` is set, so the loop body runs once with a file named `*`. Dotfiles are not included. `*` does not match a leading dot.

```sh
for file in -- *; do
  printf '%s\n' "$file"
done
```

The `--` in that glob is wrong if you meant options. The glob belongs in the list, and `--` belongs on the command that receives the name. A directory with spaces is one word only because `"$file"` is quoted. Piping `ls` into `while read` splits names and breaks on spaces. The glob is the boring form. Add `shopt -s nullglob` before the loop if an empty directory should mean zero iterations.

## Read a file line by line
Also asked as: while read line; read lines from a file; bash read file; loop over lines; IFS read -r
`read -r` reads one line and does not treat backslash as an escape. `IFS=` keeps leading and trailing spaces. The input redirect belongs on the `done` of the loop, not on `read`, or the loop runs in a subshell when it is on the left of a pipe and variables set inside it disappear.

```sh
while IFS= read -r line; do
  printf '%s\n' "$line"
done < file
```

A pipe into `while read` is the form that loses the last line when the producer does not end with a newline, and it loses variables after the loop unless `lastpipe` is set. A line is not a filename if the file can contain a newline in a name. Use `find -print0` and `read -d ''` for names. `read` without `-r` eats `\`.

## Send errors to /dev/null
Also asked as: 2>/dev/null; hide stderr; suppress error messages; discard standard error; silence a command
`2>` redirects file descriptor 2, standard error. `/dev/null` throws those bytes away. Standard output is unchanged, so the command's normal output still prints. The status is still the command's status. Hiding the message does not make a failure into success.

```sh
command -- file 2>/dev/null
```

People write `2>/dev/null` on a pipeline and only silence the last stage. The redirect binds to one command. `2>&1 >/dev/null` is the other order and does not hide stderr, because stderr was already copied from the old stdout. The order that hides both is stdout first, then stderr to the new stdout.

## Send output and errors to the same place
Also asked as: >/dev/null 2>&1; redirect stdout and stderr; send both to a file; 2>&1; bash pipe stderr to stdout
`>` sends standard output. `2>&1` then sends standard error to wherever standard output is going now. Written in that order, both streams go to the file, or to `/dev/null`. In bash 4 and later, `&>file` is the same idea. It is not POSIX. `|&` pipes both streams and is also bash 4.

```sh
command -- file >/dev/null 2>&1
```

```sh
command -- file >file 2>&1
```

Use the first when you want silence. Use the second when you want a log. Reversing the redirects leaves stderr on the terminal. A later `|` does not see stderr unless you merged it first. `2>&1` is a copy of the destination, not a promise that the two streams stay in order under load. They are still two streams until the merge.

## Append instead of overwriting
Also asked as: >> append; redirect append; do not clobber a file; >> file; add a line to a log
`>` truncates the file, then writes. `>>` opens the file for append and writes at the end. Both create the file if it is missing. Neither asks. A second `>` in the same script empties the log you just wrote.

```sh
printf '%s\n' 'pattern' >> file
```

People use `>` for a log and then wonder where the old lines went. `noclobber` (`set -o noclobber`) makes `>` fail if the file exists. `>>` still appends. An append from two scripts can interleave lines. It does not interleave partial writes smaller than `PIPE_BUF` in a reliable way you should depend on for records. One script should own the log, or the records should carry their own id.

## Fail if a command fails
Also asked as: set -e; errexit; exit on error; bash stop on failure; set -euo pipefail
`set -e` makes the shell exit when a command returns non-zero. `set -u` makes an unset variable an error. `set -o pipefail` makes a pipeline fail if any stage fails, not only the last. Together they are the usual strict header. They do not apply to a command in a `if` test, or to a command before `||`.

```sh
set -euo pipefail
```

People turn on `-e` and then a `grep` that finds nothing aborts the script, because no match is status 1. A command you expect to fail belongs in `if`. `-e` also does not catch a failure in a command substitution assigned in some bash versions the way people hope. Check `"${PIPESTATUS[@]}"` if you need the status of an earlier stage and did not set `pipefail`. `-u` fires on `$missing`. A default value is `${missing:-}`.

## Use a default when a variable is empty
Also asked as: ${var:-default}; default value; unset variable; parameter expansion default; bash fallback value
`${var:-pattern}` expands to `pattern` when `var` is unset or empty. `${var-pattern}` expands to `pattern` only when `var` is unset. The colon is the difference. Neither assignment stores the default unless you use `:=` or `=`. Quote the expansion when you pass it as one argument.

```sh
printf '%s\n' "${file:-pattern}"
```

People write `$file:-pattern` without braces and the shell looks for a variable named `file:-pattern`. The braces are required. A default that contains spaces must be quoted on the outside, or it splits. `${file:-pattern}` does not mean "the file exists." It only fills in text. An empty string that was set on purpose is replaced by the form with the colon, which surprises scripts that use empty to mean "given, but blank."

## Test a file before using it
Also asked as: test -f; [ -f file ]; [[ -f ]]; file exists; directory test; bash if file
`[[ -f file ]]` is true when the path exists and is a regular file. `-d` is a directory. `-s` is a file that exists and is not empty. `-x` is executable by you. `[[ ]]` is bash. `[` is the POSIX test, and it needs spaces and a quoted `"$file"`. A missing quote turns one path with a space into a syntax error.

```sh
if [[ -f $file ]]; then
  printf '%s\n' "$file"
fi
```

That example is the wrong quoting, and it is the bug. The path belongs in quotes: `[[ -f "$file" ]]`. `[[ ]]` does not split variables, but a leading dash in the name can still look like a test operator if you forget `--` is not how `[[ -f ]]` works. Use `[ -f "$file" ]` in a script that must run under `sh`. `-e` is "exists," including a directory, a socket, or a broken symlink's presence is false because the target is missing. A symlink to a file passes `-f`.

## Compare numbers and strings
Also asked as: [[ == ]]; compare strings; integer compare -eq; bash if equals; [[ -gt ]]
Inside `[[ ]]`, `==` compares strings. `-eq`, `-ne`, `-gt`, and `-lt` compare integers. A string compare of `10` and `2` is not numeric. `=` is the POSIX string operator inside `[`. `==` inside `[` is not portable. Quote the right-hand string unless you mean a pattern.

```sh
if [[ "$a" == 'pattern' ]]; then
  printf '%s\n' "$a"
fi
```

```sh
if [[ "$n" -gt 1 ]]; then
  printf '%s\n' "$n"
fi
```

Use the first for text. Use the second for integers. An unquoted right-hand side of `==` inside `[[ ]]` is a glob, so `*` matches any string and the test is accidentally true. Empty and unset fail `-gt` under `set -u`, and without it bash complains that the integer expression is missing. `[[ ]]` is not `sh`. BusyBox ash does not have it.

## Run a command only if the previous one worked
Also asked as: && and; || or; short circuit; run if success; command and command; if previous succeeded
`&&` runs the next command only when the previous status was 0. `||` runs the next only when the previous status was non-zero. The status of the list is the status of the last command that ran. They are not `if`. They do not group unless you add braces.

```sh
command -- file && printf '%s\n' 'pattern'
```

People chain `cmd || echo failed` and the echo succeeds, so the list succeeds even though `cmd` failed. The fix, when failure must stay failure, is `cmd || { printf '%s\n' 'pattern' >&2; exit 1; }`. Braces are a group in the current shell. Parentheses are a subshell, and an assignment inside them is lost. `&&` has a higher precedence than `||`. Write the chain the way you would read it, or use `if`.

## Change directory and stop if it failed
Also asked as: cd or exit; cd failed; do not continue in the wrong directory; pushd; bash cd error
`cd` returns non-zero when the directory is missing or not searchable. Without `set -e`, the script keeps going in the old directory and writes files there. Test `cd`, or turn errexit on before it. `cd --` matters when the path starts with a dash.

```sh
cd -- dir || exit 1
```

People `cd dir` and then `rm -rf *` in a script. If the `cd` failed, the `rm` runs where the script started. That is the expensive form of this bug. `pushd` and `popd` are interactive habits. A script that must return belongs in a subshell `( cd -- dir && command )`, which does not change the caller's directory. A failure inside that subshell does not move the parent either.

## Make a temporary file
Also asked as: mktemp; temporary file; bash temp file; mktemp -d; secure temp directory
`mktemp` creates a file with a unique name and mode `600`. `mktemp -d` creates a directory. The template must end in `XXXXXX`. GNU `mktemp` on Linux accepts that. Use the printed path. Do not invent `/tmp/myscript.$$` if you can call `mktemp`. The shell does not delete the file for you.

```sh
tmp=$(mktemp)
printf '%s\n' 'pattern' > "$tmp"
```

People put the template in quotes wrong, or they use `mktemp` from a macOS box and pass a GNU-only `--suffix`. Stick to `mktemp` and `mktemp -d`. Remove the file in a trap if the script can be interrupted. A temp file in the current directory is still a temp file. `/tmp` is shared. The mode `600` does not stop root, and it does not stop a symlink race if you build the name yourself.

## Run a cleanup when the script exits
Also asked as: trap EXIT; bash trap; cleanup on exit; trap err; remove temp on exit
`trap 'command' EXIT` runs the command when the shell exits, including after a normal end. The string is parsed when the trap runs, so expand variables then, or they may be empty if you single-quoted too early. `EXIT` is the reliable hook. `ERR` fires on a failing command when errexit cares, and it is bash.

```sh
trap 'rm -f -- "$tmp"' EXIT
```

People trap `EXIT` and call `exit` inside the trap, which runs the trap again on some setups and loops. A trap should remove its files and return. Quoting `"$tmp"` matters if the name ever contains a space. An unset `tmp` under that trap, with the quotes, removes a file named empty only if you forgot the variable. Set the trap after `mktemp` succeeds, or the trap removes nothing useful and can delete the wrong path.

## Pass filenames that contain spaces
Also asked as: null delimited; find -print0; xargs -0; filenames with spaces; bash read -d
Newlines and spaces are legal in names. A line-oriented loop breaks on them. `find -print0` writes a NUL after each name. `xargs -0` reads those records. Bash can read them with `read -d ''`. The NUL is the separator because it cannot appear in a path.

```sh
find . -type f -name 'pattern' -print0 | xargs -0 command --
```

People pipe `find` to `xargs` without `-0` and a file named `a b` becomes two arguments. `xargs` also runs the command once with many names. If the command accepts only one file, use `-n 1`, or a `while IFS= read -r -d ''` loop. `find -exec command -- {} +` is the form that does not need the pipe. Prefer it when you do not need a shell in the middle.

## Check that a command exists
Also asked as: command -v; which is not for scripts; type a command; test if installed; hash a command
`command -v name` prints the path or the function and returns 0 when the shell can run `name`. It returns non-zero when it cannot. `which` is an external program, its output is not portable, and it does not see shell functions. In a script, `command -v` is the test.

```sh
command -v command >/dev/null
```

Replace the second `command` with the name you mean. The example is the shape. People test `-x /usr/bin/name` and miss a command that lives elsewhere on `PATH`. `command -v` follows `PATH`. It does not prove the command works, only that the shell found a way to invoke that name. A function and an alias count. `hash -r` clears the shell's remembered location after you install a tool in the same script.

## Slice a string from the front or the back
Also asked as: ${var%pattern}; ${var#pattern}; remove suffix; remove prefix; basename without basename; parameter expansion
`${file%pattern}` removes the shortest match of `pattern` from the end. `${file%%pattern}` removes the longest. `#` and `##` do the same from the front. The pattern is a glob, not a regular expression. `*` is greedy in the long form. The original variable is unchanged.

```sh
printf '%s\n' "${file%.txt}"
```

People use `basename` and `dirname` in a loop over thousands of files and pay for a process each time. The expansion is enough for a suffix. It does not check that the suffix was there. A file named `file` stays `file` if you remove `%.txt`. Quote the expansion. `${file%.*}` removes the shortest trailing dot-suffix, and it also turns `.bashrc` into an empty string when the whole name matches. Look at the result before you delete from it.

## Remember the status of a pipeline
Also asked as: PIPESTATUS; pipefail; pipeline exit code; last command won; bash pipeline status
Without `pipefail`, `cmd1 | cmd2` returns the status of `cmd2`. A failing `cmd1` is hidden if `cmd2` succeeds. `set -o pipefail` makes the pipeline fail when any stage fails. `${PIPESTATUS[0]}` is the first stage, `${PIPESTATUS[1]}` the second, and the array is overwritten by the next command.

```sh
set -o pipefail
command -- file | command -- file
```

People inspect `$?` after a pipeline and think it is the producer's status. It is the last stage unless pipefail is on. Copy `PIPESTATUS` immediately if you need every stage. `pipefail` is bash. It is not in older `sh`. A `grep` in the middle that finds nothing is status 1, and under `pipefail` the whole pipeline fails. That is what you asked for. Do not set `pipefail` and then treat "no lines" as a disaster unless you mean to.

## Feed a command a here document
Also asked as: heredoc; <<EOF; here document; embed a block of text; <<'EOF'
`<<'EOF'` feeds the following lines to the command's standard input until a line that is only `EOF`. Single-quoted `EOF` does not expand `$` or backticks in the body. An unquoted `EOF` does. The closing word must be at the start of the line unless you use `<<-`, which strips leading tabs and not spaces.

```sh
command -- file <<'EOF'
pattern
EOF
```

People indent the closing `EOF` with spaces and the shell reads the rest of the script as the document. The terminator is not found, and the error points at the end of the file. Quotes on the word also change expansion. Use quotes unless you mean to substitute variables. A here document is not a here string. `<<<'pattern'` is a here string, bash only, and it is one line.

## Loop over the script's arguments
Also asked as: "$@" ; for arg; script arguments; shift; bash positional parameters
`"$@"` is every argument, each as its own word, including empty arguments and arguments with spaces. `$*` joins them on the first character of `IFS`. A `for` loop over `"$@"` is the way to walk the arguments without splitting. `$#` is the count. `$1` is the first. `shift` drops the first.

```sh
for arg in "$@"; do
  printf '%s\n' "$arg"
done
```

People write `for arg in $@` and a filename with a space becomes two iterations. The quotes on `"$@"` are the whole fix. `for arg; do` with no list is a bash shorthand for the same loop. It is not POSIX. After `shift`, `$1` is the old `$2`. A script that inspects `$1` for a flag still needs `--` when it passes the remaining names to another command.

## Build a command without splitting names
Also asked as: bash array; store arguments; "${arr[@]}"; safe argument list; do not eval
A bash array stores words. `"${args[@]}"` expands to those words, one argument each, without splitting on spaces. Build the command in the array and run it as `"${args[@]}"`. `eval` on a string is how a space or a quote in a filename becomes syntax. You do not need `eval` to hold arguments.

```sh
args=(command -- "$file")
"${args[@]}"
```

People accumulate a command in a string and then run `$cmd`. The split happens, and a name that starts with a dash becomes a flag. The array form keeps each element intact. Arrays are bash. BusyBox ash does not have them. `"${args[*]}"` joins on spaces and is the wrong expansion when you meant separate arguments. `@` is the list. `*` is one string.

## Time a command or stamp a log line
Also asked as: date +%Y-%m-%d; ISO date; timestamp a log; epoch seconds; date -u
GNU `date` prints a format you give after `+`. `%Y-%m-%d` is the calendar date. `%H:%M:%S` is the time. `date -u` is UTC. `date +%s` is seconds since the epoch, which is the value to subtract when you want a duration. macOS `date` does not accept the same `-d` as GNU date.

```sh
date -u +'%Y-%m-%dT%H:%M:%SZ'
```

People parse `date` default output, which changes with the locale. The format string is the stable form. A log line wants the timestamp in the record, not as the only thing the script prints. `date` is an external command. The shell parameter `SECONDS` counts seconds since the shell started and does not call `date`. Use that for a rough elapsed time inside one script.

## Match a string against a pattern
Also asked as: case in; bash case; pattern match; glob case; [[ == pattern ]]
`case` matches a word against globs and runs the first arm that fits. `*` is the default arm. Quote the word. Do not quote the pattern if you want glob matching. `case` is POSIX and works in ash. Inside bash `[[ ]]`, `==` with an unquoted right-hand side is also a glob, and a quoted right-hand side is a literal string.

```sh
case "$file" in
  *.txt) printf '%s\n' "$file" ;;
  *) printf '%s\n' 'pattern' ;;
esac
```

People use `grep` to test a suffix and pay for a process, and they get the wrong answer when the name contains a character `grep` treats as a regex. `case` is the suffix test. The first matching arm wins, so put the specific pattern before `*`. A pattern of `*` matches everything, including an empty string. There is no fall-through unless you write `;&`, which is bash 4 and usually a mistake.

## Wait for background jobs
Also asked as: wait; background &; bash jobs; $!; wait for pid; run in background
`command &` starts the command and does not wait. `$!` is the process id of the last background command. `wait` with no argument waits for every child of this shell. `wait "$pid"` waits for one and returns that command's status. A script that exits without waiting can have its children killed when the shell goes away.

```sh
command -- file &
pid=$!
wait "$pid"
```

People background a job and check `$?` immediately. That status is the status of starting the job, not of the job. The status arrives at `wait`. `$!` is overwritten by the next `&`. Save it first. A background job in a pipeline is the pipeline, and `$!` is the last stage. Output from the job still goes to the terminal unless you redirected it. Two jobs writing the same file will interleave.

## Print a message on stderr
Also asked as: >&2; echo to stderr; printf stderr; error message; do not mix errors with output
Standard error is descriptor 2. `>&2` after a command sends that command's standard output to standard error. Error text belongs there so a pipe on stdout stays clean. `printf` does not add a newline unless you write `\n`. `echo` does, and its flags are not portable.

```sh
printf '%s\n' 'pattern' >&2
```

People `echo` the error and then pipe the script's output into another command, and the error goes down the pipe with the data. The redirect has to be on the `printf`, or on a group `{ printf '%s\n' 'pattern' >&2; }`. `>&2` is not `2>&1`. One sends this command to stderr. The other sends stderr to stdout. A status is still separate. Printing an error does not fail the script. `exit 1` does.
