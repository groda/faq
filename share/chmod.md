# chmod

`chmod` changes the permission bits on files and directories you name. It does not change the owner, that is `chown`, and it does not change the bytes inside a file. It does not apply your umask. The mode you give is the mode that is stored. With no file it does not read standard input. A directory operand is the directory itself. Contents change only if you pass `-R`. On Linux a symlink is not the thing that is updated. `chmod` follows it and changes the target. GNU chmod from coreutils on Linux is assumed here. macOS chmod understands octal modes, symbolic modes, and `-R`. It has `-h` to operate on the symlink itself, and it has no `--reference` and no `--preserve-root`. BusyBox chmod has octal and symbolic modes and usually `-R`, and it usually lacks `--reference`. Exit status is 0 when every named path was changed, or was already in the requested mode. It is non-zero when a path is missing, a mode is illegal, or a file could not be updated. A later command that still cannot read the file is a different problem. The bits and the owner together decide access.

Basic form: chmod MODE file

`MODE` is either an octal number, such as `644`, or a symbolic change, such as `u+x`. Operands after the mode are paths. A name that starts with `-` is a flag unless it comes after `--`. Quote nothing for `chmod` itself. The shell still expands globs before `chmod` sees them, and a glob does not match a leading dot. Symbolic letters are not files. `a`, `u`, `g`, and `o` are who the change applies to.

## Set an octal mode
Also asked as: chmod 644; chmod 755; numeric mode; octal permissions; chmod absolute mode
An octal mode replaces the permission bits. Each digit is owner, group, then everyone else. `4` is read, `2` is write, `1` is execute, and the digits add. `644` is owner read and write, group read, others read. `755` is that plus execute for everyone. The file's previous mode does not matter. This form sets the bits, it does not add to them.

```sh
chmod 644 -- file
```

People use `644` on a directory and then cannot open it. A directory needs execute to be entered. `644` on a directory is read and write without search, so the names inside cannot be looked up by a normal user. `755` is the usual directory mode. `666` and `777` are not "make it work." They give every account on the machine write, and `777` also gives every account execute. The leading digit people omit is the special bits. `0755` and `755` are the same mode.

## Add or remove a permission
Also asked as: chmod u+x; chmod symbolic mode; chmod go-w; add execute; remove write; chmod a+r
A symbolic mode changes the bits that are already there. `u` is the owner, `g` is the group, `o` is everyone else, and `a` is all three. `+` adds, `-` removes, and `=` sets that who-list to exactly the bits named. `r` is read, `w` is write, and `x` is execute. `chmod u+x` adds execute for the owner and leaves the other bits alone. Several changes can be comma-separated.

```sh
chmod u+x -- file
```

```sh
chmod go-w -- file
```

Use `u+x` when the owner must be allowed to execute a file, or enter a directory, and the rest of the mode should stay. Use `go-w` when group and others must not write. `a+x` is not "make it runnable for me." It adds execute for every class. `o` is not "owner." Owner is `u`. People write `chmod +x file` and it works, because a missing who-list means `a`, but it is broader than they think. An `=` with an empty right side clears that class. `chmod g= file` removes every group bit.

## Make a file executable
Also asked as: chmod +x; make a script executable; permission denied when running; chmod a+x; executable bit
The execute bit is what the kernel checks when you run a file by name. A script also needs a shebang and read permission, but without execute the kernel will not start it. `chmod a+x` sets execute for owner, group, and others. `chmod u+x` sets it only for the owner. The bytes of the script do not change. A later `./file` uses the new bit.

```sh
chmod a+x -- file
```

People set `644` and run the file, and the shell says permission denied. `644` is read and write, not execute. People also set execute on a directory and expect the files inside to become runnable. Directory execute means the directory may be entered. Each file has its own execute bit. `+x` on a symlink follows the link and marks the target, on GNU chmod. It does not make the symlink path special.

## Let people enter a directory
Also asked as: chmod 755 directory; directory execute bit; cannot cd; permission denied listing a folder; search bit
On a directory, `x` is the search bit. You need it to `cd` into the directory and to open files under it by name. `r` lets you list the names. `w` lets you create and delete names. A directory can be executable and not readable, in which case a known name can still be opened and `ls` cannot list it. The reverse, `r` without `x`, can list on some systems and still cannot open the files.

```sh
chmod 755 -- dir
```

`chmod 644` on a directory is the usual mistake. It looks like a normal file mode and it removes the search bit. Root can still walk a directory that a normal account cannot, so a test as root hides the bug. `755` gives everyone search and gives only the owner write. `750` keeps search for the owner and the group. Recursive `chmod -R 644` strips search from every subdirectory and the tree becomes unusable for anyone but root.

## Copy the mode of another file
Also asked as: chmod --reference; copy permissions from a file; same mode as; chmod reference; set mode from another file
`--reference` reads the mode of an existing file and applies that mode to the operands. It does not copy the owner, and it does not copy the data. Symbolic and octal modes are not also given. The flag is GNU. macOS and BusyBox do not have it.

```sh
chmod --reference=file -- file
```

People use it to copy a directory's mode onto a file and then cannot tell why the file is executable. The reference's bits are copied, including execute. A symlink reference follows the link on GNU chmod, so you copy the target's mode, not a mode stored on the link. If the reference cannot be read, chmod fails and the other files are not a reason to assume the copy happened.

## Change a whole tree
Also asked as: chmod -R; recursive chmod; chmod directory and contents; chmod -R 755; change permissions underneath
`-R` walks the directory and changes every file and subdirectory it can reach. The mode is applied to each entry. An octal mode replaces the bits on every one of them, files and directories alike. A symbolic change is applied to each entry's current bits. GNU chmod does not follow a symlink it finds while walking. It changes the link's target only when the symlink is an operand you named.

```sh
chmod -R go-w -- dir
```

`chmod -R 644 dir` is the dangerous form. Every subdirectory loses execute, and every script loses nothing it didn't already lack, but the directories become unlistable for a normal user. Prefer a symbolic change that means the same thing on files and directories, or use `X` so execute is added only where it belongs. `-R` on `/` is allowed on GNU chmod unless you pass `--preserve-root`. `--preserve-root` makes a recursive operation on `/` fail. The default is `--no-preserve-root`.

## Add execute only where it already belongs
Also asked as: chmod +X; capital X; chmod -R u+rwX; execute only on directories; conditional execute
Capital `X` adds execute only if the entry is a directory, or if some execute bit is already set. Lowercase `x` adds execute to everything. `u+rwX` is the recursive form that gives the owner read and write, and execute on directories and on files that are already executable, without marking every data file executable.

```sh
chmod -R u+rwX -- dir
```

People write `chmod -R 755` to "fix a tree" and every image and source file becomes executable. `X` is the letter that avoids that. It is not a separate flag. It is a mode character, and it is easy to type as `x`. On a file with no execute bit, `X` adds nothing. On a directory, it adds search. A script that was never executable stays non-executable. You still need `u+x` or `a+x` on that file itself.

## Set the setuid, setgid, or sticky bit
Also asked as: chmod 4755; chmod u+s; setuid bit; chmod g+s; chmod +t; sticky bit on a directory
`s` on the owner's execute position is setuid. `s` on the group's execute position is setgid. `t` on the other execute position is the sticky bit. Octal puts them in a fourth leading digit. `4` is setuid, `2` is setgid, `1` is sticky. `4755` is setuid plus `755`. `1777` is sticky plus `777`, the mode of a typical `/tmp`. The special bit does not replace the execute bit under it. `chmod u+s` adds setuid and leaves execute as it was.

```sh
chmod u+s -- file
```

```sh
chmod +t -- dir
```

Use `u+s` only when a program must run with the file owner's rights. Use `+t` on a directory when names there should be deletable only by the owner of the name, or by the directory's owner. People set `4777` and give every account a setuid file, which is a different and worse mode than `755`. On a directory, setgid means new files inherit the directory's group, on Linux. It does not mean "group can read." Group read is `g+r`. GNU `ls` shows `s` or `t`, and `S` or `T` when the execute bit under that letter is off.

## See what chmod changed
Also asked as: chmod -v; chmod -c; verbose chmod; report only changes; chmod --changes
`-v` prints a line for every operand it processes, whether or not the mode changed. `-c` prints a line only when the mode actually changed. Neither flag changes the mode. Both are GNU-friendly coreutils options. BusyBox often has `-v` and may not have `-c`. The text is a diagnostic, not a script format.

```sh
chmod -c u+x -- file
```

People read `-c` as "check" or as "continue." It only filters the message. A missing file is still an error and a non-zero status. `-f` is the flag that suppresses most error messages, and it does not mean the change succeeded. `-v` on a recursive walk prints one line per entry, which is a lot of output and does not make the walk safer. Use `-c` when you want to see the files whose bits moved.

## Hide errors
Also asked as: chmod -f; chmod silent; chmod quiet; suppress chmod errors; ignore missing files
`-f` suppresses most error messages. A missing file, or a file you cannot change, does not produce the usual diagnostic. The status can still be non-zero. `-f` does not force the mode onto a file you do not own. It does not create the file. It is the quiet flag, not a success flag.

```sh
chmod -f 644 -- file
```

People add `-f` to a recursive command and then cannot tell which part of the tree was skipped. The messages were the list of failures. Use `-f` only when a missing operand is expected and the status is checked some other way. `-f` is not `--reference` and it is not "force" in the sense of crossing a permission you lack. Root can change modes that a normal user cannot, with or without `-f`.

## Do not walk the root directory by accident
Also asked as: chmod --preserve-root; chmod -R /; do not chmod root; --no-preserve-root; recursive on slash
GNU chmod will recursively operate on `/` if you ask it to. `--preserve-root` makes `chmod -R` on `/` fail instead. `--no-preserve-root` is the default. The flag matters only for a recursive operation whose operand is the root directory. It does not protect other directories.

```sh
chmod --preserve-root -R go-w -- dir
```

A variable that expands empty in `chmod -R -- $dir` can become `chmod -R --` and then fail, or, if `/` is what the variable was supposed to hold and the protection is off, walk the system. `--` does not mean "preserve root." It only ends options. macOS chmod does not have `--preserve-root`. The protection is a GNU flag, and it is off until you turn it on. Name the directory you mean.

## Change who a symbolic mode applies to
Also asked as: chmod ugo; chmod a; chmod o is not owner; who letters; chmod g+w; chmod u-s
`u`, `g`, and `o` choose the class. `a` means all three. You can stack letters. `ug+w` adds write for owner and group. A symbolic mode with no letters, `+r`, is `a+r`. `o` is other, the accounts that are neither the owner nor the group. It is the most common letter to get backwards.

```sh
chmod ug+rw -- file
```

`chmod o+x` does not make the file executable for its owner. The owner's bit is `u`. A file can be executable by others and not by its owner. Access then depends on which class the kernel puts you in. The owner is always tested as owner, not as other. Copying a class uses the class letter on the right. `chmod g=u` gives the group the same `rwx` bits the owner has. It does not copy setuid.

## Set one class to exactly these bits
Also asked as: chmod u=rwx; chmod equals; set rather than add; chmod a=r; replace one class of bits
`=` in a symbolic mode sets that class to exactly the bits named and clears the other permission bits of that class. `u=rwx` gives the owner read, write, and execute, and removes setuid from the owner class if it was expressed there. `a=r` gives everyone read and removes write and execute from everyone. It does not touch the special bits the way a four-digit octal mode does, except where the letters include them.

```sh
chmod u=rwx,go=rx -- file
```

People use `+` and then wonder why an old write bit is still there. `+` only adds. `=` is the symbolic form of "this class should look like this." An octal mode is still the blunt tool when you want the whole mode replaced in one number. `u=rwx` leaves the group bits as they were. Say `go=` as well, or use `755`, if those classes must change too.

## What a failed chmod means
Also asked as: chmod exit status; chmod operation not permitted; chmod failed; missing file exit code; did chmod work
`chmod` exits 0 when it updated every operand, including an operand that already had the requested mode. It exits non-zero when any operand could not be changed. A missing path is a failure. A file owned by someone else is a failure for a normal user, and the message is on stderr. `-f` hides the message and does not turn the status into success.

```sh
chmod 644 -- file
```

People check the new mode with `ls -l` as root and conclude the command worked for everyone. Root bypasses the directory search bit, so a directory you just made unlistable still opens for root. The status of `chmod` says whether the bits were stored, not whether a given account can use the file. Owner and group still apply. `chmod` does not print the new mode unless you passed `-v` or `-c`.
