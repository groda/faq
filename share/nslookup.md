# nslookup

`nslookup` asks a DNS server for records and prints the reply. It does not edit a zone, and it does not update `/etc/hosts`. With a name it queries and exits. With no name it starts an interactive session and waits. The server it asks is the first nameserver in `/etc/resolv.conf`, unless you pass a server as the second argument. BIND `nslookup` from bind9-dnsutils on Linux is assumed here. macOS ships a BIND-derived `nslookup` with the same common forms. BusyBox `nslookup` can query a name and a server, and it has no interactive `set` session worth relying on. A non-interactive BIND run exits non-zero when the query fails or no server answers. An interactive session exits 0 when you leave it, even if a query inside it failed. `NXDOMAIN` is a reply, not a crash of the client.

Basic form: nslookup name

The name is the thing you want looked up. A second positional argument is the server to ask, not a record type. A type is `-type=MX` or `-query=MX`. Quote a name that contains spaces or shell metacharacters. A leading dash on a name is a flag unless you put `--` before it. The client appends the search list from `resolv.conf` when the name has fewer dots than `ndots`. That is why a short name can return an answer you did not type.

## Look up a name
Also asked as: nslookup a hostname; resolve a name; dns lookup; what ip does this name have; nslookup example
With one argument, `nslookup` asks the system resolver's nameserver for the name and prints the server it used, then the answer. An `A` record is the IPv4 address. An `AAAA` record is printed too if the reply has one. The line `Server:` is the resolver you asked, not the owner of the name. `Non-authoritative answer` means the server recursed or used cache. It does not mean the answer is wrong.

```sh
nslookup -- name
```

People read `Server:` as the machine the name points at. The addresses under `Name:` are the answer. A short name may be searched under the domain in `resolv.conf`, so `nslookup host` can return `host.example.com` without you typing the suffix. A trailing dot pins the name as absolute. `nslookup` does not consult `/etc/hosts` on a typical BIND build. The hosts file is the C library. This client talks to DNS.

## Ask a specific DNS server
Also asked as: nslookup name server; query a nameserver; nslookup against 8.8.8.8; second argument is the server; ask this dns
The second positional argument is the server to query. It can be an address or a name. `nslookup` uses that server and not `/etc/resolv.conf`. This is how you compare resolvers. An IPv6 server address needs brackets so the port colon is not ambiguous, as in `[2001:db8::1]`. The type flags still work in front of the name.

```sh
nslookup -- name server
```

People put the server first and look up the wrong thing. The name is first. The server is second. People also pass `-type=MX` as the second argument. That is not a server. The type is a flag. A server that refuses recursion returns a refusal for a name it does not host. That is not `NXDOMAIN`. Asking an authoritative server for a name in another zone often fails for that reason. Use a recursive resolver, or ask the authoritative server only for names in its zone.

## Look up MX records
Also asked as: nslookup -type=MX; mail exchanger; nslookup MX; -query=MX; where does mail go
`-type=MX` asks for mail exchanger records. `-query=MX` is the same flag. The reply lists preference numbers and hostnames. A lower preference is tried first. The hostnames are not expanded to addresses unless you look those up too. The default query type is `A`, so a plain `nslookup` does not show mail servers.

```sh
nslookup -type=MX -- name
```

People expect an IP address in the MX answer. The record stores a hostname. A missing `A` or `AAAA` on that hostname is a second failure, invisible to this query. Some servers refuse `ANY` and still answer `MX`. Use the type you mean. A CNAME at the name can make the answer point at another name's MX set. Read the canonical name in the reply before you blame the preference numbers.

## Look up a record type
Also asked as: nslookup -type; nslookup TXT; nslookup NS; nslookup SOA; query type
`-type=` selects the record type. Common types are `A`, `AAAA`, `CNAME`, `TXT`, `NS`, `SOA`, and `PTR`. One query asks for one type. A name can have other types that this reply will not list. `-query=` is the same switch. The type applies to this non-interactive command. It does not change the next command in the shell.

```sh
nslookup -type=TXT -- name
```

People run `nslookup -type=A` and think a missing answer means the name does not exist. The name can still have `AAAA` or `CNAME`. `NXDOMAIN` means the name is absent. An empty answer with `NOERROR` means the name exists and has no records of that type. Those are different. `TXT` answers are strings. A long SPF record may be split across several quoted strings in one record. Join them before you compare.

## Reverse lookup
Also asked as: nslookup an ip; ptr record; reverse dns; ip to name; nslookup -type=PTR
A bare address makes `nslookup` ask the reverse zone. It builds the `in-addr.arpa` or `ip6.arpa` name and queries `PTR`. The answer is a hostname if someone published one. Many addresses have no `PTR`. That is an empty answer or `NXDOMAIN` in the reverse zone, not a failure of the forward name. `-type=PTR` with the reversed name is the same query written out.

```sh
nslookup -- address
```

People reverse an address, get nothing, and conclude the forward name is wrong. Forward and reverse are separate data. A `PTR` can also point at a name that does not point back at the address. `nslookup` does not check that round trip. A private address only resolves if the resolver you asked knows the private reverse zone. Public resolvers do not.

## See the query and the reply
Also asked as: nslookup debug; set debug; nslookup -debug; verbose dns; see the packet
`-debug` prints the query and the reply sections, including flags and the authority section. It is the non-interactive switch. In the interactive session the same idea is `set debug`. The extra text goes to standard error mixed with the answer. It does not change what was asked. BusyBox `nslookup` often has no debug dump.

```sh
nslookup -debug -- name
```

People turn on debug and try to read it as a second answer. The answer section is still the answer. The authority section lists nameservers for the zone, which are not extra addresses for the name. A timeout in debug means this server did not answer. It does not mean every server in the world is down. `-debug` is not `-type`. A query type still has to be set if you wanted `MX`.

## Use the interactive session
Also asked as: nslookup interactive; nslookup with no arguments; set type; server command; exit nslookup
With no arguments, `nslookup` reads commands from the terminal. `name` looks up a name. `server address` changes the server. `set type=MX` changes the type for later lookups. `exit` leaves. A single query on the command line does not enter this mode. BusyBox has no such session. Do not pipe a script into it and expect the BIND command language.

```sh
nslookup
```

People type `-type=MX` at the interactive prompt. That is shell syntax. Inside the session the command is `set type=MX`. People also type a name and wait because the server never answers. `Ctrl-C` stops the query. `exit` stops the program. An interactive failure still leaves you at the prompt. The exit status of the process is from when you quit, not from the failed lookup.

## Set the type in a session
Also asked as: set type=MX; set q=A; interactive query type; nslookup set query; change record type
Inside the interactive session, `set type=MX` makes the following names ask for that type. `set q=MX` is the same setting. It stays until you change it or exit. It does not affect a new `nslookup` you start later. `set type=A` returns to addresses. A name typed with no set command uses the current type, which starts as `A`.

```sh
nslookup
```

The one-line form is still the non-interactive flag, `nslookup -type=MX -- name`. Use that in a script. Use `set type` only at the prompt. People set the type, then pass a server as the next word, and the server name is looked up as an MX. The server change is the command `server`. The type is not a positional argument in the session.

## Follow a CNAME
Also asked as: nslookup cname; canonical name; alias; nslookup shows two names; cname and address
If the name is a `CNAME`, `nslookup` prints the canonical name and then the address records the server included. The first name is the alias. The second is the target. A `CNAME` and other data at the same name are illegal in DNS, so you should not expect `MX` on the alias itself. The mail records live on the target.

```sh
nslookup -- name
```

People configure a client with the alias and then look up only the target. The alias is a real name. It just is not the address. A chain longer than the server bothered to follow shows the `CNAME` and no address. Ask again for the target, or ask a recursive resolver. `nslookup` does not invent the target's records if the reply omitted them.

## See which nameservers a zone publishes
Also asked as: nslookup -type=NS; nameserver records; delegation; soa record; who serves this zone
`-type=NS` asks for the nameserver names of a zone. `-type=SOA` asks for the start of authority, including the primary name and the serial. Asking a recursive resolver returns whatever it cached or recursed. Asking one of those nameservers directly, as the second argument, shows what that server itself answers. Those can differ while a transfer is behind.

```sh
nslookup -type=NS -- name
```

People treat the `NS` hostnames as the only servers that can answer a recursive query. They are the delegation, not public resolvers. Querying an `NS` host for a name outside its zone often gets a refusal. The serial in the `SOA` is the number to compare between servers. A larger serial is newer only if the operator increments it. `nslookup` does not check that the serial increased.

## Time out and try another server
Also asked as: nslookup timeout; connection timed out; no servers could be reached; retry; DNS not answering
`connection timed out; no servers could be reached` means the servers `nslookup` tried did not answer in time. It does not mean the name is missing. `NXDOMAIN` is the missing name. A timeout is reachability. The client tries the nameservers from `resolv.conf` unless you passed a server. A firewall that drops UDP 53 looks like this.

```sh
nslookup -- name
```

People raise the timeout and hide a blackholed server. Fix the server list, or pass a server you know answers. In the interactive session, `set timeout=5` changes the wait. A non-interactive BIND client also accepts `-timeout=5` on many builds. A refused packet is a different line, `connection refused` or a `REFUSED` status, and means the server heard you and said no. Do not treat that as `NXDOMAIN`.

## Read NXDOMAIN and an empty answer
Also asked as: nxdomain; non-existent domain; no answer; NOERROR empty; name does not exist; server can't find
`NXDOMAIN` means the server says the name does not exist. `server can't find name: NXDOMAIN` is that reply. An answer that says `No answer` with no `NXDOMAIN` means the name exists and has no record of the type you asked for. A search-list append can turn a real name into a longer name that is `NXDOMAIN`, or the reverse. The printed name is the one to read.

```sh
nslookup -- name
```

People see `NXDOMAIN` from a recursive resolver and assume the zone is empty. That resolver may be lying, split, or stale. Ask an authoritative server from the `NS` set before you delete anything. A search suffix is the other trap. `nslookup host` may query `host.example.com` and report `NXDOMAIN` for the suffixed name. A trailing dot queries the name you typed.

## Force an absolute name
Also asked as: trailing dot; absolute dns name; stop search list; ndots; nslookup fqdn
A name with a trailing dot is absolute. `nslookup` does not append the search list. A name without a dot may be completed with the domain from `resolv.conf` before it is tried as typed, depending on `ndots`. The reply prints the name that was actually queried when it differs. Scripts that take a short name should add the dot if they mean the root.

```sh
nslookup -- name
```

People copy a name from a zone file, including the trailing dot, into a browser, and they copy a browser name into `nslookup` and get a search-list answer. The dot is DNS syntax, not a URL. A trailing dot on the server argument is also a name, and it has to resolve before it can be asked. An address as the server does not use the search list.

## Query over TCP
Also asked as: nslookup TCP; set vc; truncated udp; -vc; force tcp dns
DNS usually uses UDP. A truncated reply is supposed to be retried over TCP. `set vc` in the interactive session asks over TCP from the start. Some BIND builds accept `-vc` on the command line for the same thing. TCP is what you want when UDP is blocked or the answer is large. BusyBox often cannot force TCP.

```sh
nslookup -vc -- name
```

People see a short answer and do not notice truncation. Debug output shows the truncated flag. A server that allows UDP and refuses TCP looks like a timeout only on the TCP retry. `nslookup` does not fall back to a different protocol if you forced TCP and the port is closed. Use UDP, the default, unless you have a reason. A zone transfer is not this switch. `nslookup` will not AXFR a zone for you in a useful way.

## Change the port
Also asked as: nslookup port; dns on another port; set port; nonstandard port; nslookup -port
The default port is 53. In the interactive session, `set port=5353` points later queries at that port on the current server. A non-interactive BIND client often accepts `-port=5353`. The port is the DNS server's port, not an HTTP port. A server argument does not take a colon and a port the way a URL does, except the bracket form for IPv6.

```sh
nslookup -port=53 -- name server
```

People write `nslookup name 10.0.0.1:5353` and the server argument fails to parse. Set the port with the flag or `set port`. A wrong port looks like a timeout, not like `NXDOMAIN`. The server you meant never saw the packet. This does not select DNS over HTTPS. `nslookup` speaks DNS, TCP or UDP, not a web API.

## Compare two resolvers
Also asked as: nslookup different servers; split dns; public versus internal; compare dns answers; which resolver
Run the same name against two servers and compare the answer sections. The second argument is the server. Different addresses, or a different `NXDOMAIN`, mean split DNS, a stale cache, or a server that is not recursive. The `Server:` line confirms which one answered. A name that resolves only on the internal server is invisible to a laptop that is off that network.

```sh
nslookup -- name server
```

People compare one run on the VPN with one run off it and blame the name. The resolver changed with the network. Pass the server explicitly so both queries are deliberate. A cached `NXDOMAIN` can stick for the SOA minimum or the resolver's negative TTL. Asking another server is the test. `nslookup` does not flush the server's cache. It only asks.

## Look up a name in a zone you are editing
Also asked as: nslookup after editing a zone; serial did not change; cached answer; not seeing the new record; stale dns
A new record that does not show up is usually the wrong server or a cache. Ask an authoritative server from the `NS` set, and check `-type=SOA` for the serial you just published. A recursive resolver can keep the old answer until the TTL. `nslookup` prints the TTL in debug output. It does not show it in the short answer.

```sh
nslookup -type=SOA -- name server
```

People query the recursive resolver, edit the zone, and query again before the serial moved. If the serial is unchanged, the edit is not what that server is serving. If the serial moved and the recursive answer is old, wait for the TTL or ask the authoritative server. Reloading the zone is a server action. `nslookup` cannot reload it.

## Use a search name on purpose
Also asked as: resolv.conf search; dns suffix; nslookup short name; domain search list; why did it add a domain
The search list in `/etc/resolv.conf` is appended to names with fewer dots than `ndots`. `nslookup` prints the resulting name. That is how `host` becomes `host.example.com`. A script that passes a user string can be redirected onto a name in the search domain. A trailing dot disables the search.

```sh
nslookup -- name
```

People file a bug against the zone because a short name resolved. The zone may not contain that short name at the root. The resolver completed it. Interactive `set nosearch` turns the list off for the session. A non-interactive query has no standard short flag for that on every build. Prefer the trailing dot in a script. The search list is a client behavior, not a record in the zone.

## What the status codes mean
Also asked as: nslookup status; REFUSED; SERVFAIL; NOERROR; dns rcode; server failed
`NOERROR` means the server answered the question. The answer section may still be empty. `NXDOMAIN` means the name does not exist. `SERVFAIL` means the server could not get an answer. `REFUSED` means the server declined to answer, often because recursion is off or an ACL blocked you. The short output spells these out in a sentence. Debug prints the rcode.

```sh
nslookup -debug -- name
```

People treat every failure as `NXDOMAIN` and delete a record that is fine. `SERVFAIL` is the server or its upstream. `REFUSED` is policy. Neither says the name is absent. A status of success from the client, exit 0, means the exchange finished. It can still be an empty `NOERROR`. Read the reply. Exit non-zero means the client got no usable exchange or the name was not found, depending on the build. Do not script the text alone.

## Script a lookup
Also asked as: nslookup in a script; parse nslookup; non-interactive; nslookup versus dig; exit code in a script
A script should pass the name and the type on the command line and not start the interactive session. `nslookup -type=A -- name` is the non-interactive form. The output is text for a person. Columns shift when a `CNAME` is present. BIND's exit status is non-zero on `NXDOMAIN` and on no servers. It is a weak API. `dig +short` is the tool when you need a parseable answer.

```sh
nslookup -type=A -- name
```

People pipe names into `nslookup` with no arguments and the process sits in interactive mode, reading the names as commands. Some of those commands change the server. Pass one name per invocation, or use `dig`. Parsing `Address:` also catches the server address at the top. The answer addresses come after the name. A CNAME target can be the only address line. Check the status before you trust a captured address.

## IPv6 addresses
Also asked as: nslookup AAAA; ipv6 lookup; nslookup ipv6 server; A versus AAAA; bracket server
`-type=AAAA` asks for IPv6 addresses. A default lookup may print them beside `A` if the reply includes them, and it may not ask for them alone. To query an IPv6 server, put the address in brackets as the second argument. A link-local address needs a zone index the stack understands. Many servers have no `AAAA`. That is an empty answer, not `NXDOMAIN`, if the name exists.

```sh
nslookup -type=AAAA -- name
```

People look up `A`, get an address, and conclude the service has no IPv6. They did not ask. People paste an IPv6 server without brackets and the colon splits the argument. The client then complains it cannot find the server. An `AAAA` that points at an unreachable address still looks like a successful lookup. `nslookup` does not connect to the service. It only resolves the name.

## What a failed nslookup means
Also asked as: nslookup exit status; nslookup failed; can't find server; no response; nslookup error
Exit 0 means the client finished and, on BIND, got a positive answer. Exit non-zero means no server answered, the name was not found, or the arguments were illegal. The sentence on stderr is the distinction. `NXDOMAIN` is a DNS answer. `no servers could be reached` is the network. A usage error is a bad flag, and the query was not sent.

```sh
nslookup -- name
```

People test the output for the word `Address` and treat a timeout's empty stdout as `NXDOMAIN`. Check the status and the error line. A refused query can exit non-zero and still have printed `Server:`. The packet came back. The answer did not. Interactive mode is the wrong place to read `$?`, because you exit the session later. Use the non-interactive form in a script, and prefer `dig` if the status has to be reliable.