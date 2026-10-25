# nohup

`nohup` runs a command so that it ignores the hangup signal. When your terminal closes, the shell sends `SIGHUP` to jobs it started. A command under `nohup` keeps running. It does not background the command. The `&` does that. It does not detach from the session as thoroughly as a service manager, and it does not give you a way to reattach. That is `tmux` or `screen`. GNU nohup from coreutils on Linux is assumed here. Bash may have a builtin of the same name. `type nohup` says which one runs. macOS has a BSD `nohup` with the same idea. BusyBox has a thinner `nohup`. Exit status is the status of the command. `nohup` itself fails, non-zero, when it cannot run the command or cannot open the output file it chose.

Basic form: nohup command -- file

The command and its arguments follow `nohup`. A flag of the command is not a flag of `nohup`. `nohup` has no `-f`. `--` before a command that starts with a dash stops `nohup` from reading it as its own option. Quote arguments the shell would expand. If stdout is a terminal, `nohup` appends it to `nohup.out` in the current directory, or to `$HOME/nohup.out` if that fails. If stderr is a terminal, it joins stdout. If stdin is a terminal, it is redirected from an unreadable file, so the command cannot sit waiting for keyboard input.

## Keep a command running after you log out
Also asked as: nohup a command; ignore hangup; run after logout; SIGHUP; survive disconnect
`nohup` sets the command to ignore `SIGHUP`, then execs it. Closing the terminal no longer kills that process. The command is still a child of the shell until you background it. Without `&`, `nohup` waits in the foreground, and closing the terminal can still kill the shell's foreground group before the ignore matters in the way you hoped. The durable form is `nohup` and `&`.

```sh
nohup command -- file &
```

People run `nohup command` with no `&`, close the laptop, and the job dies with the session anyway. Background it. People also expect `nohup` to survive a kill. It ignores hangup only. `SIGTERM` and `SIGKILL` still stop it. A reboot stops it. `nohup` is not a service. systemd, or a real daemon, is what comes back after a reboot.

## Background it as well
Also asked as: nohup and ampersand; nohup &; run in background; disown versus nohup; fork to background
`&` backgrounds the pipeline. `nohup` only ignores hangup. You want both. The shell prints a job number and a pid and returns to the prompt. Output is not the terminal if `nohup` redirected it. `disown` is a different tool. It removes a job from the shell's table so the shell does not send `SIGHUP` on exit. `nohup` makes the process ignore the signal even if it is sent.

```sh
nohup command -- file > file 2>&1 &
```

People `nohup` a command, background it, and still lose it because they started a new shell and the old one got a hangup it forwarded before `nohup` was in the picture. The command has to be the one `nohup` execs. `nohup sudo command &` works only if sudo does not ask for a password. A password prompt has no terminal. Arrange credentials first, or the background job fails immediately.

## Find the output
Also asked as: nohup.out; where does nohup write; nohup output file; HOME nohup.out; appending nohup.out
If you do not redirect, and stdout is a terminal, `nohup` appends to `nohup.out` in the current directory. If it cannot create that file, it appends to `$HOME/nohup.out`. It prints a line saying which file it used. The file is appended, not truncated, on the next run. Stderr goes to the same place when stderr was a terminal. A redirect you wrote yourself wins, and `nohup.out` is not created.

```sh
nohup command -- file > file 2>&1 &
```

People cannot find the output because it went to `nohup.out` in a directory they have since left. The message `nohup` printed named the path. People also redirect only stdout. Stderr is still a terminal, so `nohup` merges it onto stdout's destination. Be explicit with `> file 2>&1` if you care where errors go. Two concurrent `nohup` runs appending to one `nohup.out` interleave their lines. Give them separate files.

## Redirect stdin so it does not wait
Also asked as: nohup stdin; nohup waiting for input; /dev/null stdin; nohup < /dev/null; terminal stdin
If stdin is a terminal, `nohup` redirects it from an unreadable file so a read gets end of file or an error instead of blocking on a keyboard that is about to go away. If you already redirected stdin, `nohup` leaves it. A command that insists on a terminal, such as a password prompt or a full-screen editor, will fail or misbehave. `nohup` does not allocate a fake terminal.

```sh
nohup command -- file </dev/null > file 2>&1 &
```

People `nohup` an installer that asks questions, background it, and find it exited at the first prompt. Feed it answers, or do not use `nohup` for interactive programs. `tmux` is the tool that keeps a terminal. `</dev/null` is the explicit form of what `nohup` already does to a terminal stdin. It does not hurt to write it. A pipe into `nohup` is a different stdin. `nohup` does not replace it with the unreadable file.

## See whether it is still running
Also asked as: nohup pid; check nohup job; is nohup still running; jobs versus ps; nohup background pid
The shell prints the pid when you background the job. `jobs` shows it only in that shell. A new terminal does not have that job table. `ps -p pid` works from anywhere. `pgrep -af pattern` finds it by command line. `nohup` does not write a pid file. The process is an ordinary process with `SIGHUP` ignored.

```sh
ps -p pid -o pid,stat,etime,args
```

People run `jobs` in a second terminal and conclude the command died. `jobs` is per shell. People also look for a process named `nohup`. After exec, the process name is the command, not `nohup`. A `STAT` of `S` is sleeping, which is normal for a server waiting on a socket. It is not a failure. A zombie or a missing pid means it exited. The exit status is in that shell if you `wait` on it, and nowhere if the shell is already gone.

## Ignore hangup and nothing else
Also asked as: nohup signals; SIGTERM still works; nohup does not ignore kill; SIGHUP only; nohup and SIGINT
`nohup` sets the ignore disposition for `SIGHUP` and then execs the command. The command inherits that disposition. Other signals are unchanged. Control-C sends `SIGINT` to the foreground group. A background job is not in that group, so Control-C does not reach it. `kill pid` sends `SIGTERM`. The process dies unless it handles that signal itself.

```sh
nohup command -- file > file 2>&1 &
```

People `kill` a nohup job and think nohup is broken because it died. Hangup was the only signal it ignored. People also expect `nohup` to protect a pipeline. `nohup command | other` protects `command` only if `nohup` is in front of it. The other side of the pipe is a separate process. `nohup sh -c 'command | other'` is the form that puts both under one ignored hangup.

## Use it on a pipeline or a shell snippet
Also asked as: nohup sh -c; nohup a pipeline; nohup multiple commands; nohup bash -c; hangup and pipes
`nohup` takes one command. A pipeline is several. To cover the whole pipeline, run a shell under `nohup` and put the pipeline in the shell. Single-quote the snippet so your current shell does not expand it. The inner shell is the process that ignores hangup, and its children are in its process group. Redirect that shell's stdout.

```sh
nohup sh -c 'command -- file | command -- file' > file 2>&1 &
```

People write `nohup command | other &` and only the left side is under `nohup`. The right side still gets `SIGHUP` when the terminal hangs up. The `sh -c` form is the fix. A builtin such as `cd` is not a command `nohup` can exec. It has to be inside the shell snippet. `nohup` does not take a shell function unless that function is exported and you run bash. Keep the snippet simple.

## Tell it from disown
Also asked as: disown versus nohup; bash disown; remove job from table; SIGHUP from the shell; disown -h
`disown` tells the shell to forget a job. On exit, the shell does not send `SIGHUP` to forgotten jobs. The process does not have the signal ignored, unless you also used `disown -h`, which marks it to not receive hangup. `nohup` ignores the signal inside the process, so a hangup from anywhere is ignored. They solve the same logout problem from opposite sides.

```sh
nohup command -- file > file 2>&1 &
```

People `disown` a job that already received the signal, or they close the terminal before disowning. The signal already went. `nohup` has to wrap the command from the start. `disown` only works for a job of this shell. It does nothing for a process you did not start here. Prefer `nohup` when you are writing the command. Prefer `disown` when the job is already running and you cannot restart it.

## Tell it from tmux and from a service
Also asked as: nohup versus tmux; nohup versus screen; nohup versus systemd; reattach; daemonize
`tmux` and `screen` keep a terminal session you can detach and reattach. The programs inside still have a terminal. `nohup` throws the terminal away and leaves you a file. A service manager starts the process at boot, restarts it on failure, and captures logs. `nohup` does none of that. It is the right tool for a one-off job that should finish after you disconnect.

```sh
nohup command -- file > file 2>&1 &
```

People run a long server under `nohup` and have no way back in when it prompts, and no restart when it crashes. Use a service, or `tmux`, for a server. Use `nohup` for a batch job whose only output is a file. A `nohup` job is still tied to the machine. It does not migrate. It does not survive shutdown. The pid disappears when the process exits, and nothing starts it again.

## Capture stderr on purpose
Also asked as: nohup 2>&1; stderr nohup.out; redirect nohup; nohup errors; merge streams
When stderr is a terminal, `nohup` redirects it to wherever stdout is going. When you redirect stdout to a file and leave stderr as the terminal, you get that merge onto the file. When you redirect both yourself, `nohup` does not add `nohup.out`. `2>&1` has to come after the stdout redirect, or stderr still points at the terminal and `nohup` then wires it to the old stdout.

```sh
nohup command -- file > file 2>&1 &
```

People write `nohup command 2>&1 > file` and errors still hit the terminal, then get merged onto the terminal by `nohup`, while stdout goes to the file. The order is stdout first, then stderr to stdout. People also want a separate error file. `> file 2> file` is legal and is not `nohup`'s business. `nohup` only invents a destination when the stream is still a terminal.

## Avoid a nohup.out you cannot write
Also asked as: nohup.out permission denied; cannot create nohup.out; HOME nohup.out; nohup failed to open; directory not writable
If stdout is a terminal and the current directory is not writable, GNU `nohup` tries `$HOME/nohup.out`. If that also fails, `nohup` exits non-zero and does not run the command. The message names the failure. Redirecting stdout yourself skips the search. A full disk fails the same way once the write cannot proceed.

```sh
nohup command -- file > file 2>&1 &
```

People run `nohup` in `/` or in a directory they do not own, ignore the error, and think the job is running. The status was non-zero and the command never started. Redirect to a path you can write. `nohup.out` is appended forever if you never redirect. It is not rotated. A long job can fill the disk, which then kills the job on the next write. Point output at a log you rotate, or at a directory with space.

## Run it under sudo
Also asked as: sudo nohup; nohup sudo; root nohup; nohup as another user; password prompt
`sudo nohup command` runs `nohup` as root, which execs the command as root. `nohup sudo command` runs `nohup` as you, and sudo may ask for a password on a stdin that is no longer a terminal. The second form fails in the background. The first form is the one that works without a prompt, if sudo is willing. The output file is opened by `nohup`, so its owner is root in the first form.

```sh
sudo nohup command -- file > file 2>&1 &
```

People `sudo nohup` and the redirect is opened by the shell, as them, before sudo runs. That is usually what you want for the log. The command itself is root. A file the command creates is owned by root. `nohup` does not drop privileges. It does not keep them beyond what sudo set. A password prompt with no terminal exits non-zero almost immediately. Check the pid. A dead sudo is not a running job.

## Quote the command's arguments
Also asked as: nohup quoting; nohup and glob; spaces in arguments; nohup --; shell eats the command
The shell expands globs and variables before `nohup` sees them. Quote what the command should see. `nohup` does not run a shell unless you asked for one with `sh -c`. A `*` is files in this directory, now, not later. `--` before an argument that starts with a dash belongs to the command, and it has to be where that command expects it.

```sh
nohup command -- 'pattern' > file 2>&1 &
```

People `nohup rm *` and the shell expands `*` in the current directory, then `nohup` runs `rm` on that list. The expansion is not delayed until logout. A variable in double quotes is expanded now. Single quotes pass the text. `nohup` has almost no options of its own. A `-f` after `nohup` is the command's `-f`, or a usage error if `nohup` still thinks it is reading its own flags. Put the command immediately after `nohup`.

## Watch the output as it runs
Also asked as: tail nohup.out; follow nohup output; see nohup progress; buffered output; nohup and tail -f
If you left the output on `nohup.out`, `tail -f nohup.out` follows it. If you redirected to a file, follow that file. A command that block-buffers stdout when it is not a terminal can sit silent for a long time and then write a chunk. `nohup` did not add the buffer. The C library did, because stdout is a file. `stdbuf -oL` is the usual fix for a program that honors it.

```sh
tail -f -- file
```

People `tail -f` the wrong directory's `nohup.out` and see nothing. The path was in the message `nohup` printed at start. People also expect `nohup` to flush. It does not wrap the command in a pseudo-terminal, so block buffering stays. A program that only prints on exit looks hung. Check the pid and the CPU. Silence in the log is not the same as a dead process.

## What a failed nohup means
Also asked as: nohup exit status; nohup failed; nohup exit code; command not found nohup; nohup status
`nohup` exits non-zero, without running the command, when the command cannot be found or the output file cannot be opened. Once the command starts, the status you can observe is the command's status. In the shell that backgrounded it, `wait pid` reports it. After that shell exits, the status is gone. `nohup` does not log an exit code.

```sh
nohup command -- file > file 2>&1 &
```

People background the job and check `$?`. That is the status of the backgrounding, which is 0 if the shell could fork. It is not the command's status. A "command not found" from `nohup` means the exec failed. A python traceback in the output file means the command ran and failed. Those are different. Look at the file you redirected to. An empty file and a live pid means the command has not printed yet, not that `nohup` failed.
