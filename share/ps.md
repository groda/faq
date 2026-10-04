# ps

`ps` lists processes. It reads the process table and prints a snapshot. It does not kill a process, and it does not refresh unless you run it again. With no flags, Linux `ps` prints processes on your current terminal, not every process on the machine. A daemon, a job in another session, and another user's process are absent until you ask for them. procps-ng on Linux is assumed here. The BSD-style `ps aux` and the UNIX-style `ps -ef` both work on that `ps`. macOS `ps` is BSD. `ps aux` works. `ps -ef` is thinner, and GNU long options such as `--sort` are missing. BusyBox `ps` prints a short list and has almost none of these flags. Exit status is 0 when the table was read. It is non-zero when the options are illegal. An empty list of your terminal's processes is still success.

Basic form: ps

A leading dash selects UNIX options. No dash selects BSD options. `ps aux` is not `ps -aux`. Mixing them is legal on procps and confusing. `-o` chooses columns. The default columns hide the full command line. A name that looks like a flag belongs after the options. `ps` does not read a file.

## List the processes on this terminal
Also asked as: ps default; processes in this session; ps with no flags; current tty; my shell processes
With no flags, `ps` prints processes attached to the current terminal. The columns are the pid, the tty, the time, and the command. Your shell is there. A background job of this terminal is there. A service started by systemd is not. The list is a snapshot. A process that exits before `ps` runs is absent.

```sh
ps
```

People run `ps` and conclude nothing else is running. The filter is the terminal, not the machine. `ps -e` is every process. The command column is truncated to the width of the terminal. A long `python` invocation looks like `python`. `-o args` or `ps -ww` is the full argument list, still cut by the kernel's limit. `ps` does not show kernel threads as a separate class until you ask for all processes.

## List every process
Also asked as: ps -e; ps -A; ps aux; ps -ef; all processes; every user
`ps -e` lists every process. `ps -A` is the same. `ps aux` is the BSD form. It adds the user, CPU percent, memory percent, virtual size, and resident size. `ps -ef` is the UNIX full form. It adds the parent pid and the start time. All three include other users. All three are snapshots.

```sh
ps -ef
```

```sh
ps aux
```

Use `-ef` when you want the parent pid. Use `aux` when you want RSS. Do not write `ps -aux` and expect a portable result. On procps the dash changes the parse. macOS accepts `ps aux` and is picky about GNU flags. A process in another pid namespace, as in a container, is invisible from the host unless you are looking at the host's table of that container's processes. `ps` on the host does not enter the container.

## Choose the columns
Also asked as: ps -o; custom columns; ps -o pid,cmd; only the pid; rss and args; process columns
`-o` sets the columns. Names include `pid`, `ppid`, `user`, `stat`, `rss`, `vsz`, `etime`, `lstart`, and `args`. `args` is the command line. `comm` is the short process name, 15 characters on Linux. `--no-headers` drops the header, which a script wants. `-o` is UNIX style and wants the dash. This is the form to parse. The default table is not.

```sh
ps -e -o pid,user,rss,args --no-headers
```

People cut the default `ps aux` line and the columns shift when a user name is long. `-o` does not shift. `rss` is kilobytes of resident memory, not bytes. `vsz` is virtual size in kilobytes. Virtual size includes mapped files and is not RAM. `etime` is how long the process has been running. `lstart` is when it started. `args` can still be truncated by the kernel. A process can change its own `argv`. The column is what it claims.

## Find a process by name
Also asked as: ps -C; process named nginx; pgrep versus ps; find pid by command; comm exact match
`-C` selects by the short command name, the same 15-character `comm`, matched exactly. `ps -C sshd` lists sshd. It does not match a name that merely contains the string. It does not match the full argument list. `pgrep` is the tool that prints pids by pattern. `ps` is the tool that prints the columns once you have the set.

```sh
ps -C process -o pid,user,etime,args
```

People `ps aux | grep process` and the `grep` line matches itself. The pattern is in the `grep` arguments. `-C` avoids that. A script named `process` and an interpreter whose `comm` is `python` do not match `-C process`. The short name is the interpreter. `args` contains the script. `pgrep -af pattern` searches the full line. macOS `ps` has no `-C`. Use `ps -ax -o pid,comm,args` there and filter.

## Find a process by pid or user
Also asked as: ps -p; ps -u; one pid; processes of a user; ps -p pid; parent process
`-p` selects one or more pids, comma-separated. `-u` selects a user by name or id. `-p` and `-u` can be combined. A pid that is already gone produces no row. On procps that can be a non-zero status. The parent pid is the `ppid` column, not a filter, unless you use `--ppid`. `--ppid` is GNU procps.

```sh
ps -p pid -o pid,ppid,stat,etime,args
```

People pass a process name to `-p` and get an error. `-p` is numeric. `-C` is the name. A user filter does not include processes that dropped privileges unless you name the user they run as now. The `user` column is the effective user. A process can start as root and be `nobody` by the time you look. `-u root` is a long list. It is every root process, not the login shells.

## Read the STAT column
Also asked as: ps stat; process state; R S D Z T; zombie; uninterruptible sleep; what STAT means
`stat` is the process state plus a few flags. `R` is running or runnable. `S` is sleeping, the normal state of a process waiting for an event. `D` is uninterruptible sleep, usually disk. `T` is stopped. `Z` is a zombie. A trailing `+` means it is in the foreground process group. `l` means it is multi-threaded. `s` means it is a session leader. `<` and `N` are priority.

```sh
ps -e -o pid,stat,wchan,args
```

People see `S` and think the process is stuck. Sleeping is idle. `D` that never ends is the stuck one, and it often cannot be killed until the disk I/O finishes. `Z` is already dead. The process has exited, and the parent has not waited for it. Killing the zombie does nothing. The parent must `wait`. `wchan` hints what an `S` or `D` process is waiting on. It is not a stack trace.

## Tell RSS from virtual size
Also asked as: ps rss; ps vsz; resident set; virtual memory; how much ram is this process; %mem
`rss` is the resident set, the pages in RAM, in kilobytes. `vsz` is the virtual address space, also in kilobytes. `%mem` is resident set over physical RAM. A large `vsz` with a small `rss` is a process that mapped a lot and has not touched it, or that mapped a file. Shared libraries are in the RSS of every process that mapped them. Adding RSS double-counts those pages.

```sh
ps -e -o pid,rss,vsz,args --sort=-rss
```

People add the RSS column and expect it to equal `free`'s used memory. It will be larger. Shared pages are counted once in the kernel and once per process here. `--sort=-rss` puts the largest resident set first. The sort is GNU procps. macOS sorts with `-m` for memory. A Java or browser process with a large `vsz` is not that many gigabytes of RAM. Read `rss`. Neither column includes disk cache the kernel is holding for the process's files.

## Sort by memory or cpu
Also asked as: ps --sort; sort by rss; top cpu processes; ps aux --sort; highest memory
`--sort=-rss` sorts by resident set, descending. `--sort=-pcpu` sorts by CPU percent. The minus is descending. The key is a column name. This is GNU procps. The sort happens in `ps`, not in the shell, so it sorts numbers rather than text. `ps aux --sort=-rss` works on procps. A pipe to `sort -n` is the portable form, and the column has to be numeric.

```sh
ps -e -o pid,pcpu,rss,comm --sort=-rss
```

People `ps aux | sort` and the header sorts into the middle, and RSS sorts as text, so `100` comes before `20`. `--sort` avoids that. CPU percent is since the process started, not the last second, on this snapshot. A process that burned a core for one second a day ago looks idle. `top` is the recent sample. `ps` is the lifetime average in `%cpu`. A short-lived process may not appear at all.

## Show the parent and the tree
Also asked as: ps -f; ps --forest; process tree; parent pid; pstree versus ps; who started this
`-f` adds the parent pid. `--forest` draws the tree with ASCII branches. A child has the parent's pid in `ppid`. Pid 1 is the root on a normal system. In a container, pid 1 is the container's init, and parents outside the pid namespace are not visible. A parent that exits reparents its children, usually to pid 1.

```sh
ps -ef --forest
```

People expect the tree to show threads as children. Threads share a pid in this view unless you ask for thread format. `-L` shows threads, and the tree gets noisy. `--forest` is GNU. macOS uses `ps -ax -o pid,ppid,comm` and you reconstruct the tree, or you use `pstree` if it is installed. An orphan is a live process. A zombie is not. The parent column of a zombie is who has not waited.

## Show threads
Also asked as: ps -L; ps -T; threads of a process; thread count; LWP; nlwp
`-L` prints a line per thread, with the light-weight process id in `lwp`. `-T` is the other thread view. `nlwp` on a normal line is the thread count without expanding them. A process with a hundred threads is one pid and a hundred `lwp` values. Signals are delivered to the process. You do not kill an `lwp` with `kill` from the shell and expect a clean result.

```sh
ps -L -p pid -o pid,lwp,stat,comm
```

People see one `ps` line and think the program is single-threaded. `nlwp` says otherwise. People add thread RSS and multiply the process by the thread count. The resident set is the process, not a per-thread copy. `-L` repeats it. A `D` state on one thread can stall the program while the other threads look fine. The thread line is the one to read.

## Show a full argument list
Also asked as: ps -ww; ps args; truncated command; full command line; ps eww; arguments cut off
The command column is cut to the terminal width. `-w` widens it. `-ww` widens it without limit, up to what the kernel still has. `-o args` is the argument vector. A process can overwrite its own arguments, and many servers do that to retitle themselves. The original command is then gone. `comm` stays the short name set by the kernel.

```sh
ps -e -ww -o pid,args
```

People copy a truncated line into `kill` or into a script and match the wrong process. Widen it first. Arguments are visible to other users on the system. A password passed on the command line is in this column. Environment is not, unless you use the `e` modifier, and that output is a different format. `ps eww` is BSD style. It is easy to mix with `-o`. Prefer `-o args` for the command, and do not expect secrets to be hidden.

## Show when a process started
Also asked as: ps lstart; ps etime; process start time; how long has it been running; elapsed time
`lstart` is the start time as a date. `etime` is elapsed time since start, as `[[dd-]hh:]mm:ss`. `bsdstart` is the short start column from `ps aux`. A process that has been running for a year has a large `etime` and an old `lstart`. Restarting a service resets both. A child has its own start, not the parent's.

```sh
ps -e -o pid,lstart,etime,cmd
```

People read the `TIME` column as wall clock. `TIME` is CPU time consumed, not elapsed time. A process can have been up for a week and have a `TIME` of `0:00` if it slept. `etime` is the uptime of the process. It is not reset by a thread starting. A timezone in `lstart` follows the system. Comparing two hosts' `lstart` strings without a timezone is how you mis-order a restart.

## Select by terminal
Also asked as: ps -t; processes on a tty; pts; this terminal; ps T; session processes
`-t` selects a terminal. `ps -t pts/0` lists processes on that pseudo-terminal. A process with `?` in the tty column has no terminal. Daemons and cron jobs look like that. They are still processes. Detaching does not stop them. It only drops the controlling terminal.

```sh
ps -t pts/0 -o pid,stat,args
```

People kill everything on a tty and miss the daemon the session started, because it already called `setsid` and no longer has that tty. `ppid` is how you find those children if they have not reparented. A `?` process is not a zombie. A zombie can have a tty too, until it is reaped. macOS names terminals differently. `ps -t` still selects one. The name comes from the `TT` column.

## What a failed ps means
Also asked as: ps exit status; ps exit code; ps failed; illegal option; cannot list processes
`ps` exits 0 when it printed the snapshot it could. An unknown flag exits non-zero and prints the usage on stderr. A `-p` that matches nothing exits non-zero on procps. An empty default `ps`, because this terminal has a race, can exit 0 with only a header. Permission does not hide other users' processes on a normal Linux system. `hidepid` on `/proc` does.

```sh
ps -e -o pid,comm
```

People script `ps aux | grep` and the grep finds itself, so the script thinks the service is up. Match `-C`, or filter the pid of the grep out. People also treat a non-zero `ps -p` as "ps is broken." The pid is gone. That is the answer. Under `set -e` a missing pid aborts the script. A process can exit between the check and the `kill`. The snapshot was already old when `ps` printed it.
