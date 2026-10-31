# dig

`dig` asks a DNS server for records and prints the reply. It does not edit a zone, and it does not update `/etc/hosts`. With a name it sends one query and exits. The server it asks is the first nameserver in `/etc/resolv.conf`, unless you put `@server` on the command. BIND `dig` from bind9-dnsutils on Linux is assumed here. macOS ships a BIND-derived `dig` with the same common forms. BusyBox does not include this client. Exit status is 0 when a server sent a DNS reply, including `NXDOMAIN` and an empty `NOERROR`. It is non-zero when the arguments are illegal or no server answered. A received refusal is still a reply. A timeout is not.

Basic form: dig name

The name is the query. A type may follow it, as in `name MX`. `@server` chooses the server and may appear before or after the name. `+` turns a query option on, `+no` turns it off. Quote a name the shell would expand. A leading dash is a flag unless you put `--` before the name. With no type, `dig` asks for `A`. It does not append the search list unless you pass `+search`. That is the opposite of `nslookup`.

## Look up a name
Also asked as: dig a hostname; resolve a name; dns lookup; what ip does this name have; dig example
With one argument, `dig` asks for an `A` record and prints the query, the answer, the authority, and stats. The addresses are in the answer section, in the last column. The line `SERVER:` is the resolver you asked, not the owner of the name. A status of `NOERROR` means the server answered. It does not mean an address was present.

```sh
dig -- name
```

People read the first IP in the header as the answer. That is the server you queried. The answer lines have the name, a TTL, `IN`, `A`, and the address. A missing answer section with `NXDOMAIN` means the name does not exist. A missing answer with `NOERROR` means the name exists and has no `A` record. `dig` does not read `/etc/hosts`. An address that only exists in the hosts file will not appear.

## Ask a specific DNS server
Also asked as: dig @server; query a nameserver; dig against 8.8.8.8; ask this dns; dig at server
`@server` is the server to query. It can be an address or a name. `dig` uses that server and not `/etc/resolv.conf`. This is how you compare resolvers. An IPv6 server address does not need brackets. The type still follows the name. `@server` is not a zone. It is who you ask.

```sh
dig @server -- name
```

People put the server where the name belongs and look up the wrong thing. `@` marks the server. A server that is not recursive returns `REFUSED` for a name it does not host. That is not `NXDOMAIN`. Ask a recursive resolver for a general lookup, or ask an authoritative server only for names in its zone. If `@server` is a name, `dig` has to resolve it first, using the system resolver. Pass an address when that lookup is the thing you are debugging.

## Query a record type
Also asked as: dig MX; dig TXT; dig record type; dig name type; query type
The type is a positional argument after the name. `A`, `AAAA`, `MX`, `TXT`, `NS`, `SOA`, `CNAME`, and `PTR` are the usual ones. One invocation asks for one type. Other types at that name are not listed. The default type is `A`. A type before the name is parsed as the name, and the query is wrong.

```sh
dig -- name TXT
```

People ask for `A`, get `NOERROR` with no answer, and conclude the name is missing. The name can still have `AAAA` or `MX`. `NXDOMAIN` is the status that means absent. An empty answer means the type is absent. `ANY` is not a shortcut. Many servers refuse it. Ask for the type you mean. The type is case-insensitive. It is not a flag, so `dig -MX` is a usage error.

## Print only the answers
Also asked as: dig +short; short output; only the address; dig script output; no comments
`+short` prints the answer data and nothing else. An `A` query prints addresses, one per line. An `MX` query prints the preference and the hostname. An empty answer prints nothing and still exits 0 if the server replied. `+short` hides the status, so `NXDOMAIN` and an empty `NOERROR` look the same.

```sh
dig +short -- name
```

People script `+short` and treat empty output as a failure. Check the status, or drop `+short` when you need `NXDOMAIN`. A `CNAME` makes `+short` print the target and then the address if the server included both. You can get two lines for one name. Quote the capture. `+short` is not `+noall +answer`. The second form keeps the full answer lines, including TTL and type, and is easier to parse when you still want the status from a second run.

## Show only the answer section
Also asked as: dig +noall +answer; answer section only; hide stats; dig comments off; clean dig output
`+noall` turns every section off. `+answer` turns the answer section back on. You get the records without the header and without the stats. The status line is in the header, so this form hides `NXDOMAIN` too. Add `+comments` if you want the header flags and the status without the question echo.

```sh
dig +noall +answer -- name
```

People pass `+answer` alone and still see the whole default dump, because `+answer` only turns a section on. `+noall` is what clears the rest. A `CNAME` and the address are both answer lines. Authority nameservers are not in this section. Those need `+authority`. This form is the one to read. `+short` is the one to capture a bare address. Use `+short` when a later command wants only the address, and this form when you still want the type and the TTL.

## Reverse lookup
Also asked as: dig -x; ptr record; reverse dns; ip to name; dig an address
`-x` builds the reverse name and queries `PTR`. An IPv4 address becomes an `in-addr.arpa` name. An IPv6 address becomes an `ip6.arpa` name. The answer is a hostname if someone published one. Many addresses have no `PTR`. That is an empty answer or `NXDOMAIN` in the reverse zone, not a failure of the forward name.

```sh
dig -x -- address
```

People reverse an address, get nothing, and conclude the forward name is wrong. Forward and reverse are separate data. A `PTR` can point at a name that does not point back. `dig` does not check that round trip. A private address only resolves if the resolver you asked serves the private reverse zone. Public resolvers do not. `-x` already sets the type. You do not also pass `PTR`.

## Trace the delegation
Also asked as: dig +trace; follow delegation; from the root; which nameservers; trace dns
`+trace` starts at the root and follows referrals until it gets an answer. It prints each step. It does not use your recursive resolver for the final answer. It asks the authoritative servers it was referred to. This shows which zone is supposed to own the name. It needs the root servers to answer, so a network that only allows the corporate resolver will fail the trace.

```sh
dig +trace -- name
```

People read the last address in a trace as the only address. The last step is what those authoritative servers answered. A cache in front of you can still disagree. A trace that stops with `REFUSED` found a server that will not answer that query. A trace does not follow a `CNAME` into another zone unless that referral is in the trace. Run a second trace on the target if the chain leaves the zone.

## Ask without recursion
Also asked as: dig +norecurse; rd flag; authoritative only; do not recurse; dig +rec
`+norecurse` clears the recursion-desired flag. The server answers from what it already has, or it refers you elsewhere. This is how you ask an authoritative server without it doing your lookup. `+recurse` is the default. A recursive resolver asked with `+norecurse` often returns the root referral instead of the address. That referral is not the answer.

```sh
dig @server +norecurse -- name
```

People query `8.8.8.8` with `+norecurse` and think the name is broken because they got root nameservers. They told a recursive server not to recurse. Use `+norecurse` on a server from the `NS` set. An authoritative answer has the `aa` flag. A referral has no answer section and nameservers in the authority section. `dig` does not follow that referral unless you used `+trace`.

## Look up mail servers
Also asked as: dig MX; mail exchanger; where does mail go; mx preference; dig mail records
An `MX` query returns a preference and a hostname. A lower preference is tried first. The hostname is not an address. Look up the `A` or `AAAA` of that hostname next. A name with no `MX` and an `A` record is a different mail rule, used by some senders. `dig` does not apply that rule. It only prints the records you asked for.

```sh
dig -- name MX
```

People expect an IP in the MX line. The record stores a hostname. A `CNAME` at the name means the mail records live on the target, not on the alias. Read the canonical name before you compare preferences. A zero preference is legal and wins. `dig` does not connect to port 25. A successful MX lookup can still be a host that refuses mail.

## Look up text records
Also asked as: dig TXT; spf record; dkim txt; txt strings; dig text
A `TXT` query returns one or more strings. A long SPF or DKIM value may be several quoted strings in one record. They are concatenated with no extra space. `+short` prints the quoted strings. `+multiline` wraps long records so they fit. The quotes are DNS string syntax, not part of the SPF text.

```sh
dig -- name TXT
```

People copy the quotes into an SPF check and fail the comparison. The quotes are delimiters. People also expect one `TXT` record. A name can have SPF, a verification token, and something else, all as `TXT`. Read every answer line. `dig` does not pick the SPF one. A `CNAME` means the strings live on the target. Query the target if the answer is only an alias.

## See the nameservers and the serial
Also asked as: dig NS; dig SOA; zone serial; delegation; who serves this zone
`NS` returns the nameserver hostnames of a zone. `SOA` returns the primary name, the serial, and the timers. Ask a recursive resolver for the cached view. Ask one of those nameservers, with `@server`, for what that server itself has. The serial is the number to compare. A larger serial is newer only if the operator increments it.

```sh
dig -- name SOA
```

People treat `NS` hostnames as public resolvers. They are the delegation. Querying one of them for a name outside its zone often returns `REFUSED`. The serial in a recursive answer can lag the authoritative serial by the SOA minimum or the record TTL. Compare both before you decide an edit is missing. `dig` cannot reload a zone. It can only show the serial the server returned.

## Force TCP
Also asked as: dig +tcp; dns over tcp; truncated udp; dig +vc; tc flag
`+tcp` sends the query over TCP. The default is UDP. A UDP reply with the truncated flag is supposed to be retried over TCP, and `dig` does that unless you disabled it. Force TCP when UDP is blocked or the answer is large. `+notcp` stays on UDP and shows the truncated flag instead of retrying.

```sh
dig +tcp -- name
```

People see a short UDP answer and miss the `tc` flag in the header. The answer is incomplete. `+tcp` asks for the full reply. A server that allows UDP and refuses TCP looks like a timeout only on the TCP query. `dig` does not fall back if you forced TCP and the port is closed. A zone transfer is `AXFR`, not `+tcp`. `dig` can send `AXFR`, and many servers will refuse it. That refusal is the server's policy.

## Set the timeout and the retries
Also asked as: dig +time; dig +tries; connection timed out; no servers could be reached; dns timeout
`+time=5` waits five seconds for a reply. `+tries=2` sends at most two queries. The default is a few seconds and a few tries, which feels like a hang. A timeout means this server did not answer. It does not mean the name is missing. `NXDOMAIN` is the missing name. `+time` takes seconds. It is not milliseconds.

```sh
dig +time=3 +tries=1 @server -- name
```

People raise the timeout and hide a blackholed server. One try and a short timeout is the probe. A refused packet comes back immediately with status `REFUSED`. That is not a timeout. A timeout has no status line, because there was no DNS reply, and the exit status is non-zero. `no servers could be reached` is the usual text. Pass `@server` so you know which address failed.

## Query over IPv6
Also asked as: dig -6; dig AAAA; ipv6 lookup; dig -4; force ipv4
`-6` sends the query over IPv6. `-4` forces IPv4. The record type `AAAA` is a different switch. It asks for IPv6 addresses of the name, over whatever transport you used. A name can have `A` and no `AAAA`. That is an empty `AAAA` answer, not `NXDOMAIN`, if the name exists. `-6` fails if this host has no IPv6 route to the server.

```sh
dig -t AAAA -- name
```

```sh
dig -6 @server -- name
```

Use the first to ask what addresses the name has. Use the second to ask a server over IPv6. People mix them and conclude the name is broken when the local host cannot reach the server on IPv6. The transport error is a timeout. The empty `AAAA` is a reply. `-t AAAA` is the type. The positional type `AAAA` is the same query. An address in the answer is not a promise that the service listens on it.

## Use a nonstandard port
Also asked as: dig -p; dns port; port 5353; nonstandard port; dig port
`-p` sets the server port. The default is 53. The port belongs to the server, not to the name. A wrong port looks like a timeout, not like `NXDOMAIN`. This does not select DNS over HTTPS. `dig` speaks DNS on that port.

```sh
dig -p 53 @server -- name
```

People write `@10.0.0.1:5353` and the server argument fails or is taken as a name. The port is `-p`. A server that answers on 53 and not on the port you chose never sees the packet. The error is reachability. `-p` applies to `@server` and to the system resolver alike. It does not change the record type.

## Turn the search list on
Also asked as: dig +search; resolv.conf search; short name; ndots; dig domain suffix
By default `dig` queries the name you typed and does not append the search list. `+search` uses the search list and `ndots` from `/etc/resolv.conf`, which is what the C library does. A short name can then become `name.example.com`. The printed question shows the name that was sent. A trailing dot keeps the name absolute even with `+search`.

```sh
dig +search -- name
```

People compare `dig host` with a browser and get different answers. The browser searched. `dig` did not. `+search` is the match. A script should not pass `+search` for a name the user already qualified, or a suffix can capture it. `nslookup` searches by default. `dig` does not. That single difference explains a lot of "it works in one tool" reports.

## Read names from a file
Also asked as: dig -f; batch file; dig a list; many names; dig from stdin
`-f file` reads names from a file, one query per line. `-f -` reads standard input. Each line is a `dig` argument list, so a line may contain a type. This is the batch form. One failing line does not stop the file. The output is every reply in order, which is a lot unless you set `+short`.

```sh
dig +short -f file
```

People pipe names into `dig` with no `-f` and `dig` ignores the pipe, then waits on the terminal or queries nothing useful. The file flag is what reads them. A line that starts with `;` is a comment. A blank line is skipped. `-f` does not parallelize. A long file is a long run. A name with a space needs the quoting rules of a `dig` command line, because the line is parsed that way.

## See DNSSEC status
Also asked as: dig +dnssec; dnssec; ad flag; do bit; rrsig; is the answer validated
`+dnssec` sets the DO bit so the server may return `RRSIG` records. The `ad` flag in the header means the resolver validated the answer. `dig` itself does not validate. It prints what the server sent. `+multiline` makes the `RRSIG` and `DNSKEY` records readable. A missing `ad` flag does not mean the data is false. It means this resolver did not mark it validated.

```sh
dig +dnssec +multiline -- name
```

People query an authoritative server and expect `ad`. Authoritative servers usually do not set `ad`. A validating recursive resolver does, if it could build a chain. A `SERVFAIL` on a signed zone often means validation failed at that resolver. Ask another resolver before you blame the zone. `+dnssec` does not publish a key. It only asks for the signatures.

## Compare two resolvers
Also asked as: dig different servers; split dns; public versus internal; compare dns answers; which resolver
Run the same name and type against two `@server` values and compare the status and the answer section. Different addresses, or a different `NXDOMAIN`, mean split DNS, a stale cache, or a server that is not recursive. The `SERVER:` line confirms who answered. A name that resolves only on the internal server is invisible to a resolver off that network.

```sh
dig @server -- name
```

People compare one run on the VPN with one run off it and blame the name. The resolver changed with the network. Pass `@server` so both queries are deliberate. A cached `NXDOMAIN` can stick for the SOA minimum. Asking another server is the test. `dig` does not flush the server's cache. `+trace` shows the delegation, which is a different question from "what does this resolver say."

## Read the status and the flags
Also asked as: dig status; NOERROR; NXDOMAIN; SERVFAIL; REFUSED; aa flag; header flags
The header line has `status`. `NOERROR` means the server answered the question. The answer may still be empty. `NXDOMAIN` means the name does not exist. `SERVFAIL` means the server could not get an answer. `REFUSED` means it declined, often because recursion is off or an ACL blocked you. `aa` means authoritative. `rd` means you asked for recursion. `ra` means the server is willing to recurse.

```sh
dig -- name
```

People treat every failure as `NXDOMAIN` and delete a record that is fine. `SERVFAIL` is the server or its upstream. `REFUSED` is policy. Neither says the name is absent. Flags are in the header, which `+short` hides. Drop `+short` when you need the status. A reply with `NOERROR` and an empty answer is a successful exchange. The exit status is 0. The type you asked for is not there.

## What a failed dig means
Also asked as: dig exit code; dig exit status; dig failed; no servers could be reached; usage error
Exit 0 means a DNS reply arrived, including `NXDOMAIN`, `SERVFAIL`, and `REFUSED`. Exit non-zero means no reply arrived, or the command line was illegal. A timeout is the non-zero case. A missing name is not. A script that treats non-zero as `NXDOMAIN` is wrong. A script that treats exit 0 as "an address was printed" is also wrong. Read the status line, or test the answer.

```sh
dig +short -- name
```

People capture `+short` and check `$?`. Both an address and `NXDOMAIN` can be 0, and the capture is empty in the second case. Use the full output for the status, or run a second query without `+short` when the branch matters. A usage error prints a short message and does not send a packet. A firewall drop prints `no servers could be reached` after the tries. Those are the two non-zero shapes. The reply statuses are the zero shape.
