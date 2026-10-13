# lsof
lsof lists every file that a process currently has open. On Unix a “file” is a regular file, a directory, a device, a library, a pipe, a Unix-domain socket, or a network socket. It does not show files that are merely present on disk and not held open, nor does it show routing tables or packet contents. Basic form: lsof
Selection options (`-c`, `-p`, `-u`, `-i`, path names, \ldots) are ORed by default; add `-a` to AND them. Quote any argument the shell would otherwise expand or split, and put `--` before a path that a user will replace if it might start with a dash. Exit status is 0 when every requested search item was found and listed, and 1 when any requested name, PID, user, or address could not be located or when another error occurred (use `-Q` to ignore missing search items). This describes the full lsof package on Linux. macOS ships a closely related lsof whose option set is similar but not identical. Busybox supplies only a minimal stub that prints PID and path and ignores most options.

## What does lsof show by default?

Also asked as: bare lsof; lsof with no options; all open files; default output

With no arguments lsof prints every open file belonging to every process it is allowed to see. Columns are COMMAND, PID, USER, FD, TYPE, DEVICE, SIZE/OFF, NODE, NAME. A non-root user sees only his own processes; root sees everything. People expect a short list of interesting files and are buried in libraries and cwd entries. The gotcha is that the default is the entire system; you almost always want a selection option.

```sh
lsof
```

```sh
sudo lsof
```

## Which process has a file open?

Also asked as: lsof /path; who has this file; process using a file; file is busy

Give the path as an argument. lsof reports every process that has that inode open, with the file-descriptor mode (r, w, u). It works for regular files, directories, and devices. People pass a directory and expect only that directory; they get every open file inside it only with `+D`. The gotcha is that a path that no longer exists on disk (unlinked) still appears if a process holds it; look for `(deleted)` in the NAME column.

```sh
lsof -- /path/to/file
```

## List files opened by a process

Also asked as: lsof -p; files of a PID; what does this process have open; lsof -p pid

`-p` followed by a PID (or a comma-separated list) restricts the list to that process. The output includes cwd, rtd, txt, mem mappings, and ordinary descriptors. People use `-c` by habit and get every process whose name merely starts with the same string. The gotcha is that the count of lines is not the number of file descriptors; cwd, txt and mem do not consume descriptor slots.

```sh
lsof -p 1234
```

## List files opened by a command name

Also asked as: lsof -c; files of a command; all nginx processes; command prefix

`-c` selects processes whose command name begins with the given string (the first characters of argv[0]). Multiple `-c` options are ORed. People expect an exact match and are surprised by partial hits. The gotcha is the prefix rule; there is no built-in exact-match flag, so post-filter or use a PID when you need precision.

```sh
lsof -c nginx
```

## List files opened by a user

Also asked as: lsof -u; files of a user; open files by username; exclude a user

`-u` takes a login name or numeric UID. Prefix with `^` to exclude that user. The result is every file that user’s processes hold. People combine it with other options without `-a` and get the union instead of the intersection. The gotcha is the default OR; see the section on AND.

```sh
lsof -u www-data
```

```sh
lsof -u ^root
```

## Show network connections and sockets

Also asked as: lsof -i; all network files; Internet sockets; TCP and UDP

`-i` selects every Internet socket (TCP, UDP, raw). Add a protocol, address, or port for a narrower set: `-i TCP`, `-i :80`, `-i @192.0.2.1`. The NAME column shows local and remote addresses plus state. People confuse it with `ss` and expect queue lengths or timer data that lsof does not provide. The gotcha is that Unix-domain sockets are omitted unless you also give `-U`.

```sh
lsof -i
```

## Find what is listening on a port

Also asked as: lsof -i :port; who is using port 80; process on a port; lsof port

`-i :port` (or `-i TCP:port`) shows every socket bound to that port, listening or connected. Add `-sTCP:LISTEN` to keep only listeners. You usually want `-n -P` so names are not resolved. People forget the colon or the protocol and match too much. The gotcha is that both IPv4 and IPv6 listeners appear; add `-i4` or `-i6` if you need one family.

```sh
sudo lsof -nP -iTCP:80 -sTCP:LISTEN
```

## Show only listening TCP sockets

Also asked as: lsof listening; -sTCP:LISTEN; TCP listeners; equivalent of ss -l

Combine `-iTCP` with the state selector `-sTCP:LISTEN`. The output is the set of processes that have called listen(). People use only `-i` and then have to filter ESTABLISHED lines by hand. The gotcha is that the state syntax is `-sTCP:LISTEN`, not a free-form filter; a missing colon or wrong case produces a usage error.

```sh
lsof -iTCP -sTCP:LISTEN
```

## Show numeric hosts and ports

Also asked as: lsof -n; lsof -P; do not resolve; raw addresses

`-n` inhibits host-name lookups; `-P` inhibits port-to-service conversion. Together they make the NAME column stable and fast for scripts. People leave them off, then scripts break when DNS is slow or `/etc/services` changes. The gotcha is that both flags are needed; `-n` alone still turns port 22 into `ssh`.

```sh
lsof -nP -i
```

## Find deleted but still open files

Also asked as: lsof deleted; unlinked files; disk full but df shows free; +L1

Files that have been unlinked yet are still held open appear with `(deleted)` in the NAME column and continue to consume space. `+L1` (link count less than 1) selects exactly those. People delete a log, see df unchanged, and restart the wrong service. The gotcha is that you must look at the process that still holds the descriptor; killing it (or closing the fd) releases the blocks.

```sh
sudo lsof +L1
```

## List open files in a directory

Also asked as: lsof +D; files under a directory; recursive open files; +d

`+D dir` walks the directory tree and reports every open file underneath it. `+d` does only one level. The walk can be slow on large trees. People run `+D /` and wait forever. The gotcha is the cost; prefer a specific file or a narrower path when possible, and add `-x` if you need to cross mount points.

```sh
lsof +D /var/log
```

## Output only PIDs

Also asked as: lsof -t; terse output; PIDs for kill; machine readable PIDs

`-t` prints only the process IDs, one per line, with no header. It is the form you pipe to `xargs kill` or `kill`. People use ordinary output and then parse the PID column by hand. The gotcha is that `-t` still respects selection options and the exit-status rules; an empty result exits 1 when a search was requested.

```sh
lsof -t -iTCP:8080 -sTCP:LISTEN
```

## Combine conditions with AND

Also asked as: lsof -a; AND selections; intersection of filters; multiple conditions

By default every selection option is ORed. `-a` changes the combination to AND for the whole command. Items of the same type (several `-c`, several PIDs) are still ORed among themselves before the AND. People place `-a` between two options and expect pairwise logic; it always ANDs everything. The gotcha is the global effect of `-a`.

```sh
lsof -a -u www-data -iTCP
```

## Show Unix domain sockets

Also asked as: lsof -U; Unix sockets; IPC sockets; local sockets

`-U` selects Unix-domain sockets. Combine with other options and `-a` when you want only those belonging to a user or process. The NAME column shows the socket path or an abstract name. People look for them under `-i` and find nothing. The gotcha is that abstract sockets (those starting with `@`) have no filesystem path.

```sh
lsof -U
```

## Repeat / watch mode

Also asked as: lsof -r; repeat every N seconds; watch a port; continuous lsof

`-r` (or `+r`) re-runs the listing every few seconds (default 15, or the number you supply) until interrupted. Useful for watching a port or a file. People expect it to highlight changes; it simply reprints the full list. The gotcha is that each iteration is a fresh snapshot; a process that exits between runs disappears without a special marker unless you use the `+r` form that marks differences.

```sh
lsof -i :22 -r 5
```

## Field output for scripts

Also asked as: lsof -F; machine readable; field mode; parse lsof

`-F` followed by a string of field letters produces one field per line in a form meant for other programs (`p` PID, `c` command, `n` name, \ldots). It is mutually exclusive with `-t`. People try to parse the human columns and break on spaces in paths. The gotcha is that you must request the fields you need; the default `-F` set is minimal.

```sh
lsof -Fpcn -p 1234
```

## Exclude a user or a PID

Also asked as: lsof exclude; caret negation; -u ^user; -p ^pid

Prefix a selection value with `^` to exclude it: `-u ^root`, `-p ^1`. The exclusion is applied before other logic. People try shell negation and get syntax errors. The gotcha is that mixing inclusion and exclusion of the same kind in one option is rejected; use separate options or rethink the filter.

```sh
lsof -u ^root
```

## Show the current working directory of a process

Also asked as: lsof cwd; working directory; -d cwd; where is the process

`-d cwd` selects the current-working-directory entry (the FD column value `cwd`). Combine with `-p` or `-c` and usually `-a`. People look for the path in `/proc/pid/cwd` by hand. The gotcha is that `cwd` is not a numeric descriptor; it appears in the FD column as the literal string `cwd`.

```sh
lsof -a -p 1234 -d cwd
```

## Why is the disk full when df shows free space?

Also asked as: deleted open files; space not freed; lsof +L1 disk; unlinked log

An unlinked file whose last directory entry is gone still occupies blocks until every process closes it. `+L1` or a search for `(deleted)` finds them. People run `df` and `du` and cannot reconcile the numbers. The gotcha is that the space is charged to the process that holds the descriptor, not to any visible path; you must stop or restart that process (or close the fd) to reclaim it.

```sh
sudo lsof +L1
```

## List all TCP connections

Also asked as: lsof TCP; all TCP sockets; -iTCP; established and listen

`-iTCP` (or `-i TCP`) selects every TCP socket regardless of state. Add a state filter or a port if you need less. The NAME column includes the state in parentheses. People expect the same columns as `ss` and miss queue depths. The gotcha is that UDP is excluded; use `-iUDP` or plain `-i` for both.

```sh
lsof -iTCP
```

## Filter by file descriptor

Also asked as: lsof -d; specific FD; descriptor 1 and 2; -d 0,1,2

`-d` takes a comma-separated list of descriptor numbers or names (`cwd`, `txt`, `mem`, \ldots). Only those entries are shown. People use it to reduce noise when they care only about standard streams. The gotcha is that the names are the ones lsof itself prints in the FD column; a numeric descriptor that is closed does not appear.

```sh
lsof -a -p 1234 -d 0,1,2
```

## Show the parent PID

Also asked as: lsof -R; PPID column; parent process; process tree

`-R` adds a PPID column. It does not change which processes are selected. People expect a tree view; they get a flat list with an extra number. The gotcha is that the column appears only when `-R` is given; it is not part of the default output.

```sh
lsof -R -c nginx
```

## What does the FD column mean?

Also asked as: FD column; file descriptor modes; cwd txt mem; r w u

The FD column shows the descriptor number plus a mode letter (`r` read, `w` write, `u` read/write) or a special name (`cwd`, `rtd`, `txt`, `mem`, `DEL`). People treat every line as a numeric descriptor and mis-count open files. The gotcha is that `txt` and `mem` are mappings, not descriptors the process can read or write with read/write.

```sh
lsof -p 1234
```

## How does lsof differ from ss for ports?

Also asked as: lsof vs ss; which to use for ports; process on port; ss or lsof

Both can answer “who is listening on port N”. `ss -tulpn` is faster and shows queue lengths; lsof shows every file the same process has open and works for non-network files as well. People use lsof on a host with tens of thousands of connections and wait. The gotcha is that lsof must walk process tables and `/proc`, so it is slower for pure socket questions.

```sh
sudo lsof -nP -iTCP:80 -sTCP:LISTEN
```

```sh
sudo ss -tulpn sport = :80
```

## Show files that prevent a filesystem from being unmounted

Also asked as: lsof mount point; device busy; +D /mnt; cannot umount

Run lsof on the mount point (or `+D` on it) to see every process that still has a file open on that filesystem. Those processes keep the mount busy. People try `umount` and only see “device is busy”. The gotcha is that a process whose cwd is on the filesystem is enough; look for `cwd` entries as well as ordinary files.

```sh
lsof +D /mnt/data
```

## Show only IPv4 or only IPv6 sockets

Also asked as: lsof -i4; lsof -i6; IPv4 only; IPv6 network files

`-i4` restricts Internet sockets to IPv4; `-i6` to IPv6. They combine with port and state filters. People see a dual-stack listener only under one family and think the other is down. The gotcha is that a socket bound to `::` appears under IPv6; check both when a service should be reachable on either.

```sh
lsof -i4 -iTCP:80 -sTCP:LISTEN
```

## Ignore missing search items in scripts

Also asked as: lsof -Q; exit status 1; quiet missing; no error on absent PID

`-Q` tells lsof to ignore search items it cannot find and to exit 0 unless a real error occurs. Without it a missing PID or path produces exit status 1 even when other items were listed. People write `if lsof -p $pid; then` and are surprised by the failure when the process has already exited. The gotcha is that the default treats “not found” as an error.

```sh
lsof -Q -p 1234
```
