# du

`du` adds up the space used by files and directories. It walks a directory recursively. It does not report free space on a filesystem. That is `df`. With no operand it starts at `.`. A file operand is that file. A directory operand is the tree under it. The default number is allocated disk space in 1024-byte blocks, not the byte length you see in `ls -l`. Hard links are counted once. Symlinks are not followed. GNU du from coreutils on Linux is assumed here. macOS du understands `-h`, `-s`, `-d`, and `-a`, and it does not have GNU `--exclude`, `--apparent-size`, or `--files0-from`. BusyBox du usually has `-h`, `-s`, and `-a`, and it usually lacks the long filters. Exit status is 0 when every named path was walked. It is non-zero when a path is missing or a directory cannot be read. The numbers it already printed are not rolled back.

Basic form: du dir

Operands are paths. A name that starts with `-` is a flag unless it comes after `--`. A glob is expanded by the shell first, and it does not match a leading dot. Quote a pattern you pass to `--exclude`. Default units are 1024-byte blocks, or 512-byte blocks if `POSIXLY_CORRECT` is set. `-h` is the form people read. `-s` is the form that prints one total.

## Summarize a directory
Also asked as: how much space does a directory use; du a folder; disk usage of a tree; du default output; blocks used
With a directory operand, `du` prints one line per subdirectory and a final line for the directory itself. The number is allocated space, in 1024-byte blocks unless you change the unit. The directory line includes the subdirectories. It is not an extra charge on top of them. The path is printed after the number, with no columns and no header.

```sh
du -- dir
```

People add the lines and double-count. The last line is already the total of what was walked. A file smaller than the block size still costs a block, so a 6-byte file can show as 4. `du` does not see a directory it cannot enter. It warns on stderr, skips that branch, and exits non-zero. Hidden names are included. `du` itself walks the tree. The shell glob is not required.

## Print a size people can read
Also asked as: du -h; human readable du; du in megabytes; du -sh; powers of 1024
`-h` prints powers of 1024, such as `K`, `M`, and `G`. The numbers are rounded. `1.0G` is not an exact byte count, and it is not 1000 million bytes. `-h` does not change what is walked or whether subdirectories get their own lines. Combine it with `-s` when you want one readable total.

```sh
du -h -- dir
```

People compare `du -h` with `df -H` or with a disk label in decimal GB. Those are powers of 1000. GNU `du` uses `--si` for that scale. `-h` matches GNU `df -h` and `ls -h`. A rounded `4.0K` can be one 4096-byte block. It is not proof the file contains 4096 bytes of data.

## Print only the total
Also asked as: du -s; du -sh; summarize; only one number; do not list subdirectories
`-s` prints one line per operand, the total for that tree, and does not print a line for each subdirectory. `-h` still belongs with it if you want a unit suffix. This is the command people mean by "how big is this directory."

```sh
du -sh -- dir
```

Without `-s`, a large tree prints a line per directory and the total is the last line, not the sum of the lines. `-s` on a file prints that file. `-s` on several paths prints one line each, not one line for all of them. The grand total of several operands is `-c`. A missing operand is still an error. `-s` does not hide the warning.

## Count files, not only directories
Also asked as: du -a; all files; size of each file; du include files; per-file disk usage
`-a` prints a line for each file as well as each directory. Directory lines still include the files under them. The file line is that file's own allocated size. This is how you find a large file without a second tool. It does not change the unit. Add `-h` to read it.

```sh
du -ah -- dir
```

The output is not a sorted list. The largest file is not last. Pipe to `sort -h` if you want order, and remember that `-h` suffixes need `sort -h`, not `sort -n`. `-a` does not follow symlinks. A symlink is listed as a symlink, with the size of the link itself, unless you pass `-L`. Hidden files are included, because `du` walks the directory rather than trusting a shell glob.

## Stop at a depth
Also asked as: du -d; du --max-depth; only one level; du depth; do not list deep folders; du -d 1
`-d` limits how deep a printed line may be. `--max-depth` is the same flag. `-d 1` prints the starting directory and the directories directly inside it, not their children as separate lines. The size on each line still includes everything under that directory. The walk is not shallower. The printing is. `-d 0` is the same as `-s`.

```sh
du -h -d 1 -- dir
```

People use `-d 1` to mean "do not count files deeper than this." The count still includes them. Only the extra lines are suppressed. A depth is relative to the operand, not to `/`. `-d` is GNU and on macOS. BusyBox often has it. It does not exclude a name. A large child still shows up inside its parent's number.

## Measure bytes instead of disk blocks
Also asked as: du -b; du --bytes; apparent size in bytes; file size not blocks; du matches ls -l
`-b` prints apparent size in bytes. Apparent size is the length of the file, the number `ls -l` shows, not the blocks allocated on disk. A 6-byte file prints as 6. A sparse file can print as larger than the disk space it occupies. `-b` is GNU and is defined as `--apparent-size` plus a block size of 1 byte.

```sh
du -b -- file
```

`--apparent-size` without `-b` still rounds up to the current block size, so a short file can print as 1, meaning one block, not one byte. People pass `--apparent-size` expecting bytes. Use `-b` for bytes. `-b` on a directory is the sum of apparent sizes, which is not the size the directory would take in a `tar` that stores holes differently, and not the allocated total `du` prints by default.

## See allocated space, not the byte length
Also asked as: du disk usage; allocated blocks; sparse file; du smaller than ls; why du and ls disagree
The default `du` number is device usage. A sparse file whose logical length is 1 GB and whose allocated space is 4 KB shows as about 4 KB. `ls -l` shows the logical length. Holes do not consume blocks. The other direction happens too. A short file still allocates a whole block, so `du` can be larger than the byte length.

```sh
du -h -- file
```

```sh
du -b -- file
```

Use the first form for "what did this cost on disk." Use the second for "how long is the file." People copy a sparse file the wrong way and the copy stops being sparse. `du` then jumps. That is the copy, not `du` changing its mind. Directory totals follow the same rule. Default `du` adds allocated blocks. `-b` adds lengths.

## Stay on one filesystem
Also asked as: du -x; one file system; do not cross mounts; du skip other filesystems; du -x /
`-x` does not enter a directory that is on a different filesystem from the one where the walk started. A mount point inside the tree is not descended. Its contents are not added. The mount point directory itself may still appear as a small local directory. This is how you measure the root filesystem without counting other disks mounted under it.

```sh
du -xh -- dir
```

Without `-x`, `du /` walks every mounted filesystem it can enter, including network mounts, and can hang on a stuck one. `-x` does not skip a directory that is merely large. It skips a different filesystem. Bind mounts of the same filesystem are still the same filesystem and are still walked. `df -h -- dir` tells you which filesystem the start path is on.

## Skip names that match a pattern
Also asked as: du --exclude; exclude a directory; du skip node_modules; exclude a glob; du -X; exclude-from
`--exclude` takes a shell glob and skips matching names while walking. A matching directory is not entered, so its contents are not counted. Quote the pattern, or the shell expands it. `-X` reads globs from a file, one per line. Both are GNU. macOS du does not have them.

```sh
du -sh --exclude='pattern' -- dir
```

The pattern matches the name, not a full path you have to anchor, so `*.o` skips object files at every level. It also skips a directory whose own name matches. `--exclude` does not subtract a size you already computed. It changes the walk. A file you named on the command line is still measured even if it matches, because the exclude applies to names found while walking, and an operand is what you asked for.

## Add a grand total
Also asked as: du -c; du --total; total of several directories; grand total line; du -ch
`-c` prints an extra final line, `total`, that adds the operands. Each operand still gets its own line. Without `-c`, several directories are not summed. `-h` applies to the total too. A subdirectory line is not an operand. Do not pass `-c` and also add the lines yourself.

```sh
du -ch -- dir dir
```

`-c` adds what `du` counted. If two operands are the same tree, or one is inside the other, the total double-counts. `-c` does not unique the paths. Hard links already counted once inside one walk can be counted again in a second operand. The word `total` is a label. A directory literally named `total` is a different line.

## Count a hard link every time
Also asked as: du -l; count links; hard link counted twice; du count-links; shared inode
By default `du` counts a hard-linked file once per walk. The second name adds nothing, because the blocks belong to one inode. `-l` counts each name. The total can then be larger than the filesystem usage `df` reports for those files. `-l` does not follow symlinks. A symlink is not a hard link.

```sh
du -lh -- dir
```

People see `du` smaller than the sum of `ls -l` on a tree full of hard links and think files were skipped. They were counted once. Use `-l` only when you want a sum of names rather than a sum of disk blocks. Backup tools that store each link as a full file are the case for `-l`. Disk-usage questions are the case for the default.

## Follow symlinks
Also asked as: du -L; du -H; follow symlinks; dereference; du counts the link target; du -D
The default, `-P`, does not follow symlinks. A symlink counts as the small size of the link itself. `-L` follows every symlink and counts the target, and it can walk out of the tree and loop. `-H` follows a symlink only when it is an operand you named, not when it is found inside a directory. GNU also spells `-H` as `-D`.

```sh
du -sh -- dir
```

```sh
du -shL -- dir
```

Use the first form for the space the directory entries cost. Use `-L` when you mean the files the links point at and you know there is no cycle. A broken symlink counts as a link in the default mode and as an error under `-L`. `-L` can make `du` larger than the filesystem you started on, because the target may be on another disk. Combine it with `-x` if that is not what you want.

## Count inodes instead of blocks
Also asked as: du --inodes; how many files in a tree; inode usage; count files with du; number of inodes
`--inodes` prints how many directory entries `du` saw, instead of how many blocks they use. A file and a subdirectory each consume an inode. This is the walk's inode count, not the filesystem's free-inode count. Free inodes are `df -i`. The flag is GNU.

```sh
du --inodes -d 1 -- dir
```

People use it as a file counter and forget that directories are inodes too. The number is not "regular files only." `-a` still changes whether files get their own lines. `--inodes` does not make the walk faster in a useful way on a huge tree. You still visit the names. A hard link is one inode and may be counted once, the same way blocks are.

## Hide small entries
Also asked as: du -t; du --threshold; only show large directories; du bigger than; exclude small lines
`--threshold` hides printed lines whose size is smaller than the given size. A negative size hides lines larger than that size. The size uses the same units as the rest of `du`, so `-t 100M` with `-h` still means 100 megabytes of 1024, not a string compare against the printed suffix. This is GNU.

```sh
du -h -t 100M -- dir
```

The threshold filters output. It does not stop the walk, and a directory line that survives still includes the small files under it. A child that was hidden is not subtracted. People pass `-t 1G` looking for one large file and get a directory whose children are all small. Add `-a` if the lines you want are files. `--threshold` is not `--exclude`. One filters size. The other filters names.

## Read names from a NUL-separated list
Also asked as: du --files0-from; du -0; null separated names; du from find -print0; filenames with newlines
`--files0-from` reads a list of paths separated by NUL bytes and summarizes each. `-` as the list name means standard input. `-0` is a different flag. It makes `du` end its own output lines with NUL. GNU has both. macOS and BusyBox generally do not. Quote nothing in the NUL stream. The names are raw.

```sh
du -sh --files0-from=-
```

A newline-separated list breaks on a name that contains a newline, and `du` then measures the wrong path or misses it. Build the list with `find -print0`. `--files0-from` does not also turn on NUL output. Add `-0` if the next tool must read `du`'s lines safely. Operands on the command line are not mixed with the list in a useful way. The list is the input.

## Do not include subdirectory totals in a directory line
Also asked as: du -S; separate dirs; exclusive size; size excluding subdirectories; du apparent directory only
`-S` makes a directory line exclude the size of its subdirectories. The line is then the files directly in that directory, plus the directory itself, not the whole tree. Subdirectory lines still appear and carry their own exclusive size. The numbers no longer nest. Adding them up is how you recover the total.

```sh
du -Sh -- dir
```

The default line includes children, so it is larger than the files you see in that one folder. `-S` is the other question. People use it and then think the top directory shrank. The children moved to their own lines. `-S` is not `-s`. `-s` prints one total and hides the children. `-S` prints the children and stops adding them into the parent.

## Set the block size
Also asked as: du -B; du -k; du -m; block size; DU_BLOCK_SIZE; du in megabytes exactly
`-k` prints 1024-byte blocks, which is the GNU default. `-m` prints 1024-based megabyte blocks. `-B` sets an explicit size, as in `-BM`. The number is a count of those blocks, rounded up, unless you also asked for apparent bytes with `-b`. `DU_BLOCK_SIZE`, `BLOCK_SIZE`, and `BLOCKSIZE` change the default. `-h` overrides them.

```sh
du -k -- dir
```

`POSIXLY_CORRECT` switches the default to 512-byte blocks, so the same tree prints numbers twice as large. Pass `-k` in a script if the consumer expects 1024. `-m` rounds a small directory up to 1. It is not `-h`. A suffix of `MB` on `-B` is decimal. `M` is binary. The same rule as the rest of coreutils.

## Why du and df disagree
Also asked as: du versus df; disk usage versus disk free; deleted file still uses space; du smaller than df used; open file deleted
`df` reads filesystem totals. `du` walks trees it can enter and adds what it sees. A file that was deleted but is still open counts in `df` and not in `du`, because the directory entry is gone and the blocks remain until the last process closes it. A directory `du` cannot enter counts in `df` and is missing from `du`. Another tree on the same filesystem is in `df` and not in this `du`.

```sh
du -shx -- dir
```

Use `du -shx` for one tree on one filesystem. Use `df -h -- dir` for free space on the filesystem that holds it. Neither number is wrong. They are different measurements. Reserved blocks show up in `df` and not as files in `du`. A sparse file is small in default `du` and large in `ls -l`. Match the flag to the question before treating the gap as missing data.
