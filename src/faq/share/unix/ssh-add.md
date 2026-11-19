# ssh-add

`ssh-add` loads a private key into an `ssh-agent` so later `ssh` can use it without asking for the passphrase each time. It does not create a key. That is `ssh-keygen`. It does not copy a key to a server. That is `ssh-copy-id`. With no arguments it adds the default keys (`~/.ssh/id_rsa`, `id_ecdsa`, `id_ed25519`, and a few others) if they exist. OpenSSH on Linux is assumed here. macOS ships OpenSSH as well, and its agent is often already running via the system. Exit status is 0 when the key was added or the list was printed. It is non-zero when no agent is running, the key file cannot be read, or the passphrase is wrong.

Basic form: ssh-add

The agent must already be running and `SSH_AUTH_SOCK` must point at its socket. Keys are files you name, or the defaults. A passphrase is read from the terminal, or from a prompt program if `SSH_ASKPASS` is set. The key stays in the agent until the agent exits, you remove it, or a lifetime you set expires. Nothing is written to the server by this command.

## Add a key so ssh stops asking
Also asked as: ssh-add; add key to agent; load private key; ssh-add id_ed25519; unlock key
`ssh-add file` loads that private key into the running agent. You type the passphrase once. After that, `ssh` to a host that trusts the matching public key can authenticate without prompting. With no file it tries the default names in `~/.ssh`. The public key is not what you add. The private key is.

```sh
ssh-add -- file
```

People run `ssh-add` and get "Could not open a connection to your authentication agent." No agent is running, or this shell does not have `SSH_AUTH_SOCK`. Start one with `eval "$(ssh-agent)"` or use the desktop session's agent. People also pass the `.pub` file. The agent wants the private half. A wrong passphrase fails the add and the key is not loaded. Try again. The key file should be mode `600` or `ssh-add` may refuse it.

## List the keys the agent holds
Also asked as: ssh-add -l; list keys; what keys are loaded; ssh-add -L; show public keys
`-l` lists the fingerprints of keys currently in the agent. `-L` prints the full public keys. An empty agent prints "The agent has no identities." and exits non-zero. This is how you check that an add worked, or which key will be offered. It does not show keys that exist only as files on disk.

```sh
ssh-add -l
```

People expect `-l` to list every private key in `~/.ssh`. It lists only what has been added to this agent. People also run it in a new terminal where `SSH_AUTH_SOCK` is unset and get the "could not open a connection" error. The list is per agent. Two agents do not share keys. A key with a comment shows that comment. The fingerprint is what you compare to a server log.

## Remove a key from the agent
Also asked as: ssh-add -d; delete key; remove identity; ssh-add -D; unload key
`-d file` removes that key from the agent. `-D` removes every key. The file on disk is not deleted. Only the agent copy is dropped. After a remove, `ssh` will not offer that key until you add it again. A key that was never added is a harmless error on `-d`.

```sh
ssh-add -d -- file
```

People `-d` and expect the key file to vanish. It stays. People also `-D` and then wonder why a host that worked a minute ago now asks for a password or fails. The agent is empty. Add the key again, or use a different authentication method. Removing a key does not revoke it on any server. The public key in `authorized_keys` is still valid. You only stopped offering it from this agent.

## Set a time limit on a key
Also asked as: ssh-add -t; key lifetime; expire key; ssh-add timeout; temporary key
`-t` sets a lifetime in seconds. `ssh-add -t 3600 file` keeps the key for one hour, then the agent drops it. The default is forever, until the agent exits or you remove it. A short lifetime is useful on a shared or unattended machine. The timer starts when the key is added.

```sh
ssh-add -t 3600 -- file
```

People set `-t` and expect it to lock the key rather than remove it. When the time is up the key is gone. Add it again if you still need it. People also use a very short lifetime and an automated `ssh` fails later with no obvious cause. Check `ssh-add -l`. The key will be missing. A lifetime does not survive an agent restart. The new agent starts empty.

## Add a key from a non-default path
Also asked as: ssh-add path; key not in .ssh; add a specific key; ssh-add ~/.ssh/work; named key file
Pass the path. `ssh-add ~/.ssh/work_ed25519` loads that file. Relative paths are relative to the current directory. The agent stores the key material, not the path, for later use. You can add several keys. `ssh` will offer them in an order the client chooses until one is accepted or the server rejects the set.

```sh
ssh-add -- file
```

People add a key and `ssh` still fails because the server has a different public key. Confirm with `ssh-add -L` and compare to `authorized_keys`. People also add the same key twice. The agent keeps one. A path with a space needs quotes. A key encrypted with a passphrase you do not have cannot be added. There is no bypass. Generate a new key or recover the passphrase.

## Start the agent first
Also asked as: ssh-agent; no agent; SSH_AUTH_SOCK; Could not open a connection; start agent
`ssh-add` talks to a running agent. `eval "$(ssh-agent)"` starts one and sets `SSH_AUTH_SOCK` and `SSH_AGENT_PID` in the current shell. A desktop environment often starts an agent for you at login. A script or a new terminal may not see it. Without the socket variable, `ssh-add` cannot connect.

```sh
eval "$(ssh-agent)"
```

People start the agent in one terminal and run `ssh-add` in another. The second shell does not have the variables. Export them, or start the agent in the shell you will use. People also run `ssh-agent` without `eval` and the output is printed but not applied. The `eval` is what sets the variables. Killing the agent (`ssh-agent -k` or kill the pid) drops every key it held. They are not saved.

## Confirm a host key or use confirm mode
Also asked as: ssh-add -c; confirm before use; ask each time; ssh-add confirm; key confirmation
`-c` marks the key so the agent asks for confirmation (via `SSH_ASKPASS` or a prompt) each time the key is about to be used. The passphrase is still entered only once, at add time. This is a speed bump against a process using your agent without you noticing. Not every agent setup shows the confirm prompt cleanly. Test it.

```sh
ssh-add -c -- file
```

People expect `-c` to require the passphrase on every `ssh`. It requires a confirmation, not the passphrase. The key is already unlocked. People also set `-c` in a headless session and the confirm cannot be answered, so `ssh` hangs or fails. Use `-c` where a prompt can appear. A key added with `-c` still appears in `ssh-add -l`. The confirm happens at use time.

## Add a certificate or a key with a constraint
Also asked as: ssh-add certificate; key constraints; ssh-add -s; pkcs11; hardware key
`ssh-add` can load a key that is paired with a certificate file in the same directory (same name with `-cert.pub`). It can also talk to a PKCS#11 token or a security key with the right flags and libraries. A hardware-backed key may still prompt on the device. The ordinary file case is the common one. Constraints such as `-c` or `-t` apply at add time.

```sh
ssh-add -- file
```

People have a certificate and add only the private key, then the server rejects the authentication because the cert was not offered. Keep the cert next to the key with the expected name, or pass the options your OpenSSH version documents. People also expect `ssh-add` to generate a certificate. It does not. Certificates come from a CA or from `ssh-keygen`. The agent only holds what you give it.

## What a failed ssh-add means
Also asked as: ssh-add exit status; ssh-add failed; bad passphrase; agent refused; cannot add key
No agent, or no `SSH_AUTH_SOCK`, exits non-zero with "Could not open a connection to your authentication agent." A bad passphrase exits non-zero and does not load the key. A missing file or a permission error exits non-zero. `-l` on an empty agent exits non-zero with "The agent has no identities." A successful add exits 0 and later `ssh` can use the key.

```sh
ssh-add -- file
```

People treat an empty `-l` as a broken agent. The agent is running and holds nothing. Add a key. People also retry a bad passphrase in a script and lock nothing (the agent does not rate-limit the add itself the way some tokens do). Check the file and the passphrase. Under `set -e`, a failed `ssh-add` aborts a script. That is useful before a later `ssh` that would otherwise hang waiting for a prompt. Start the agent and add the key in the same shell that will run `ssh`, or export the socket so the later command can see it.
