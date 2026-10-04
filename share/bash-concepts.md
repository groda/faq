# bash-concepts

These are the words behind the commands. A process has a current directory, an environment, and three standard streams already open. The shell starts that process, expands the words, and waits for a status. The kernel stores a file under an inode. A directory entry is only a name that points at one. Bash on Linux, with GNU coreutils, is assumed here. macOS uses the same streams and the same inode idea, and its default `/bin/bash` is old. BusyBox ash has streams, pipes, and statuses, and it has no `[[ ]]` and no arrays. A concept is not a flag. Quoting and expansion happen before the command runs.

Basic form: command -- file

The shell splits the line into words, expands globs and variables, then runs the command. Single quotes hide `$` and `*`. Double quotes expand `$` and keep the result as one word. `--` is for the command, so a filename may start with a dash. The status is a small integer. `0` is success. Anything else is failure. A pipe connects two processes. It is not a file you can seek.

## Standard output
Also asked as: stdout; file descriptor 1; where print goes; normal output; the stream a pipe reads
Standard output is file descriptor 1. `printf`, `echo`, and a command's normal report write there. The shell can point it at a file with `>`, at the end of a file with `>>`, or into a pipe. The command does not know which. It writes bytes. A terminal shows them. A pipe hands them to the next command. A redirect throws them away or stores them.

```sh
printf '%s\n' 'pattern' > file
```

People say "print" and mean the screen. The screen is only the current destination of descriptor 1. A command in a pipe has no screen on that descriptor. It has the pipe. Buffering changes when the destination is not a terminal. Output can sit in a block until the buffer fills or the command exits. That delay is not a lost print. `>&2` sends this command's output to standard error instead.

## Standard error
Also asked as: stderr; file descriptor 2; error stream; why errors skip the pipe; diagnostics
Standard error is file descriptor 2. Warnings and errors go there so a pipe on standard output stays clean. By default it still points at the terminal, which is why a failing command in a pipeline can print an error you did not redirect. `2>` points it at a file. `2>/dev/null` discards it. The status is separate. Hiding the message does not make the command succeed.

```sh
command -- file 2> file
```

`2>&1` copies standard error onto wherever standard output is going at that moment. The order is the whole trick. `> file 2>&1` sends both to the file. `2>&1 > file` sends errors to the old standard output, then moves standard output to the file. People write the second and think the shell is ignoring them. It applied the redirects left to right. Descriptor 2 is not "the log." It is whatever you pointed it at.

## Standard input
Also asked as: stdin; file descriptor 0; what a program reads; the pipe a command consumes; keyboard input
Standard input is file descriptor 0. A program that reads with no filename reads it. With no redirect and no pipe, that is the terminal, and the program waits until you send end of file. A pipe fills it from the previous command. `< file` fills it from a file. The program cannot tell those apart by the bytes alone.

```sh
command -- file < file
```

People run `grep pattern` with no file and think it is searching the directory. It is waiting for stdin. A command that ignores stdin, such as `rm`, does not become a filter because you piped to it. The pipe still connects. The command just never reads it, and the producer may get a broken pipe if it writes. `-` as a filename means stdin only for commands that chose that convention. It is not a shell feature.

## A pipe
Also asked as: pipe; vertical bar; cmd | cmd; connect stdout to stdin; pipeline
A pipe sends the standard output of the left command to the standard input of the right command. Both commands run at the same time. The left one blocks when the pipe buffer is full. The right one blocks when the buffer is empty. Standard error is not in the pipe. The status of the pipeline is the status of the rightmost command, unless `pipefail` is set.

```sh
command -- file | command -- file
```

People put `2>` on the right and wonder why the left command's errors still hit the terminal. The redirect belongs to one stage. Merge on the left if those errors must enter the pipe. A pipe is not a temporary file. You cannot rewind it. `cat` in the middle copies bytes and adds nothing. The stages are separate processes. A variable set on the right is gone when the pipeline finishes, because that stage was a subshell.

## Exit status
Also asked as: $? ; exit code; zero means success; non-zero failure; status of the last command
Every command returns a status, a number from 0 to 255. `0` means success. `1` often means "the test failed" or "lines differed." Higher values often mean "could not run." `$?` is the status of the last command, and the next command replaces it. A pipeline without `pipefail` reports the last stage only. `set -e` makes a non-zero status exit the shell, with exceptions for tests and `||`.

```sh
command -- file
printf '%s\n' "$?"
```

People treat any output as success and any silence as failure. `grep` can succeed and print nothing if you only asked for a count of zero. `grep` exits 1 when it finds no match, which is a result, not a crash. `exit 2` in a script is how you choose the status. A status above 128 often means the process died from a signal. 130 is the usual Control-C. The number is not a message. The message was on stderr.

## File descriptors
Also asked as: fd; open files; descriptor 0 1 2; /dev/fd; extra descriptors
A file descriptor is a small integer the process uses to read or write an open file, pipe, or socket. 0, 1, and 2 are already open. The shell's redirects are instructions to arrange those integers before the command starts. `3> file` opens descriptor 3. `2>&1` means "make 2 point where 1 points now." The command inherits the arrangement. It does not inherit the redirect syntax.

```sh
command -- file 2>&1
```

People think `>` changes a command's source code. It changes the table of open descriptors in the new process. Closing a descriptor is `>&-`. A script that runs out of descriptors leaked them in a loop, often by opening files in awk or by redirecting inside a loop the naive way. `/dev/fd/1` is a path to descriptor 1 on Linux. It is a way to name the stream. It is not a second copy of the data.

## Redirects
Also asked as: > >> <; redirection; overwrite versus append; noclobber; open a file for the command
`>` truncates a file and attaches descriptor 1 to it. `>>` appends. `<` attaches descriptor 0. The shell opens the file before the command runs. If the open fails, the command does not run. `set -o noclobber` makes `>` fail when the file exists. `>|` overrides that. A redirect is not visible to the command as an argument.

```sh
command -- file > file
```

The classic bug is `command file > file` on the same path. The shell truncates the file, then the command reads an empty file. Write to a new name and rename it. Several redirects on one command apply left to right. `> file 2>&1` is both streams. A redirect on a compound group `{ cmd; } > file` applies to every command in the group. A redirect on one command in a pipeline applies to that stage only.

## Inodes
Also asked as: inode; file metadata; link count; what a filename points at; stat inode
An inode is the filesystem object. It holds the mode, the owner, the size, the timestamps, and the pointers to the data blocks. It does not hold the name. A directory entry holds the name and an inode number. Two names can point at the same inode. That is a hard link. `ls -i` prints the number. `stat` prints the metadata. `rm` removes a name. The inode goes away when the last name is gone and no process still has it open.

```sh
ls -i -- file
```

People delete a file that a process has open and expect the disk space to return. The directory entry is gone. The inode remains until the process exits, so `df` still shows the space and `du` does not. People also expect a hard link to be a copy. It is a second name. Edit one and you edit the other. A hard link does not cross filesystems. The inode numbers are per filesystem. The same number on two disks is not the same file.

## Directories and names
Also asked as: directory entry; filename; dentry; path lookup; what rm removes
A directory is a file whose contents are names. Each name points at an inode. `.` is the directory itself. `..` is its parent. A path is a walk through those names. The kernel checks execute permission on each directory in the walk, not only on the final file. Write permission on a directory is what lets you create or remove a name there. Write permission on the file is what lets you change its bytes.

```sh
ls -l -- dir
```

People `chmod 644` a directory and can no longer open files inside it. The directory lost search permission. People `rm` a file they own and get "Permission denied" because the directory is not writable. The sticky bit on a directory, as on `/tmp`, adds a rule. You may remove only names you own. A name is not the file. Rename is a new name, often on the same inode, not a copy of the bytes.

## Symlinks
Also asked as: symbolic link; ln -s; symlink versus hard link; follow a link; readlink
A symlink is a file whose content is a path. Opening it normally follows that path. `ls -l` shows the target. `rm` on the link removes the link, not the target. A hard link is another name for the same inode. A symlink can point at a directory, at a missing path, and across filesystems. A dangling symlink fails when something follows it, and succeeds when something operates on the link itself.

```sh
ls -l -- file
```

People `rm` a symlink and think they deleted the data. They deleted a path string. People `chmod` a symlink on Linux and change the target, because `chmod` follows. `chown -h` and `ls` without following are the operations on the link. A relative target is resolved from the directory that holds the link, not from your current directory. Move the link and the target can stop making sense.

## Processes
Also asked as: process; pid; fork and exec; child process; what the shell starts
A command name that is not a builtin starts a new process. The shell forks, arranges the descriptors and the current directory, then execs the program. The process id is how you name that running image. `$$` is the shell's own pid. `$!` is the pid of the last background process. When the child exits, its status is what the shell reports. A builtin such as `cd` runs inside the shell and can change it.

```sh
command -- file
```

People `cd` in a script they ran as a command and expect their terminal to move. The `cd` happened in the child. The parent shell is unchanged. `export` in the child does not change the parent's environment. A program image is what `exec` loaded. The arguments are in the process. They are not reread from the script. Kill the pid and you kill that image, not the name you typed, if a new process has reused nothing yet. Pids are reused.

## Subshells
Also asked as: subshell; parentheses; pipeline subshell; variable lost after pipe; ( cmd )
Parentheses run the commands in a child shell. `cd`, variable assignments, and `exit` inside them do not affect the caller. A pipeline also runs its stages in subshells. An assignment in the last stage is gone when the pipe finishes. `{ cmd; }` is a group in the current shell. The semicolon and the spaces are required. It is not a subshell.

```sh
( cd -- dir && command -- file )
```

Use the parentheses when you want to visit a directory and come back, including on failure of the inner command if you handled the status. Do not use them for a result you need to keep. `var=$(command)` is a subshell too. The assignment of its output happens in the current shell. The commands inside the substitution do not. `set -e` in a subshell exits that subshell, not the parent, unless the parent was waiting on it under errexit.

## Quoting
Also asked as: single quotes; double quotes; word splitting; quote a variable; IFS
The shell splits the result of an unquoted expansion on `IFS`, then globs what remains. `"$file"` is one word. `'$file'` is the characters dollar, f, i, l, e. Single quotes are literal. Double quotes expand variables and command substitutions and still prevent splitting. A quote ends at the matching quote. You cannot put a single quote inside single quotes by escaping it. You end them, add `\'`, and start them again, or you use `$'...'` in bash.

```sh
printf '%s\n' "$file"
```

People quote the assignment, `file="a b"`, and then use `$file` unquoted. The assignment was safe. The use is where it splits. A glob character in the value expands only if you leave it unquoted. Quotes are shell syntax. The command does not see them. `command` receives the words the shell produced. `echo` is a bad test of that, because it joins its arguments with spaces. `printf '%s\n'` one argument at a time shows the words.

## Globbing
Also asked as: star; wildcard; pathname expansion; nullglob; dotfiles; what the shell expands
`*`, `?`, and `[abc]` expand to matching names before the command runs. `*` does not match a leading dot and does not cross `/`. If nothing matches, bash leaves the pattern as a literal word. `nullglob` makes it disappear. `failglob` makes it an error. The command never sees the star unless the star survived. `**` is not recursive unless `globstar` is set, and that option is bash 4.

```sh
printf '%s\n' -- *.txt
```

People pass `*.txt` to `find -name` in quotes, which is right for `find`, and pass it unquoted to `rm`, which is the shell deleting matches. Those are different. A pattern that matches a name starting with a dash can look like a flag to the command. `--` before the pattern does not apply to names the pattern expands to if you put `--` in the wrong place. Put `--` on the command, then the glob. Hidden files need an explicit `.` pattern.

## Variables and the environment
Also asked as: environment variable; export; local variable; env; child inherits
A shell variable is a name in this shell. `export` marks it for the environment. A child process gets a copy of the environment, not a live link. Changing it in the child does not change the parent. `VAR=value command` sets the variable for that one command. `VAR=value` on its own sets it in the shell. Empty and unset are different. `${var-pattern}` fires only if unset. `${var:-pattern}` fires if unset or empty.

```sh
name='pattern' command -- file
```

People `export` everything and then cannot tell what a child depends on. Export what the child must see. A variable in single quotes is not expanded. A variable in the program text of `awk` or `sed` is not the shell variable unless you closed the quotes or passed `-v`. The environment is strings. It is not a place to hide a password from `ps` on every system. Command arguments are visible. Environment often is too.

## Current directory
Also asked as: cwd; pwd; cd; working directory; relative path; .
Every process has a current directory. A relative path starts there. `.` is that directory. `..` is its parent. `cd` changes it for this process only. A script's `cd` does not move your terminal. `/` at the start of a path ignores the current directory. `pwd -P` prints the path with symlinks resolved. `pwd -L` prints the path you walked, which may contain a symlink.

```sh
cd -- dir || exit 1
```

People `cd` and then delete, and the `cd` failed, and the delete runs in the old directory. Test the `cd`, or use `set -e`. A current directory that is removed out from under a process stays as the process's cwd until it `cd`s. `ls` with no operand lists that directory. It does not list `/`. The prompt's folder name is a courtesy. The kernel uses the inode the process holds.

## Permissions
Also asked as: mode bits; rwx; chmod meaning; directory execute; who can open
The mode is owner, group, other. `r` reads, `w` writes, `x` executes a file or searches a directory. The kernel picks one class. You are the owner, or in the group, or other. You are not two of them. A directory needs `x` to be entered and `r` to be listed. `w` on a directory creates and removes names. `w` on a file changes bytes. `rm` needs `w` on the directory.

```sh
ls -l -- file
```

People set `777` to make a problem go away and give every account write and execute. People set `644` on a directory and lock themselves out of it, then test as root and do not notice, because root bypasses the bits. The execute bit on a script is what lets the kernel start it. The shebang is what it starts. A mode is not an ACL. Linux ACLs can add entries `ls -l` only hints at with a `+`.

## Devices and /dev/null
Also asked as: /dev/null; discard output; character device; /dev/zero; special files
`/dev/null` is a character device. Writes succeed and disappear. Reads return end of file immediately. It is how you discard a stream. It is not a regular file, and it does not fill up. `/dev/zero` returns zero bytes on read. `/dev/fd/1` names a descriptor. These paths are files in the broad sense that you can open them. They are not disk files with a length.

```sh
command -- file >/dev/null 2>&1
```

People `rm /dev/null` and then recreate it as a regular file. Redirects start filling that file, and programs that expected a device misbehave. The fix is a device node, not an empty file. Permissions on `/dev/null` should allow everyone to read and write. A full disk is unrelated to `/dev/null`. Discarding output does not discard the status. The command can still fail.

## Mounts and filesystems
Also asked as: mount point; filesystem; df versus a folder; cross device; bind mount
A filesystem is a pool of inodes and blocks. A mount attaches it at a directory. Paths under that directory are on that filesystem until another mount covers them. `df` reports the pool. `du` walks names. A hard link cannot cross a mount. `mv` across a mount is a copy and a delete. A bind mount is the same filesystem at a second path. Filling one fills the other.

```sh
df -h -- dir
```

People add up `df` rows and double-count a bind mount. People `rm -r` a tree and enter a mount point they did not mean to, because a mount is a directory that can be walked. `--one-file-system` on GNU tools is the brake. The mount point directory and the filesystem mounted on it are different inodes. Unmount and the directory underneath is visible again. It was hidden, not deleted.

## What the shell expands first
Also asked as: expansion order; brace expansion; tilde; word split; when globs run
Bash expands braces first, then tildes, then parameters and command substitutions, then splits words, then globs. A variable that contains `*` globs after it is expanded, if it is unquoted. A tilde in quotes does not expand. A brace in quotes does not expand. Alias expansion happens on the first word, before this, in interactive shells. Scripts do not expand aliases unless you enable it.

```sh
printf '%s\n' ~/file
```

People put a pattern in a variable, quote the use, and the pattern never globs. That is correct for a filename and wrong for a pattern they meant to expand. People leave a pattern unquoted and a filename with a space becomes two globs. The order is why `"$file"` is the safe filename and the quotes have to be at the use. The shell does not re-expand the result of a quoted expansion. One pass.

## Background and job control
Also asked as: background; ampersand; jobs; fg; nohup; SIGHUP
`command &` runs the command in the background and does not wait. `$!` is its pid. The shell's status is the status of starting it, not of the job. `wait` collects the real status. A background job still has the terminal as its standard input unless you redirected it. On hangup, an interactive job can receive `SIGHUP` and die. `nohup` ignores that signal and redirects output if you did not.

```sh
command -- file >/dev/null 2>&1 &
wait "$!"
```

People background a server and close the terminal, and the server dies with the hangup. Redirect the streams and use `nohup`, or a service manager. People check `$?` right after `&` and think the job succeeded. That number is not the job. Two background jobs writing the same file interleave. `&` does not create a pipe. It only skips the wait. A script that exits waits for its children or they get the hangup treatment of that shell.
