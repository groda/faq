# ssh

`ssh` opens a login on another machine, or runs one command there, over an encrypted connection. It does not copy files. That is `scp` or `rsync`. It does not set up the server. With a host and no command it starts a remote shell and waits. OpenSSH on Linux is assumed here. macOS ships OpenSSH as well, with the same common flags. Dropbear is a different client and does not take every OpenSSH flag. Exit status is the status of the remote command. A login that ends with `exit` is 0. A failed connection, a refused key, or a timeout is 255. That number is `ssh` itself. It is not the remote program.

Basic form: ssh host

The host is a name or an address. `user@host` picks the account. Flags come before the host. A command may follow the host, and then `ssh` runs that instead of a shell. Quote the command so your local shell does not eat it. `--` is not how you end `ssh` options on every version. Put options first. A host that starts with a dash is a bad name. Use the name from `~/.ssh/config` when you have one.

## Log in to a host
Also asked as: ssh host; ssh user@host; log in remotely; open a remote shell; connect with ssh
`ssh host` connects and starts a shell as your current username. `ssh user@host` uses that account. The first connection to a new host asks you to confirm the host key. Type `yes` and it is stored in `~/.ssh/known_hosts`. A changed key is a warning and a refusal. That can be a reinstall, or it can be the wrong machine.

```sh
ssh user@host
```

People expect a password prompt and get a key failure, or the reverse. `ssh` tries keys, then the password, depending on the server. People also type the password into the local terminal before the prompt exists, and it is echoed nowhere useful. Wait for the prompt. A hanging connect with no message is often a firewall dropping the port. `-v` shows where it stopped. A refused connection means something answered and said no, which is a different failure from a timeout.

## Run one command
Also asked as: ssh host command; remote command; ssh and exit; run remotely; non-interactive ssh
A command after the host runs there instead of an interactive shell. `ssh` prints the remote stdout and stderr, and exits with the remote status. No command means a shell. The remote shell is non-interactive for this form, so it may not read `.bashrc`. Quote the command. A pipeline or a redirect in that command is remote only if the quotes keep it from your local shell.

```sh
ssh user@host -- command -file
```

```sh
ssh user@host 'command -- file'
```

Use the second form when the command has spaces, globs, or pipes. Those must be in the remote shell. An unquoted `*` expands on your machine. People also expect aliases from their remote interactive shell. A non-interactive bash does not load them unless the server is set up for that. Put the full path, or write a small script on the remote side. `ssh` does not allocate a terminal for a command unless you pass `-t`. A program that insists on a tty fails without it.

## Pick a user, a port, and a key
Also asked as: ssh -l; ssh -p; ssh -i; identity file; port 22; which key
`-l user` is the account. `-p port` is the port. The default port is 22. `-i file` is the private key to offer. You can repeat `-i`. A key file must be mode `600` or `ssh` refuses it. These flags are how you avoid a long `user@host` and a default key that the server does not accept.

```sh
ssh -i file -p 22 user@host
```

People pass the public key to `-i`. The public key is the `.pub` file, and it stays on the server. `-i` wants the private key. People also use `-P` for the port. `-P` is not the port. `-p` is the port. A wrong port looks like a timeout if it is filtered, or a refusal if something else is listening. `-v` shows the key `ssh` offered and whether the server accepted it. Several keys can be tried. The one that works is in the verbose trace.

## Copy your key so the password stops
Also asked as: ssh-copy-id; install public key; authorized_keys; passwordless ssh; copy key to server
`ssh-copy-id user@host` appends your public key to `~/.ssh/authorized_keys` on the server. After that, `ssh` can log in with the key. This is not an `ssh` flag. It is a helper. The private key never leaves your machine. A password prompt during `ssh-copy-id` is the last one you should need for that account, if the server allows keys.

```sh
ssh-copy-id -i file user@host
```

People paste the private key into `authorized_keys`. The file wants the public key, one line, starting with `ssh-ed25519` or `ssh-rsa`. People also leave `~/.ssh` on the server as world-writable, and the server ignores the file. `authorized_keys` should be `600`, and `.ssh` should be `700`. A key that still prompts means the server rejected it. `-v` says why. A forced command or a source restriction on the key can also refuse a normal login.

## Use a config so the command stays short
Also asked as: ~/.ssh/config; Host alias; IdentityFile in config; ssh config; shorten ssh command
`~/.ssh/config` holds defaults. A `Host` block matches the name you type. `HostName` is the real address. `User`, `Port`, and `IdentityFile` apply inside the block. Then `ssh work` is enough. The file must not be writable by others. A more specific `Host` block should come before a `Host *` block. The first match wins for each keyword.

```sh
ssh host
```

People edit the config and nothing changes, because a `Host *` earlier already set the key. Order matters. People also commit a private key path that only exists on one laptop. `IdentityFile` is local. A `Host` pattern is a glob, not a regex. `Host *.example.com` matches that suffix. `-F file` uses a different config and ignores the default one. `-G host` prints the config `ssh` would use, which is the way to see which block won.

## See why a connection failed
Also asked as: ssh -v; verbose ssh; debug ssh; connection refused; connection timed out; permission denied
`-v` prints the steps. Repeat it for more. `Connection refused` means the host answered and nothing is listening on that port, or a firewall rejected it. `Connection timed out` means no answer. `Permission denied` means you reached `sshd` and the key or the password failed. `Host key verification failed` means the key changed since `known_hosts` was written.

```sh
ssh -v user@host
```

People read a timeout as a bad password. The password stage never started. People also delete `known_hosts` on a key warning without looking. A changed key after a reinstall is normal. A changed key you did not expect is a reason to stop. The verbose line `Offering public key` followed by `Authentications that can continue` means that key was not accepted. Try the next, or check `authorized_keys`. `-v` does not fix the problem. It names the stage that failed.

## Allocate a terminal
Also asked as: ssh -t; ssh -tt; pseudo terminal; remote sudo; tput; ssh needs a tty
`-t` allocates a pseudo-terminal. A remote `sudo` that reads a password needs one. So does `top`, and a full-screen editor. `ssh host command` does not allocate one by default. `-t` forces it. `-tt` forces it even when the local side is not a terminal, which is the form inside a script that still needs a remote password prompt.

```sh
ssh -t user@host -- command
```

People `ssh host sudo command` and sudo says a terminal is required. `-t` is the missing piece. People also pass `-t` for a command that must not have a terminal, such as a transfer inside the session, and the remote program changes its buffering. Use `-T` to refuse a terminal. A `-tt` sudo in a script still needs the password from somewhere. Keys for the sudo account, or a sudoers rule, is the durable fix. `-t` only provides the tty.

## Forward a port
Also asked as: ssh -L; local forward; tunnel a port; ssh -L 8080:localhost:80; port forward
`-L local:remotehost:remoteport` listens on your machine and forwards connections through `ssh` to that host and port, as seen from the server. `-L 8080:localhost:80` is the usual shape. You then open `localhost:8080` at home. `-N` means do not run a remote command, which is what you want for a tunnel that only forwards. `-f` backgrounds it.

```sh
ssh -N -L 8080:localhost:80 user@host
```

People bind the local port and curl the remote address by mistake. The local port is the one on your machine. People also use a local port below 1024 and get permission denied. Use a high port, or run as root. A forward to `localhost` is evaluated on the server. That is the server's localhost, which is what you want for a service bound there. `-R` is the opposite direction. It listens on the server and brings connections back to you.

## Keep a connection alive
Also asked as: ssh timeout; ServerAliveInterval; connection reset; broken pipe; idle ssh drops
A firewall or a NAT may drop a quiet connection. `ServerAliveInterval=30` sends a keepalive every 30 seconds. `ServerAliveCountMax` is how many can be missed. Put them in the config, or pass them with `-o`. This does not stop the remote shell from exiting when you do. It only keeps the TCP session from being reaped while it is idle.

```sh
ssh -o ServerAliveInterval=30 user@host
```

People set the keepalive and still lose the session when their laptop sleeps. The timer does not run while the machine is suspended. People also confuse this with `tmux`. A keepalive does not let you reattach to a program. `tmux` or `screen` on the server does that. The interval is seconds. A very small value is needless traffic. 30 is a common choice. The server may have its own `ClientAliveInterval`, which is the other side of the same idea.

## Copy a file
Also asked as: scp; ssh copy file; sftp; rsync over ssh; transfer a file
`ssh` itself does not take a source and a destination path. `scp file user@host:path` copies to the server. `scp user@host:path file` copies back. A colon marks the remote side. `rsync -a -e ssh` is the better copy for anything large or repeated, because it can resume and skip unchanged files. `sftp` is the interactive form.

```sh
scp -- file user@host:dir
```

People `scp host:file` and forget the colon, so it looks for a local file named `host:file`. The colon is the remote marker. People also `scp` a directory without `-r` and get an error. `-r` is recursive. A trailing slash means different things to `scp` and to `rsync`. Check which tool you are in. `scp` of a missing remote path fails. It does not create the parent directories on every version. Make the directory first, or use a tool that will.

## Jump through a bastion
Also asked as: ssh -J; ProxyJump; jump host; bastion; ssh through a gateway
`-J bastion` connects to the bastion and from there to the host, in one command. `ProxyJump` in the config is the same thing. Your key for the final host is used from your machine, forwarded through the jump, if agent forwarding or the jump's own config allows the path. You do not have to shell into the bastion and ssh again.

```sh
ssh -J user@host user@host
```

People `ssh` the bastion, then `ssh` the inside host, and their agent and their config do not apply on the second hop the way they expect. `-J` keeps both hops under the local client. People also put the final host in the jump position. `-J` is the middle box. The last argument is the destination. A jump that needs a different key is a `Host` block with its own `IdentityFile`. Verbose mode shows both connections. A failure on the second hop still exits 255.

## Agent and forwarding
Also asked as: ssh-agent; ssh-add; agent forwarding; -A; key passphrase once
`ssh-agent` holds unlocked keys. `ssh-add file` adds a key, and you type the passphrase once. Later `ssh` uses the agent. `-A` forwards the agent to the server, so a hop from there can use your local key. Forwarding is convenient and it is a risk. A compromised remote account can use your agent while you are connected. Do not forward into a machine you do not trust.

```sh
ssh-add -- file
```

People start a new terminal and the agent is gone. The agent has to be in the environment, often via `SSH_AUTH_SOCK`. People also `ssh-add` the `.pub` file. Add the private key. A key with a passphrase and no agent prompts every time. That is the agent missing, not a wrong password. `-A` does nothing if no agent is running. Check `ssh-add -l`. An empty list means the agent has no keys.

## Disable password login, and other options
Also asked as: ssh -o; PreferredAuthentications; BatchMode; StrictHostKeyChecking; ssh option
`-o Key=value` sets one config option for this run. `BatchMode=yes` fails instead of prompting, which is what a script wants. `PreferredAuthentications=publickey` does not try a password. `StrictHostKeyChecking=accept-new` stores a new host key and still refuses a changed one. These are the same keys as in `~/.ssh/config`.

```sh
ssh -o BatchMode=yes user@host
```

People run `ssh` from cron and it hangs on a host-key prompt or a password prompt. `BatchMode=yes` turns the hang into a fast failure. People also set `StrictHostKeyChecking=no` and accept any host. That disables the check that detects a changed server. `accept-new` is the narrower choice. An option on the command line overrides the config file. `-F /dev/null` ignores your config entirely, which is the clean test when a stanza is misbehaving.

## Escape a stuck session
Also asked as: ssh escape; ~.; kill a hung ssh; enter tilde; ssh escape character
A session that is alive but quiet can be closed with the escape sequence. Press Enter, then `~.` and `ssh` drops the connection. `~?` lists the sequences. `~^Z` suspends `ssh`. The tilde is recognized only at the start of a line. This is local. It does not send `~.` to the remote program. It is how you leave a dead network without waiting for the timeout.

```sh
ssh user@host
```

People type `~.` in the middle of a line and it is sent to the remote shell as text. Enter first. People also change the escape character and forget. `-e none` disables it. A truly dead TCP session may ignore the escape until a keepalive fails. `~.` still works when the client is reading the terminal. If the terminal itself is wedged, the escape cannot help. The sequence is the clean exit. Killing the local `ssh` process is the other one.

## What a failed ssh means
Also asked as: ssh exit code; ssh exit 255; ssh failed; permission denied publickey; ssh status
`ssh` exits 255 when the connection failed, the authentication failed, or the protocol broke. A remote command's status is passed through when the session started. A remote `exit 1` is 1, not 255. A timeout is 255. `BatchMode` failures are 255. You cannot tell a bad key from a down host by the number alone. The message and `-v` are the distinction.

```sh
ssh user@host -- command
```

People test `$?` for any non-zero and treat it as a remote failure. 255 means `ssh` never got a remote status. Do not retry a 255 in a tight loop against a host that is refusing you. People also ignore `Permission denied (publickey)` and type a password that will never be asked for, because the server has passwords disabled. The parenthetical is the list of methods that were tried. Fix the key, or the account, before you run it again. Under `set -e`, a failed `ssh` aborts a script. A remote command that exits 1 does too. Those are different problems, and the output usually says which.
