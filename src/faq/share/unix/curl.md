# curl

`curl` transfers data from a URL. It does not render a page, and it does not run JavaScript. With a URL it writes the body to standard output and the progress meter to standard error. OpenSSL-backed curl 7.88 on Linux is assumed here. macOS ships curl as well, often an older one, with the same common flags. BusyBox has a thinner wget more often than curl. Exit status is 0 when the transfer finished. It is non-zero on a DNS failure, a refused connection, a timeout, or a TLS error. An HTTP 404 is still exit 0, unless you pass `-f`. The status is about the transfer, not about the document.

Basic form: curl URL

The URL is the operand. Flags come before it. Quote the URL. A `?` or an `&` in it is shell syntax if you do not quote. `-o` names the output file. `-O` uses the remote name. `--` before a URL that starts with a dash stops it looking like a flag. Several URLs are several transfers. A missing scheme gets `http://` only if you configured that. Write the scheme.

## Fetch a page
Also asked as: curl a url; download a page; curl http; get a webpage; curl to stdout
`curl URL` writes the body to stdout and a progress meter to stderr. The body is the document. Headers are not included unless you pass `-i` or `-D`. A redirect is not followed unless you pass `-L`. The meter vanishes if stdout is not a terminal, or if you pass `-s`. Quote the URL.

```sh
curl -- 'URL'
```

People paste a URL with `&` and the shell backgrounds the command. The quotes are required. People also expect curl to show the status code. It does not, by default. `-w '%{http_code}\n'` prints it. A 200 and a 404 both land on stdout and both exit 0 without `-f`. Look at the code if you care. `-v` shows the request and the response headers on stderr. It is the first debug step.

## Save to a file
Also asked as: curl -o; curl -O; save the download; output file; remote name
`-o file` writes the body to that path. `-O` uses the last part of the URL as the name, in the current directory. `-o -` is stdout, which is the default. A second `-o` pairs with a second URL. The progress meter still goes to stderr. A directory as `-o` fails. The parent must exist.

```sh
curl -o file -- 'URL'
```

```sh
curl -O -- 'URL'
```

Use `-o` when you want a chosen name. Use `-O` when the URL already ends in the filename. People `-O` a URL that ends in a query string and get a file named with a `?` in it. Name it yourself. People also `-o` the same file for two URLs and the second overwrites the first. One `-o` per URL. An existing file is truncated. `-C -` resumes if the server allows it. A failed transfer can leave a partial file. Check the status before you use it.

## Follow redirects
Also asked as: curl -L; follow redirect; location header; curl redirect; 301 302
`-L` follows a `Location` header. The default is not to. A site that answers 301 or 302 then gives you the redirect page's body, which is often empty or a short note. `-L` fetches the next URL, up to a limit. `--max-redirs` changes the limit. Credentials are not sent to a different host unless you pass `--location-trusted`.

```sh
curl -L -- 'URL'
```

People curl a shortened link and get an empty file. They did not follow. People also follow a redirect to a file URL or a different scheme and are surprised. `-L` follows what the server said. `--proto-default` and `--proto` can restrict schemes. A redirect loop hits the limit and fails. The status is non-zero. `-v` shows each hop. A 303 after a POST becomes a GET on the next hop. That is the HTTP rule curl implements.

## Fail on an HTTP error
Also asked as: curl -f; curl --fail; exit code on 404; fail on http error; curl -fL
`-f` makes an HTTP 400 or above count as failure. The body is discarded. The exit status is 22. Without `-f`, a 404 is a successful transfer of an error page. `--fail-with-body` fails and still writes the body, on newer curl. `-f` does not change connection errors. Those already fail.

```sh
curl -fL -- 'URL'
```

People test `$?` and treat a 404 as success, because they forgot `-f`. The file then contains the error page. People also pass `-f` and lose the body, so they cannot see the server's message. Drop `-f` when you need the page, or use `--fail-with-body` if your curl has it. A 200 with a wrong document is still success. `-f` only watches the status code. It does not validate the content.

## Be quiet
Also asked as: curl -s; silent; hide the progress; curl -sS; no meter
`-s` hides the progress meter and most error messages. `-S` with `-s` brings the error messages back. `-sS` is the usual pair in a script. You want no meter, and you still want to see a failure. `-s` alone swallows both. The body is unchanged.

```sh
curl -sS -- 'URL'
```

People pass `-s` and a failed transfer prints nothing, exits non-zero, and they cannot see why. Add `-S`. People also expect `-s` to hide the body. It does not. Redirect the body to a file if you do not want it. `-s` does not imply `-f`. A quiet 404 still looks like a successful empty-ish run. Combine `-sS` with `-f` in a script that must notice HTTP errors.

## Send a header
Also asked as: curl -H; custom header; Authorization header; Accept header; curl header
`-H 'Name: value'` adds a header. Repeat the flag for several. A header you set replaces curl's default of the same name. `-H 'Accept:'` with an empty value removes a header. Quote the argument. A header with a space needs the quotes. `-D file` writes the response headers to a file. `-i` includes them in the output, ahead of the body.

```sh
curl -H 'Accept: application/json' -- 'URL'
```

People put the header after the URL and curl reads it as a second URL. Flags come first. People also `-i` and then parse the body, and the headers are in the way. Use `-D` for the headers and `-o` for the body if you need them apart. A typo in the header name is sent as typed. The server ignores it. `-v` shows the exact request. Authorization belongs in `-H` or in `-u`. It does not belong in the URL on a shared machine, because it lands in the history and in the process list.

## POST form data
Also asked as: curl -d; curl --data; POST a form; curl -X POST; send data
`-d 'key=value'` sends a POST with `Content-Type: application/x-www-form-urlencoded`. Repeat `-d` or join with `&`. `-X POST` is not required. `-d` already implies POST. `-d @file` reads the body from a file. `--data-urlencode` encodes the value. A GET with a query string is `-G` plus `-d`, which moves the data into the URL.

```sh
curl -d 'pattern' -- 'URL'
```

People pass `-X POST` with no `-d` and send an empty POST. The method flag does not provide a body. People also `-d` a JSON string and forget the content type. `-d` sets the form type. For JSON, add `-H 'Content-Type: application/json'`. A `@` in the data starts a file read. `--data-raw` sends a literal `@`. Quote the payload. An unquoted `&` is a shell background.

## Upload a file
Also asked as: curl -T; curl --upload-file; PUT a file; curl -F; multipart upload
`-T file` uploads the file with PUT to the URL. `-F 'field=@file'` sends a multipart form upload, which is what browsers do for a file field. They are different. `-T` is the raw upload. `-F` is the form. The `@` means "read this path." A field without `@` is a plain value.

```sh
curl -F 'field=@file' -- 'URL'
```

```sh
curl -T file -- 'URL'
```

Use the first for a form a site expects. Use the second for a PUT of the bytes. People `-F` a path without `@` and the server receives the path string, not the file. The `@` is the switch. People also `-T` to a URL that does not accept PUT and get a 405. The method has to match the server. `-v` shows which you sent. A large upload can time out. `--max-time` caps the whole transfer.

## Set a timeout
Also asked as: curl -m; curl --max-time; timeout; curl --connect-timeout; do not hang
`-m 30` fails the transfer if it has not finished in 30 seconds. `--connect-timeout 5` fails if the connection is not up in 5 seconds. They are different. A connect timeout does not limit a slow download. `-m` limits the whole operation. A timeout exits non-zero. A partial `-o` file may exist.

```sh
curl -m 30 -- 'URL'
```

People set only the connect timeout and a stalled download hangs. `-m` is the whole-transfer cap. People also set `-m 0` expecting "no limit" and that is already the default. A timeout is not a retry. `--retry 3` retries transient failures. It does not retry a 404. Combine `-m` with `--retry` in a script that talks to a flaky network. The timer is seconds, and it may be fractional on this curl.

## Use a proxy
Also asked as: curl -x; curl --proxy; http proxy; HTTPS_PROXY; socks proxy
`-x http://host:port` sends the request through that proxy. `-x socks5h://host:port` is a SOCKS5 proxy that resolves names itself. An empty `-x ''` overrides an environment proxy for this run. `http_proxy` and `https_proxy` in the environment are picked up with no flag. A proxy that needs a password takes `-U user:password`.

```sh
curl -x http://host:port -- 'URL'
```

People export a proxy and cannot reach an internal host. `-x ''` skips the proxy for one command. `NO_PROXY` is the environment list of exceptions. People also point `-x` at an HTTP proxy and ask for a `file://` URL. Proxies apply to network schemes. A refused proxy is a connection error, exit non-zero, and it looks like the target is down. `-v` shows the proxy connect. Fix the proxy before you blame the URL.

## Skip certificate checks, carefully
Also asked as: curl -k; curl --insecure; SSL certificate problem; self signed; insecure tls
`-k` does not verify the server's certificate. The transfer still encrypts. It does not check that you reached the right server. Use it for a local tool with a self-signed cert you already trust. Do not put it in a script that talks to the public internet. The durable fix is the right CA store, or `--cacert file` for a private CA.

```sh
curl --cacert file -- 'URL'
```

People `-k` a production URL and hide a real problem. An expired or a mismatched certificate is a reason to stop. People also see `SSL certificate problem` and assume the site is down. The TCP connection worked. The name or the trust failed. `--cacert` points at the CA that signed the server. `-k` is the override. A verbose run shows the certificate name curl saw. Compare it to the host you asked for.

## Send a cookie
Also asked as: curl -b; curl -c; cookie jar; curl cookie; session cookie
`-b 'name=value'` sends a cookie. `-b file` sends the cookies from a Netscape-format file. `-c file` writes received cookies to a file. A login often needs both. You `-c` on the login request, then `-b` on the next. `-c` creates the jar. A cookie attribute such as `HttpOnly` is the server's business. curl stores what it was given.

```sh
curl -c file -d 'pattern' -- 'URL'
```

People send `-b` on the login and never save the response cookie. The next request has no session. `-c` is the save. People also expect curl to use the browser's cookies. It does not. Export them, or log in through curl. A jar from `-c` is not a secret-free file. It holds the session. Mode it so only you can read it. `-b` with a wrong path fails the way a missing file fails, non-zero.

## Retry a flaky transfer
Also asked as: curl --retry; retry failed download; transient error; curl --retry-delay; try again
`--retry 3` retries up to three times on transient errors. A connection reset, a timeout, and a 5xx can qualify. A 404 does not. `--retry-delay 2` waits two seconds between tries. `--retry-all-errors` retries more kinds of failure, and it is newer. A retry restarts the transfer. It does not resume unless you also asked to resume.

```sh
curl --retry 3 --retry-delay 2 -- 'URL'
```

People retry a 400 and nothing changes. The request is wrong. Retry will not fix it. People also retry a large download and start over each time. `-C -` resumes if the server supports range requests. A retry that keeps failing exits with the last error. The status is still non-zero. Look at `-v` once before you raise the count. A DNS failure retries. A typo in the host will use all the attempts.

## Print only the status, or a header
Also asked as: curl -I; curl -w; http code; write-out; response code; just the headers
`-I` sends `HEAD` and prints the response headers. No body. Some servers do not implement `HEAD` and answer 405. `-w '%{http_code}\n'` prints the status after a normal GET. `-o /dev/null` discards the body when you only wanted the code. `-D -` writes response headers to stdout.

```sh
curl -o /dev/null -sS -w '%{http_code}\n' -- 'URL'
```

People use `-I` to check a URL that only answers `GET`, and get a 405 that the real request would not. Use a GET and `-o /dev/null` when you need the code of the real method. People also `-w` without discarding the body, and the code is glued to the page. `-o /dev/null` separates them. `-sS` keeps the meter off. The write-out happens even on failure, with a code of `000` when no response arrived. That zero is "no HTTP response," not a real status.

## Authenticate
Also asked as: curl -u; user and password; basic auth; bearer token; curl --user
`-u user:password` sends HTTP basic authentication. The password in the command is visible in the process list. `-u user` prompts for the password. A bearer token is a header, `-H "Authorization: Bearer token"`. `-n` reads `~/.netrc` for the host. Prefer the netrc or a prompt over a password in the command line.

```sh
curl -u user -- 'URL'
```

People put the token in the URL and it lands in the shell history and in the server's access log. A header is still visible to `-v`, and still in the process list for the duration. A netrc file with mode `600` is the calmer option. People also `-u` a host that wants a token and get a 401. Basic is not Bearer. The `WWW-Authenticate` header in a `-v` trace says which scheme the server asked for. Match that.

## What a failed curl means
Also asked as: curl exit status; curl exit code; curl failed; could not resolve host; connection refused; http code 000
Status 0 is a finished transfer, including an HTTP error page if you did not pass `-f`. Status 6 is a name that did not resolve. Status 7 is a refused connection. Status 28 is a timeout. Status 22 is `-f` seeing an HTTP error. Status 60 is a certificate problem. `curl` prints a short reason on stderr unless you passed `-s` without `-S`. `000` from `-w` means no response at all.

```sh
curl -sS -f -- 'URL'
```

People treat every non-zero as "the site is down." A certificate error and a DNS failure and a 404 under `-f` are different. Read the line. People also pass `-s` and have no line to read. Use `-sS`. A partial file from `-o` is not a successful download. Check the status before you open it. Under `set -e`, a failing curl aborts a script. That is what you want for a fetch a later step will parse. A 404 without `-f` will not abort, and the later step will parse an error page. Pass `-f` when the body must be the real document.
