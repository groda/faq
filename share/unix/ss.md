# ss
ss shows socket statistics that the kernel already keeps. It lists TCP, UDP, Unix domain, packet, and other sockets, with their states, queue lengths, addresses, and optionally the processes that own them. It does not show routing tables, interface counters, or packet captures; those jobs belong to `ip`, `nstat`, and tools such as `tcpdump`. It does not invent sockets that the kernel does not know about. Basic form: ss
Options come first, then an optional filter expression. Filters use a small language (`state`, `sport`, `dport`, `src`, `dst`) and must be quoted when they contain spaces or shell metacharacters. Short and long options can be mixed. Exit status is 0 when the command runs and prints its result (even if the result is empty), and non-zero on usage errors or failure to open the socket tables. This describes the iproute2 `ss` on Linux. macOS does not ship `ss`; use `netstat`. Busybox does not include `ss`.

## What does ss show by default?

Also asked as: default ss output; ss with no options; what sockets does bare ss list; ss established only

With no arguments ss prints non-listening sockets that already have a connection (mainly TCP in ESTABLISHED or related states, plus connected Unix sockets). Listening sockets and unbound UDP sockets are omitted, so a quiet machine can look almost empty. People confuse this with “all sockets” and then wonder why their web server does not appear. The one gotcha is that the default is deliberately narrow; add `-a` or `-l` when you want listeners.

```sh
ss
```

## Show all sockets

Also asked as: ss -a; ss --all; list every socket; include listening and unconnected

`-a` (or `--all`) includes listening, unconnected, and connected sockets of every family that ss knows about. The output grows quickly and mixes TCP, UDP, Unix, and netlink. People who want only listening ports still need `-l` or a state filter; `-a` alone is not a substitute for “what is bound”. The gotcha is that Unix and netlink sockets dominate the list on a typical desktop, so you usually combine `-a` with a family flag.

```sh
ss -a
```

## Show listening sockets

Also asked as: ss -l; ss --listening; list open ports; what is listening

`-l` (or `--listening`) restricts the list to sockets in the listening state. Pair it with a family (`-t`, `-u`) so you do not also see every listening Unix socket. The command prints the local address and port (or service name) and the peer as `*`. People confuse it with the default and expect established connections as well. The gotcha is that a socket that is bound but not yet listening does not appear; use `-a` or a state filter for those.

```sh
ss -tl
```

## Show TCP sockets

Also asked as: ss -t; ss --tcp; list TCP connections; TCP only

`-t` (or `--tcp`) limits output to TCP. By default you still see only non-listening sockets; add `-a` for everything or `-l` for listeners only. The state column uses the usual TCP names (ESTAB, LISTEN, TIME-WAIT, CLOSE-WAIT, \ldots). People expect `-t` alone to show listening ports and are surprised when it does not. The gotcha is that the flag selects the protocol family, not the connection state; state is controlled separately.

```sh
ss -t
```

```sh
ss -ta
```

## Show UDP sockets

Also asked as: ss -u; ss --udp; list UDP; UDP listeners

`-u` (or `--udp`) selects UDP. Because UDP has no connection state machine, almost every UDP socket is UNCONN, so `ss -u` by itself usually prints nothing. Always pair it with `-a` or `-l`. People who come from netstat expect the same behaviour as TCP and think UDP is broken. The gotcha is exactly that missing `-a`/`-l`; the flag alone is not enough.

```sh
ss -ua
```

## Show Unix domain sockets

Also asked as: ss -x; ss --unix; list Unix sockets; IPC sockets

`-x` (or `--unix`) shows Unix domain sockets (stream, datagram, seqpacket). As with UDP, you normally want `-a` so listening and unconnected sockets appear. The Netid column uses `u_str`, `u_dgr`, etc. People confuse these with network sockets and look for IP addresses that will never be present. The gotcha is that many system services (dbus, systemd, X11) appear only here, not under `-t`.

```sh
ss -xa
```

## Show the process that owns a socket

Also asked as: ss -p; ss --processes; which process is using this port; show PID

`-p` (or `--processes`) appends a Process column with the command name, PID, and file descriptor. You need root (or equivalent capability) to see processes owned by other users; without it you see only your own. The format is `users:(("name",pid=N,fd=M))`. People run it as a normal user and conclude nothing is listening. The gotcha is the privilege requirement; also, some sockets are owned by the kernel and show no users.

```sh
sudo ss -tp
```

## Show numeric addresses and ports

Also asked as: ss -n; ss --numeric; do not resolve names; raw ports

`-n` (or `--numeric`) suppresses service-name and (when combined with other options) host-name lookups, printing raw numbers. This is faster and stable for scripts. People leave it off, then wonder why port 22 appears as `ssh` and why the command is slow when DNS is broken. The gotcha is that `-n` does not affect the filter language; you still write `:22` or `:ssh` in filters according to the version’s rules.

```sh
ss -tn
```

## Find which process is listening on a port

Also asked as: who is listening on port 80; ss sport = :80; process on a port; ss -tulpn port

Combine `-l`, `-p`, `-n`, and a family, then a filter that matches the local port. The filter must be quoted so the shell does not split it. The command prints the matching listening socket and its process. People forget the quotes or use `dport` by mistake and see nothing. The gotcha is that the filter is applied after the options; a missing `-l` or `-p` (or missing root) produces an empty or incomplete result.

```sh
sudo ss -tulpn -- 'sport = :80'
```

## Show only established TCP connections

Also asked as: ss state established; established connections only; ss -t state ESTABLISHED

The filter keyword `state` followed by a TCP state name keeps only sockets in that state. `established` (or `ESTABLISHED`) is the common case. The output is the same columns as a normal TCP listing. People write the state in the wrong place or forget that the keyword is required. The gotcha is case: most versions accept both forms, but a typo produces a usage error rather than an empty list.

```sh
ss -tn state established
```

## Show sockets in a particular state

Also asked as: ss state TIME-WAIT; filter by state; CLOSE-WAIT sockets; ss state listening

Any of the documented TCP states can be used after `state`. Group keywords such as `connected`, `synchronized`, or `bucket` are also accepted and expand to several states. The command simply drops non-matching rows. People try to combine states with `|` inside the shell and get a syntax error. The gotcha is that the filter language is not a shell expression; use the documented `or` form or run the command twice.

```sh
ss -tn state time-wait
```

```sh
ss -tn state connected
```

## Show a summary of socket counts

Also asked as: ss -s; ss --summary; socket statistics summary; how many sockets

`-s` (or `--summary`) prints aggregate counts by protocol and a few TCP sub-states instead of the per-socket table. It does not parse every socket, so it stays fast on a busy host. People expect the detailed list and are surprised by the short report. The gotcha is that the numbers come from kernel counters, not from a filtered view; you cannot pass a port filter and still get a summary of only that port.

```sh
ss -s
```

## Show timer information

Also asked as: ss -o; ss --options; TCP timers; keepalive timer

`-o` (or `--options`) adds a timer column showing active timers (keepalive, retransmission, time-wait, \ldots) with their remaining time and retransmission count. Useful for spotting stuck or slow connections. People look for this data in the default columns and conclude it is missing. The gotcha is that a zero or absent timer is normal for an idle established connection that has not yet started its keepalive.

```sh
ss -tno
```

## Show internal TCP information

Also asked as: ss -i; ss --info; RTT and cwnd; congestion window

`-i` (or `--info`) appends internal TCP metrics: congestion algorithm, RTT, MSS, congestion window, bytes sent/acked, retransmissions. The extra data usually appears on a second line unless you also use `-O`. People expect a single neat column and then break scripts that assume one line per socket. The gotcha is the multi-line format; add `-O` when you need to pipe the output.

```sh
ss -tin
```

## Show socket memory usage

Also asked as: ss -m; ss --memory; socket buffers; memory per socket

`-m` (or `--memory`) adds kernel memory counters for the socket (receive and send buffer usage, etc.). It is useful when a process holds unexpectedly large buffers. The numbers are protocol-specific and not the same as the Recv-Q/Send-Q byte counts. People confuse the two and think a large memory figure means a large queue of unread data. The gotcha is that memory can be reserved even when the queues are empty.

```sh
ss -tm
```

## Show only IPv4 sockets

Also asked as: ss -4; ss --ipv4; IPv4 only; no IPv6

`-4` (or `--ipv4`) restricts the listing to IPv4. It is an alias for `-f inet`. Combine it with the usual family and state flags. People who dual-stack a service see only half the listeners and think the other half is down. The gotcha is that a socket bound to `::` may still appear under IPv6; check both if you need the full picture.

```sh
ss -tln -4
```

## Show only IPv6 sockets

Also asked as: ss -6; ss --ipv6; IPv6 only; inet6

`-6` (or `--ipv6`) restricts the listing to IPv6. Dual-stack listeners bound to `::` appear here. The rest of the options work exactly as they do for IPv4. People filter for a numeric IPv4 address and wonder why nothing matches. The gotcha is the address family of the bind; a socket listening on `0.0.0.0` never appears under `-6`.

```sh
ss -tln -6
```

## Filter by remote address or port

Also asked as: ss dst; connections to an IP; dport filter; remote port

After the options you may give `dst address` or `dport = :port` (quoted). The filter keeps only sockets whose peer matches. It works with both listening and established sockets, but a listening socket’s peer is usually `*`. People put the filter before the options or forget the quotes and get a shell error. The gotcha is that `dst` matches the peer, not the local address; use `src` for the local side.

```sh
ss -tn dst 192.0.2.1
```

```sh
ss -tn -- 'dport = :443'
```

## Filter by local port

Also asked as: ss sport; local port filter; source port; sport = :22

`sport = :port` matches the local port. Quote the expression. Combine with `-l` when you care only about listeners. The command drops every socket whose local port does not match. People use `dport` by habit from packet filters and see the wrong set. The gotcha is the colon before the port number; omitting it is a syntax error in the filter language.

```sh
ss -tln -- 'sport = :22'
```

## Count established connections

Also asked as: how many established; count TCP connections; ss | wc; number of connections

Use `-H` to drop the header, a state filter, and `wc -l`. The pipeline counts lines, each of which is one socket. People forget `-H` and count the header as a connection, or they use `-a` and include listeners. The gotcha is that the count is of sockets, not of unique remote hosts; a single client with many connections increases the number.

```sh
ss -tnH state established | wc -l
```

## Suppress the header line

Also asked as: ss -H; no header; ss for scripts; machine readable

`-H` removes the column titles so the first line is already data. It is the reliable way to feed ss into awk, sort, or wc. People still reach for `tail -n +2` and then break when the header text changes. The gotcha is that `-H` does not change the column order or the multi-line behaviour of `-i`/`-m`; you may still need `-O`.

```sh
ss -tnH state established
```

## Print each socket on one line

Also asked as: ss -O; ss --oneline; single line output; no wrapped lines

`-O` (or `--oneline`) forces every socket onto a single line even when `-i`, `-m`, or `-e` would normally wrap. Scripts that assume one record per line then work. People enable the detail flags and then discover their awk scripts skip half the data. The gotcha is that the line becomes very long; you still need a field parser that understands the extra tokens.

```sh
ss -tinO state established
```

## What do Recv-Q and Send-Q mean?

Also asked as: Recv-Q Send-Q meaning; queue columns; backlog; unread bytes

The two queue columns change meaning with state. On a LISTEN socket Recv-Q is the number of completed connections waiting for accept, and Send-Q is the backlog limit. On an established socket Recv-Q is unread bytes and Send-Q is unacknowledged sent bytes. People treat the numbers as always “bytes in the buffer” and mis-diagnose a full accept queue. The gotcha is exactly that state-dependent meaning; a large Send-Q on a listener is normal, not a problem.

```sh
ss -tl
```

## Replace netstat -tulpn

Also asked as: ss equivalent of netstat -tulpn; modern netstat; ss -tulpn; listening ports with process

The direct replacement is `ss -tulpn` (add sudo for full process information). `-t` TCP, `-u` UDP, `-l` listening, `-p` processes, `-n` numeric. It prints the same information netstat used to, faster, and without walking `/proc`. People still type the netstat flags in the same order and wonder why the output looks different. The gotcha is the Process column format; it is not the old `pid/name` string.

```sh
sudo ss -tulpn
```

## Show extended socket details

Also asked as: ss -e; ss --extended; inode and cookie; uid of socket

`-e` (or `--extended`) adds the owning UID, the socket inode number, and the kernel socket cookie. Useful for correlating with `/proc` or for SELinux contexts when combined with `-Z`. People expect it to include process names and are disappointed; that is still `-p`. The gotcha is that the extra fields appear only when the kernel provides them; some socket types omit the cookie.

```sh
ss -te
```

## Show sockets in a network namespace

Also asked as: ss -N; ss --net; namespace sockets; ss inside netns

`-N name` (or `--net name`) switches ss into the named network namespace before listing. You must have permission to enter that namespace. The rest of the options work unchanged. People run ordinary `ss` on the host and conclude a container has no listeners. The gotcha is that the namespace must already exist; ss does not create it.

```sh
ss -tln -N myns
```

## Why is ss faster than netstat?

Also asked as: ss vs netstat speed; netlink vs /proc; why use ss; ss performance

ss asks the kernel for the socket table over netlink instead of reading and parsing every file under `/proc/net`. On a host with tens of thousands of connections the difference is seconds versus milliseconds. People still prefer netstat out of habit and then hit timeouts in monitoring scripts. The gotcha is that the speed advantage disappears if you also request process information (`-p`) for every socket; that step still walks `/proc`.

```sh
ss -tn
```

## Show all listening TCP and UDP with processes

Also asked as: ss -tulpn; common ss command; listening ports and PIDs; what ports are open

This is the combination most people actually type when they want “what is listening and who owns it”. Numeric output avoids slow DNS, and root is required for complete process data. The result is a short table of listeners. People omit `-n` and then script against service names that change with `/etc/services`. The gotcha is UDP listeners appear as UNCONN; that is correct and expected.

```sh
sudo ss -tulpn
```

## Show SSH connections

Also asked as: ss ssh; established ssh; connections on port 22; who is connected via ssh

Filter for TCP established sockets whose local or remote port is 22 (or the ssh service). Quote the parenthesised expression. The command lists the current SSH sessions. People forget the `or` and only see one direction. The gotcha is that after OpenSSH 9.8 the process name is often `sshd-session` rather than `sshd`, so grepping the Process column for the old name fails.

```sh
ss -tnp state established -- '( sport = :22 or dport = :22 )'
```
