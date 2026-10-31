# podman

`podman` runs containers. A container is a process with its own filesystem and its own network view, built from an image. `podman` does not need a daemon. It does not replace a virtual machine, and it does not edit the image unless you build or commit. With no arguments it prints help and exits non-zero. Podman from the containers/podman project on Linux is assumed here. On macOS and Windows, `podman` drives a Linux virtual machine. `podman machine` starts that VM. The commands after that are the same. There is no BusyBox build. Exit status of `podman run` is the exit status of the process inside the container. A client error, such as a missing image name or a bad flag, exits non-zero before the container starts. Rootless is the normal way to run it. Root is optional.

Basic form: podman run image

`run` creates a container from an image and starts it. The image name can include a registry and a tag, as in `image:tag`. Flags for `run` come before the image. Anything after the image is the command inside the container, replacing its default. Quote a value the shell would expand. `--` is rarely needed. A name that starts with a dash belongs after the image if it is the program's own flag.

## Run a container in the foreground
Also asked as: podman run; start a container; run an image; pull and run; podman run image
`podman run image` pulls the image if it is not local, creates a container, and runs its default command in the foreground. Your terminal is attached. The container is removed when it exits only if you passed `--rm`. Without `--rm` the stopped container remains, and `podman ps -a` shows it. The image stays either way.

```sh
podman run -- image
```

People expect `run` to start a server and return. A server that stays in the foreground holds the terminal. `-d` is the background form. People also pass the command's flags before the image, and `podman` reads them as its own. Flags for the program inside go after the image name. A missing image is pulled from the configured registries. A typo in the name fails the pull and nothing starts. The status you see is the program's status once it exits.

## Run in the background
Also asked as: podman run -d; detach; daemon container; run and return; podman background
`-d` starts the container and returns. `podman` prints the container id. The process keeps running. Logs go to the container's log, not your terminal. `podman logs` reads them. Without `-d`, a long-running process sits in the foreground and dies with the terminal unless you arranged otherwise.

```sh
podman run -d -- image
```

People `-d` an interactive program and it exits immediately, because stdin is closed. A server that listens is the thing to detach. People also lose the id. `podman ps` lists what is running. A name from `--name` is easier to type than the id. `-d` does not restart the container after a reboot. A restart policy or a systemd unit does that. The status of a detached `run` is 0 if the container started, not the later status of the process inside.

## Name the container
Also asked as: podman run --name; container name; name a container; podman ps name; friendly name
`--name` assigns a name you choose. Later commands take that name instead of the id. Names are unique. A second `run` with the same name fails until you remove the first container. The name survives stop and start. It is not the image name. One image can have many containers.

```sh
podman run --name pattern -d -- image
```

People reuse a name and get an error that the name is in use. The old container still exists, stopped or not. `podman rm` removes it. `podman run --replace` removes a container of that name and creates a new one. That flag is the blunt form. People also expect the name to appear in the image list. `podman ps` shows container names. `podman images` shows images. They are different lists.

## Publish a port
Also asked as: podman run -p; map a port; publish port; host port to container; -p 8080:80
`-p host:container` publishes a container port on the host. `8080:80` means host port 8080 forwards to port 80 in the container. The process inside must listen on that container port. Rootless podman can only bind host ports above 1024 unless you have granted it the right. A port already in use fails the run.

```sh
podman run -d -p 8080:80 -- image
```

People publish a port and curl `localhost` inside the container by mistake. From the host, the address is the host's port. From another container, the published port is not the usual path. People also forget that the application inside still binds to its own port. Changing only `-p` does not change what the program listens on. A firewall on the host can block the published port. `podman` does not open host firewalls. Rootless networking uses a user-space stack, and some protocols behave differently than under root.

## Mount a directory
Also asked as: podman run -v; volume mount; bind mount; mount host directory; -v host:container
`-v hostpath:containerpath` bind-mounts a host directory into the container. The path inside is where the process sees those files. A relative host path is relative to the current directory. The directory should exist. SELinux, when it is enforcing, often needs `:Z` on the mount so the container may read it. `:Z` relabels the host path. Do not use it on a system directory.

```sh
podman run -v dir:dir -- image
```

People mount a file and the path inside is a directory, or the reverse, and the mount looks empty. The types must agree. People also edit files on the host and the container sees them, because it is the same files. That is the point of a bind mount. A named volume, `-v name:containerpath`, is managed by `podman` and is not a path you pick. Rootless mounts cannot always preserve ownership the way people expect. A file owned by you on the host may look like a different id inside.

## Pass environment variables
Also asked as: podman run -e; environment variable; --env; set a variable inside; pass config
`-e NAME=value` sets a variable in the container's environment. `-e NAME` copies the variable from your environment. Repeat the flag for several. The image may also declare variables. A value with spaces needs quotes so the shell does not split it. Secrets in `-e` are visible in `podman inspect` and in the process list on the host.

```sh
podman run -e NAME=pattern -- image
```

People export a variable and expect the container to inherit the whole environment. It does not. Pass what you mean with `-e`. People also put secrets on the command line. They land in the shell history and in the container config. A file mounted into the container is the better place for a secret. `--env-file` reads `NAME=value` lines from a file. The file is not copied into the image. It is read at run time.

## List running containers
Also asked as: podman ps; list containers; what is running; podman ps -a; show stopped
`podman ps` lists running containers. `-a` includes stopped ones. The columns are id, image, command, created, status, ports, and names. A stopped container still occupies its name. An image is not a container. `podman images` is the other list. `ps` does not show containers in another user's rootless namespace.

```sh
podman ps -a
```

People run a container, it exits, and `ps` is empty. It stopped. `-a` shows it, with an exited status and the code. People also look for a container they started as root while they are rootless, or the reverse. Those are different stores. `podman ps` as your user does not show root's containers. A container with no name shows its id. `--name` is how you avoid typing ids.

## List and remove images
Also asked as: podman images; list images; podman rmi; delete an image; remove unused images
`podman images` lists images in the local store. The name and tag identify them. `<none>` is an untagged image, often left by a build. `podman rmi image` removes an image. It refuses if a container still references it, unless you force it. `podman image prune` removes unused images. Removing an image does not remove the containers that were created from it until those are gone.

```sh
podman images
```

People delete an image and expect the running container to vanish. The container has its own writable layer. It keeps running. People also `rmi` a tag and think the image is gone when another tag still points at it. The id is the object. Tags are names. A prune does not remove images that a container, even a stopped one, still uses. Remove the containers first if you mean to reclaim the space.

## Stop, start, and remove a container
Also asked as: podman stop; podman start; podman rm; remove container; restart a container
`podman stop name` sends the stop signal and waits, then kills. `podman start name` runs a stopped container again, with the same mounts and the same writable layer. `podman rm name` deletes a stopped container. `-f` removes it even if it is running. `podman rm -a` removes all stopped containers. Start does not pull a new image. It reuses the container.

```sh
podman stop -- pattern
```

People `rm` a running container and get an error. Stop it, or pass `-f`. People also `run` again to "restart" and end up with two containers. `start` is the one that keeps the old writable layer, including files the process wrote inside. `run` creates a new container. A stop does not remove the image. A remove of the container does not remove the image either.

## Read the logs
Also asked as: podman logs; container logs; see output; podman logs -f; follow logs
`podman logs name` prints the container's standard output and standard error. `-f` follows, like `tail -f`. The log is what the process wrote, not a host syslog. A container that logs to a file inside itself does not show that file here. `--tail` limits how many lines. Logs survive stop. They go away when you remove the container.

```sh
podman logs -- pattern
```

People expect `logs` to show why a container will not start. The start failure is often a `podman` message, printed when you ran it, not in the log. People also follow a log and wonder why it is empty. The process wrote to a file, or it has not printed yet, or it is fully buffered because stdout is not a terminal. `-d` does not change that. An interactive container you did not detach has its output on your terminal already. `logs` is for the detached case.

## Run a command in a container that is already up
Also asked as: podman exec; shell into a container; exec bash; run a command inside; podman exec -it
`podman exec name command` runs a new process inside an existing container. `-i -t` allocates a terminal and keeps stdin open, which is what an interactive shell needs. The container must be running. `exec` does not start a stopped container. The process shares the container's filesystem and network. It is not a new container.

```sh
podman exec -it -- pattern command
```

People `exec` into a stopped container and get an error. `start` it first, or use `run` for a one-shot. People also ask for `bash` in an image that has only `sh`. The shell has to exist in the image. A missing command is an error from the exec, and the container keeps running. `exec` as rootless is still rootless. A shell inside does not grant host root. Changes you make to the writable layer survive stop, until you remove the container.

## Build an image
Also asked as: podman build; Containerfile; Dockerfile; build an image; podman build -t
`podman build -t name .` builds an image from a `Containerfile` or a `Dockerfile` in the current directory. `-t` names the result. `-f` points at a file elsewhere. The build context is the directory you name. Files outside it are invisible to `COPY`. The result is a local image. Nothing is pushed unless you push.

```sh
podman build -t pattern .
```

People `COPY` a file that is not in the context and the build fails. The context is the last argument, usually `.`. People also expect the build to start a container. It only produces an image. `podman run` is the next step. A failed instruction stops the build. The layers before it may be cached. `--no-cache` rebuilds. Rootless build cannot do some things a root build can, such as certain mounts. The error says so.

## Pull an image without running it
Also asked as: podman pull; download an image; fetch image; podman pull image; update an image
`podman pull image` downloads an image and stores it. It does not create a container. A later `run` of that name uses the local copy. A tag such as `latest` is a name, not a promise. Pull again to get whatever the registry now calls by that tag. `podman pull` does not update containers that already exist.

```sh
podman pull -- image
```

People pull and expect the running container to change. It does not. The old container still uses the layers it was created with. Create a new container to use the new image. People also pull without a registry and rely on the search list in the registries config. An unqualified name can come from a registry you did not mean. Prefer a full name when it matters. A failed pull leaves no partial container. It may leave a partial image that a later pull resumes.

## Copy files in and out
Also asked as: podman cp; copy from container; copy to container; podman cp file; transfer a file
`podman cp name:path hostpath` copies from the container to the host. Swap the arguments to copy in. The container can be stopped. `cp` is a copy, not a mount. Later changes on one side are not reflected on the other. For a live shared directory, use `-v` at run time.

```sh
podman cp -- pattern:file file
```

People `cp` into a path that does not exist in the container and the copy fails, or lands somewhere they did not expect. The destination directory should exist. People also use `cp` where a mount belonged, and then wonder why the next run does not have the file. A new `run` is a new writable layer unless you mounted something. `cp` into a running container is visible to the process immediately. Ownership inside follows the container's user namespace when you are rootless.

## Use a restart policy
Also asked as: podman run --restart; restart always; restart on failure; container comes back; --restart=always
`--restart=always` restarts the container when it exits, including after a reboot if the podman restart service is enabled. `--restart=on-failure` restarts only when the process exits non-zero. The default is no restart. A restart policy is not a supervisor you can talk to. `podman stop` still stops it. A container that exits immediately will be restarted in a loop under `always`.

```sh
podman run -d --restart=always -- image
```

People set a restart policy and reboot, and the container stays down. Rootless podman needs the user lingering, and the restart service enabled, or a reboot does not bring it back. `loginctl enable-linger` is the usual piece. People also use `always` for a job that should run once. It will run again. `on-failure` is the narrower policy. A restart does not pull a new image. It starts the same container.

## Run rootless, or as root
Also asked as: rootless podman; podman versus sudo podman; rootless container; unprivileged port; user namespace
With no `sudo`, `podman` is rootless. Images and containers live in your home directory. Ports below 1024 are not available by default. The process inside can think it is root and still be an ordinary user on the host. `sudo podman` is a different store, owned by root. Containers from one are invisible to the other. Rootless is the one to use unless you need a privileged port or a device.

```sh
podman run -- image
```

People `sudo podman pull` and then `podman run` as themselves, and the image is missing. They pulled into the other store. People also expect a rootless container to bind port 80. It cannot, without extra privileges. Map a high port, or grant the capability. A volume's file ownership surprises people because the user namespace maps ids. A file that is yours on the host can appear as root or as nobody inside, depending on the map.

## Generate a unit so it starts on boot
Also asked as: podman generate systemd; quadlet; start on boot; systemd service; podman container unit
`podman generate systemd --name name` prints a systemd unit that starts that container. Quadlet is the newer form. A `.container` file under `~/.config/containers/systemd/` becomes a user unit. Rootless units start at user login unless lingering is enabled. The unit is not created by `run` itself. You generate it, or you write the quadlet file.

```sh
podman generate systemd --name pattern
```

People reboot and the container is gone. Nothing installed a unit. People also generate a system unit for a rootless container and it fails, because the container is in the user's store. A user unit is the match for a rootless container. `--new` in the generated unit creates the container at start rather than requiring it to exist. Read the unit before you enable it. A generated unit does not update itself when you change the `run` flags. Generate again, or edit.

## Tell it from docker
Also asked as: podman versus docker; docker compatible; alias docker=podman; podman socket; same flags
Most `docker` commands work as `podman` commands. `run`, `ps`, `exec`, `build`, `pull`, and `rmi` behave the same for ordinary use. `podman` has no daemon. `docker` talks to one. A compose file often works with `podman compose` or a provider. Differences show up in rootless networking, in systemd integration, and in the socket path. An alias is fine for interactive use. A script should call the one it means.

```sh
podman run -- image
```

People point a Docker-oriented tool at `podman` and the socket is missing. `podman system service` opens a Docker-compatible socket if a tool requires one. People also expect `podman` and `docker` to share images. They do not, unless you configured a shared store. Each has its own. A flag that is Docker-only fails with a usage error. The common flags are the same. Prefer the `podman` command in anything you will keep.

## What a failed podman means
Also asked as: podman exit status; podman failed; container exited; error pulling image; name already in use
A usage error exits non-zero and no container is created. A pull error exits non-zero and nothing starts. `podman run` without `-d` returns the process's exit code, so a failing program looks like a failing `podman`. With `-d`, a successful start is 0 even if the process exits a moment later. `podman ps -a` shows the exited status. The log shows what it printed.

```sh
podman run -- image
```

People read a non-zero status from a foreground `run` as a `podman` bug. The program inside exited that way. People also ignore "name already in use" and retry. Remove the old container, or pass `--replace`, or pick another name. A rootless failure about ports, or about a volume label, is an environment limit, not a bad image. The message names the constraint. Under `set -e`, a failing foreground `run` aborts a script. That is usually what you want. A detached run that immediately exits does not fail the `podman` command. Check `ps -a` before you treat a detached start as a running service.
