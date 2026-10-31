# cp

`cp` copies files. It reads a source and writes a new file, or it writes into a directory you named last. It does not move the source. That is `mv`. It does not merge two directories' contents in a special archive format. A directory source is an error unless you pass `-R`. With no operand it does not read standard input. GNU cp from coreutils on Linux is assumed here. A plain copy follows a symlink and copies the target's bytes. The new file's owner is you. Its mode is the source mode minus the umask, not a full copy of timestamps and owner, unless you pass `-p` or `-a`. macOS cp understands `-R`, `-p`, and `-a` in a BSD sense that is not identical to GNU `--preserve=all`. It has no `--reflink`. BusyBox cp usually has `-R`, `-p`, and `-a` as a thinner archive copy. Exit status is 0 when every source was copied. It is non-zero when a source is missing, a directory was named without `-R`, or a destination could not be written. An overwrite is not a failure.

Basic form: cp -- file file

Two operands are source and destination. More than two, and the last must be an existing directory. Each earlier operand is copied into it. A name that starts with `-` is a flag unless it comes after `--`. The shell expands globs before `cp` sees them. Quote a name with spaces. If the destination exists and is a file, `cp` overwrites the bytes. It does not ask, unless you pass `-i`.

## Copy a file
Also asked as: how to copy a file; cp source dest; duplicate a file; copy to a new name; cp one file
With two file operands, `cp` creates the destination if it is missing and overwrites it if it is a regular file. The source is unchanged. The new file is owned by you. Timestamps are not kept. A missing source is an error and a non-zero status. A destination directory that does not exist is not created when you named a path ending in a new directory. `cp` does not create parent directories.

```sh
cp -- file file
```

People use `cp` and then compare `ls -l` and wonder why the times differ. A plain copy does not preserve timestamps. `-p` does. People also copy a file onto a directory and think they replaced the directory. If the destination is a directory, the file is copied into it, under the same base name. `cp -- file dir` writes `dir/file`. It does not make `dir` become the file.

## Copy into a directory
Also asked as: cp into a folder; copy several files; last argument is the directory; cp files dir; more than two operands
When the last operand is a directory, every earlier operand is copied into it. The name inside the directory is the source's base name. Several sources are legal only in this form. Two operands where the second is not a directory is the rename form, not the into-directory form. A trailing slash on the destination does not create it.

```sh
cp -- file file dir
```

People `cp file dir/new` and `dir` does not exist. That is a file copy to a path whose parent is missing, and it fails. `cp` will not make `dir`. `mkdir` first, or copy to `.` and move. A glob that matches nothing is left literal by bash, and `cp` looks for a file named `*.txt`. A glob that matches one file, with a destination that does not exist, is the two-operand form and may create a file with an unexpected name.

## Copy a directory tree
Also asked as: cp -r; cp -R; recursive copy; copy a folder; cp directory; duplicate a tree
`-R` copies a directory, its files, and its subdirectories. `-r` is the same on GNU cp. Without it, a directory source is an error. The copy is a new tree. Hard links inside the source become separate files, unless you used `-a`. A symlink is copied as a symlink if you used `-a` or `-P`, and as the target's bytes if you used a plain `-R` on GNU cp. That default is the usual surprise.

```sh
cp -R -- dir dir
```

If `dir` already exists, the source is copied inside it, so you get `dir/dir`. If it does not exist, it is created as the copy. People run the command twice and then have a nested copy. `-R` does not mean "merge, and delete extras." Files in the destination that are not in the source stay. `rsync -a --delete` is the tool when the destination must become an exact mirror. `cp` only copies.

## Keep mode, owner, and times
Also asked as: cp -p; cp -a; preserve timestamps; archive copy; copy permissions; cp --preserve
`-p` keeps the mode, the owner if you are allowed to, and the timestamps. `-a` is archive mode. On GNU cp it is `-dR --preserve=all`. It copies the tree, keeps attributes, and does not follow symlinks. Owner is preserved only if you are root, or the owner would not change. A plain `cp` drops all of that on purpose.

```sh
cp -a -- dir dir
```

People use `-p` on a directory and still fail, because `-p` does not imply `-R`. Add `-R`, or use `-a`. People use `-a` as root and also copy numeric owners onto a machine where those ids are different users. Preserve is literal. It does not map names. `--preserve=mode,timestamps` is the GNU list when you want times and bits but not owner. Links are a separate attribute. `-a` includes them.

## Do not overwrite an existing file
Also asked as: cp -n; no clobber; do not overwrite; cp --no-clobber; skip if dest exists
`-n` does not overwrite an existing destination file. The source is not copied over it. GNU cp exits non-zero if you used `-n` and skipped a file, on current coreutils. Older GNU cp exited 0. Do not depend on the status as the only check. `-i` asks instead. `-f` is a different flag. It does not mean "overwrite." It means "if the destination cannot be opened, unlink it and try again."

```sh
cp -n -- file file
```

People pass `-n` and think a same-named file was updated. It was left as it was. `-u` is the flag that copies only when the source is newer, or the destination is missing. `-i` and `-n` override each other. The last one on the command line wins. macOS `cp -n` exists on current systems and is the skip. It is not a merge. A directory copy with `-n` still creates files that are not already there.

## Ask before overwriting
Also asked as: cp -i; interactive copy; prompt before overwrite; confirm overwrite; cp alias -i
`-i` prompts before overwriting an existing destination. A new destination does not prompt. Answering no leaves the destination. An alias `cp -i` is why an interactive `cp` asks and a script does not. Scripts do not expand aliases unless the shell was told to. `-f` after `-i` does not restore a prompt. `-i` after `-n` turns the prompt back on.

```sh
cp -i -- file file
```

A "no" answer is not a failed copy in a way you should script. Check the destination if it matters. `-i` on a recursive copy asks for existing files, which is a lot of prompts and not a review of the tree. It does not ask before creating a new file. People expect `-i` to confirm the whole command. It confirms overwrites.

## Copy the symlink, or copy the target
Also asked as: cp symlink; cp -P; cp -L; follow symlinks; copy the link itself; dereference
A plain `cp` of a symlink copies the file it points at, and the destination is a regular file. `-P` copies the symlink itself. `-L` follows symlinks in the source, including ones found while walking a tree. `-H` follows only a symlink that you named on the command line. `-a` includes "do not follow," because it includes `-d`.

```sh
cp -P -- file file
```

```sh
cp -L -- file file
```

Use `-P` when the link is the thing you are copying. Use `-L` when you want the bytes, even if the tree is full of links. A dangling symlink fails under `-L` and copies as a dangling symlink under `-P`. People `cp -a` a tree and then think the links are broken because they assumed the targets were copied in. The targets were not, unless they sat inside the tree and were named there too.

## Hard link instead of copying bytes
Also asked as: cp -l; hard link copy; copy without duplicating data; cp --link; same inode
`-l` makes a hard link. The destination is another name for the same inode. No bytes are copied. Edit one name and you edit the other. A hard link cannot cross filesystems, and it cannot point at a directory. In a recursive copy, `-l` links each file. GNU `cp -al` is the archive form of that, used for snapshot-style trees.

```sh
cp -l -- file file
```

People use `-l` to save space and then change one copy, and the other changes too. That is the link. `cp` without `-l` is the independent copy. A later `rm` of one name leaves the other. `-l` is not `-s`. `-s` makes a symlink, which can dangle and can cross filesystems. `-l` cannot. If the destination exists, `cp` still has to be allowed to replace it. `-n` will refuse.

## Make a symlink instead of a copy
Also asked as: cp -s; cp symbolic link; link instead of copy; cp --symbolic-link; relative symlink
`-s` creates a symlink to the source instead of copying bytes. The destination names the source path. A relative source stays relative, which breaks if you move the destination. An absolute source survives a move of the destination and breaks if you move the source. This is GNU and common. It is not a copy of the content.

```sh
cp -s -- file file
```

People use `-s` inside a backup and the backup contains pointers back at the live tree. That is what they asked for. Use a plain `cp` or `-a` when the backup must hold bytes. `-s` fails across the cases where `symlink` fails, including an existing destination it cannot replace. It does not follow a source that is already a symlink in a special helpful way. You get a link to the path you named.

## Update only when the source is newer
Also asked as: cp -u; update copy; copy if newer; cp --update; skip older source
`-u` copies when the destination is missing, or when the source is newer than the destination. An older source does not overwrite a newer destination. The comparison is the modification time. It is not a content compare. Equal timestamps do not force a copy. This is GNU. macOS cp often has no `-u`.

```sh
cp -u -- file file
```

People use `-u` as a sync. It does not delete files that disappeared from the source, and it does not notice a source that changed without a newer timestamp. A copy that preserved times, then a second `-u` run, can skip files whose timestamps were set back. `-a` preserves times. Combine them only if that is the comparison you want. `rsync -a` is the tool when the rule is more than "newer mtime."

## Copy into a directory you named with a flag
Also asked as: cp -t; target directory; cp --target-directory; xargs cp; destination first
`-t` names the destination directory. The other operands are sources. This is the form `xargs` can call, because `xargs` appends names. `-T` is the opposite. It forces the two-operand form, so a directory destination is not treated as a container. Both are GNU. macOS cp does not have `-t`.

```sh
cp -t dir -- file file
```

People let `xargs` append a destination and `cp` reads it as another source. The last argument became a source, and the copy failed or wrote the wrong way. `-t dir` fixes the order. `-T` is the guard when you always want source and destination as two files. `cp -T -- file dir` fails if `dir` is a directory, instead of copying `file` into it. Use that when "into" would be the bug.

## Keep the relative path under the destination
Also asked as: cp --parents; recreate subdirectories; copy with path; parent directories; not just the basename
`--parents` uses the source path under the destination directory. `cp --parents -- dir/file dest` writes `dest/dir/file`, creating `dest/dir`. Without it, the same command writes `dest/file` if `dest` is a directory. The flag is GNU. The destination must be a directory.

```sh
cp --parents -- dir/file dir
```

People want the folders recreated and `cp -R` of the top directory is the ordinary way to get them. `--parents` is for a list of files whose relative paths should be kept. A source of `../file` can create an ugly relative path under the destination. The flag does not rewrite `..`. It copies the path text you gave. Combine it with a path you are willing to see recreated.

## Copy sparsely, or force a full copy
Also asked as: cp sparse; cp --sparse; holes in a file; cp --reflink; copy on write
A sparse file has holes. GNU cp can keep those holes, so the destination does not allocate blocks for the zeros. `--sparse=always` makes a hole where the source has a long run of zeros. `--sparse=never` writes the zeros out. `--reflink=always` asks for a copy-on-write clone and fails if the filesystem cannot do it. `--reflink=auto` clones if it can and copies if it cannot. These flags are GNU.

```sh
cp --sparse=always -- file file
```

People copy a VM image and the destination is suddenly huge. The holes were filled in. `--sparse=always` is the request to keep them. People use `--reflink=always` on a filesystem without clone support and the copy fails. `auto` is the tolerant form. A reflink shares blocks until one side is written. It is not a hard link. The two names can diverge. `df` may not drop by the full size until the shares are broken.

## What a failed cp means
Also asked as: cp exit status; cp exit code; cannot create; permission denied; cp failed; missing destination directory
`cp` exits 0 when it copied every source. A missing source is non-zero. A directory without `-R` is non-zero. A destination directory that does not exist is non-zero. Permission denied on the destination directory is non-zero. Overwriting a file you are allowed to write is success, not a warning. `-v` prints each file it copies. It does not turn a failure into success.

```sh
cp -R -- dir dir
```

People check that the destination exists and treat that as a good copy. A failed `cp` can leave a partial file. A recursive copy can have created the first files and failed on a later one. The status is the only summary. A same-filesystem `mv` is the cheaper rename when you do not need the source to remain. `cp` always reads and writes the bytes, unless a reflink avoided that. Same filesystem does not make `cp` a rename.
