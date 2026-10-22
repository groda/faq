# tee

`tee` copies standard input to standard output and also writes that same stream to each file you name. It does not search, and it does not create a pipeline by itself. The shell's `|` is what feeds it. With no file it still copies stdin to stdout, which is a no-op you rarely want. GNU tee from coreutils on Linux is assumed here. macOS tee understands `-a` and `-i` and has no `--output-error`. BusyBox tee usually has `-a` and `-i` only. Exit status is 0 when every copy was written. It is non-zero when a named file cannot be opened or a write fails. A broken pipe on stdout also fails the run. Data already written to a file is not rolled back.

Basic form: tee file

Operands are paths. A name that starts with `-` is a flag unless it comes after `--`. The shell opens nothing for you. `tee` creates a missing file and truncates an existing one, unless you pass `-a`. Quote a name with spaces. There is no prompt before an overwrite.

## Copy a stream into a file and still see it
Also asked as: tee a file; split stdout; save and display; pipeline to a file; tee stdout
`tee` reads stdin and writes those bytes to each file and to stdout. A program piped into `tee` does not need to know about the file. The next program in the pipeline sees the same bytes. Order is preserved. `tee` does not add a newline or a header.

```sh
command -- file | tee file
```

People redirect with `>` and then cannot pipe onward, because the redirect consumed stdout. `tee` is the split. People also expect `tee` to buffer a whole file and write it at the end. It writes as it reads. A slow disk slows the pipeline. A missing parent directory is an error, and that file is skipped. Other files and stdout still get the data unless the error mode says to exit.

## Append instead of overwriting
Also asked as: tee -a; append with tee; tee --append; do not truncate; add to a log
`-a` opens each named file for append. Bytes go at the end. Without `-a`, `tee` truncates the file to empty as soon as it opens it, before the first byte arrives. Several files can be mixed. `-a` applies to all of them. There is no per-file append flag.

```sh
command -- file | tee -a -- file
```

People run `tee` twice on a log and the second run wipes the first. They forgot `-a`. People also want append on one file and truncate on another. GNU tee cannot. Run two tees, or append with a second command. `-a` creates the file if it is missing. It does not create parent directories. A permission error on the file exits non-zero after complaining on stderr.

## Write the same stream to several files
Also asked as: tee file1 file2; multiple files; copy stdin to many files; tee several outputs
Every file operand gets a full copy of stdin. Stdout gets one more copy. The writes are interleaved in time, not split across files. Each file receives the same bytes. A failure on one file is reported. GNU tee keeps writing the others unless `--output-error` is set to exit.

```sh
command -- file | tee -- file file
```

People expect the stream to be divided, half in each file. It is duplicated. People also name the same file twice. Both opens truncate it, and the two writers race. Name it once. A directory operand fails for that operand. The rest of the copies continue. `/dev/null` as one operand discards that copy and is a regular, if useless, file operand.

## Ignore interrupt while a pipe is running
Also asked as: tee -i; ignore interrupt; tee SIGINT; Ctrl-C and tee; --ignore-interrupts
`-i` makes `tee` ignore SIGINT. A Control-C in the terminal does not kill `tee` itself. The producer may still die, and then `tee` sees end of file and exits. The flag is for a pipeline you do not want a stray interrupt to cut off at `tee`. It does not protect the other commands.

```sh
command -- file | tee -i -- file
```

People pass `-i` and expect the whole pipeline to survive Control-C. Only `tee` ignores the signal. The left side still gets it unless that command ignores it too. `-i` does not mean "interactive," and it does not prompt. It is not `-a`. A kill -TERM still stops `tee`. The flag is specifically the interrupt signal.

## Save a command's output and pass it on
Also asked as: command | tee | command; middle of a pipeline; capture and continue; log a pipe
Put `tee` in the middle of a pipeline to keep a copy and still feed the next stage. The file gets the bytes that arrived on stdin. The next command gets those same bytes on its stdin. `tee` does not parse lines. A partial last line is copied as a partial last line.

```sh
command -- file | tee -- file | command -- file
```

People put the redirect on the middle command, `command | tee > file | next`, and the next command gets nothing, because the redirect stole stdout. The file belongs as an argument to `tee`. Buffering surprises people. A producer that block-buffers when stdout is a pipe can look stuck. That is the producer, not `tee`. `stdbuf` is the usual hammer. `tee` itself writes through.

## Write a file you do not have permission to truncate
Also asked as: sudo tee; write a root file; permission denied redirect; tee a protected file; echo sudo
A shell redirect opens the file as you, before `sudo` runs. `sudo command > /etc/file` fails if you cannot open that path. `sudo tee /etc/file` runs `tee` as root, so `tee` opens the file. Your shell only writes the pipe. The pattern is `printf ... | sudo tee file >/dev/null` when you do not want a second copy on the screen.

```sh
printf '%s\n' 'pattern' | sudo tee -- file >/dev/null
```

People `sudo tee` and still see the text, then think it was printed twice by mistake. Once is the file, once is stdout. Redirect stdout to `/dev/null` if you only wanted the file. `sudo tee -a` appends as root. A plain `sudo tee` truncates. That is a sharp way to empty a config file. Check `-a` before you run it on a live path.

## Send a copy to the terminal
Also asked as: tee /dev/tty; write to the terminal; see output and redirect; force a tty; show on screen
`/dev/tty` is the controlling terminal. `tee /dev/tty` writes a copy there even when stdout is a pipe or a file. Use it when a later redirect would hide a progress line you still want on the screen. It is a file operand. It fails if the process has no controlling terminal, as in a daemon or some cron jobs.

```sh
command -- file | tee /dev/tty | command -- file
```

People redirect the whole pipeline to a log and lose the prompts. A copy to `/dev/tty` stays on the screen. People also confuse `/dev/tty` with `/dev/stdout`. Stdout may be the pipe. `/dev/tty` is the terminal. Writing secrets to `/dev/tty` still shows them to anyone looking at that session. `tee` does not know they are secrets.

## Do not clobber an existing file by accident
Also asked as: tee noclobber; tee overwrite; truncate file; set -o noclobber and tee; refuse to overwrite
`tee` has no noclobber flag. It truncates an existing file. The shell's `set -o noclobber` does not apply to files `tee` opens. It only applies to the shell's own redirects. If the file must not be truncated, use `-a`, or test for the file before you run, or write to a new name and rename.

```sh
tee -a -- file
```

People turn on noclobber, pipe to `tee file`, and wipe the file. The option never saw that open. A wrapper can refuse: `[ -e file ]` before a truncating `tee`. `-a` is the builtin answer when append is acceptable. There is no `--no-clobber` in GNU tee. Do not invent one. An empty stdin still truncates, because the truncate happens at open, before the first read.

## Diagnose a write error
Also asked as: tee -p; tee --output-error; write error; broken pipe tee; exit on error
`-p` diagnoses write errors on outputs that are not pipes. `--output-error` picks the mode. `warn` reports and continues. `exit` stops on the first write error. `warn-nopipe` is the `-p` default. With no mode, GNU tee exits on a pipe error and warns on a file error. A full disk is a write error. A broken pipe on stdout is a write error.

```sh
command -- file | tee --output-error=exit -- file
```

People see a warning, a non-zero status, and a short file, and think `tee` lost data at random. The destination filled or vanished. The status is the report. `warn` keeps going after a bad file, so the other copies can finish. `exit` is what you want when a partial log is worse than a stopped job. macOS tee has no `--output-error`. `-p` there is not this GNU flag.

## Count on the exit status
Also asked as: tee exit code; tee exit status; tee failed; pipefail tee; did tee work
`tee` exits 0 when stdin was copied to stdout and to every file. It exits non-zero when an open or a write fails. With `pipefail` set, a failure of `tee` fails the pipeline even if the last command succeeded. Without `pipefail`, a pipeline reports the last command only, and a failed `tee` in the middle is hidden.

```sh
command -- file | tee -- file | command -- file
```

People check the output file's size and ignore the status. A failed open leaves the old file untouched, or leaves no file, and the pipeline may still look green. Check `$?` or set `pipefail`. An interrupt of the producer is end of file, and `tee` then exits 0 if its writes succeeded. The short file is the producer's short life, not a `tee` error. The status does not say the content is complete.

## Read nothing and still create the file
Also asked as: empty stdin tee; tee creates empty file; printf pipe tee; no input; truncate on open
If stdin is empty, `tee` still opens each file. A truncating open makes the file empty. An appending open leaves it as it was. `tee` then exits 0. An empty result is not an error. It is a zero-length copy. A script that generates no output still wipes a log if it used `tee` without `-a`.

```sh
printf '' | tee -- file
```

People use `tee` to "ensure the file exists" and are surprised the contents are gone. Create with `touch` if you must not truncate. Use `-a` if you must not lose bytes. A closed stdin, as in `tee file </dev/null`, is the same empty copy. `tee` does not read a filename from stdin. Operands are the files. Stdin is the data.

## Copy a binary stream
Also asked as: tee binary; tee a tar; does tee corrupt binary; null bytes; tee an image
`tee` copies bytes. It does not look for newlines, and it does not treat a NUL as end of file. A tar stream, a gzip stream, or a device image survives `tee` unchanged. Text flags do not exist on `tee`. What corrupts a binary pipe is a command that assumes text, not `tee`.

```sh
command -- file | tee -- file | command -- file
```

People insert `tee` to debug a binary pipeline and then open the file in an editor that rewrites the end. The copy `tee` made was faithful. The editor was not. A terminal that receives binary may beep or change character set. That is the terminal, if stdout is a tty. Redirect or pipe stdout when the stream is not text. `tee` does not add or strip a trailing newline.

## Split with process substitution
Also asked as: tee >(command); process substitution; bash tee fan out; send to a command not a file
Bash process substitution `>(command)` looks like a file. `tee` can write to it, and that command receives the stream on stdin. You can fan out to a logger and a file. This is Bash, not POSIX sh. Dash has no `>(...)`. The process substitution is an operand. Quote nothing around it, or the shell will not set it up.

```sh
command -- file | tee >(command -- file) -- file
```

People quote `'> (cmd)'` and `tee` creates a silly filename. The substitution has to be unquoted. People also expect a failure in the `>(...)` command to fail `tee`. The coupling is a pipe. It often shows up as a broken pipe or a non-zero status, and it can be racy at exit. Prefer a real file operand when you need a durable copy. Use process substitution for a side process, and do not use it in `sh`.

## Watch a log and keep a copy
Also asked as: tee a growing log; follow and save; copy stdin to a log; tail and tee; record a session stream
`tee -a` on a log file is the append form of a recording. A program that writes the log itself does not need `tee`. `tee` is for a program that only writes stdout. `tail -f` reads a file. It does not replace `tee`. One produces the file. The other watches it.

```sh
command -- file | tee -a -- file
```

People `tail -f` a file `tee` has not created yet and get an error. Start the producer, or use a tool that waits. People also name the log on a filesystem that fills. `tee` then errors, and the pipeline stops or warns according to the output-error mode. A log without `-a` is truncated every run. That is the usual reason a cron log only has the last job.

## What a failed tee means
Also asked as: tee failed; cannot open; no space left; tee error; permission denied tee
A non-zero status means an open failed or a write failed. The message on stderr names the path. `No space left on device` is a full filesystem. `Permission denied` is the directory or the file. `No such file or directory` is a missing parent, not a missing file. `tee` would have created the file. A broken pipe means the next command exited. The copy in the named file may still be complete.

```sh
command -- file | tee -- file
```

People rerun the same `tee` and wipe a partial file they needed. Rename the partial copy first, or use `-a` on the retry. A failure with several file operands may have finished some of them. The status does not say which. The messages do. Under `set -e`, a failing `tee` aborts a script only if the pipeline is arranged to expose that status. `pipefail` is that arrangement. Without it, the next command's status is all you see.
