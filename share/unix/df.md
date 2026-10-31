# df

`df` reports free space on mounted filesystems. It does not add up the files in a directory. That is `du`. With no operand it lists every filesystem it can see. With a path, it lists the filesystem that contains that path, not the size of the directory. The `Size` column is the filesystem. `Used` and `Avail` are that filesystem's usage. A bind mount of the same device appears again, once per mount. GNU df from coreutils on Linux is assumed here. macOS df understands `-h`, `-i`, and `-P`, and it does not have GNU `--output` or `--total`. BusyBox df usually has `-h` and `-P` and often lacks the long filters. Exit status is 0 when the listed filesystems were read. It is non-zero when a named path cannot be stated or a filesystem's stats cannot be read. An empty `Avail` column is not the failure. The status is.

Basic form: df

Operands are paths, not filesystem types. A type filter is `-t` or `-x`. A name that starts with `-` is a flag unless it comes after `--`. Quote nothing for `df` itself. The shell still expands globs, so `df *` asks about every name in the directory and prints the filesystem of each, which is almost never what you wanted. Default units are 1024-byte blocks, or 512-byte blocks if `POSIXLY_CORRECT` is set. `-h` is the form people read.

## Show free space on mounted filesystems
Also asked as: how much disk space is free; df with no arguments; list filesystems; df default columns; mounted filesystems
With no operand, `df` prints one row per mounted filesystem it is willing to show. The columns are the source, the size, the space used, the space available, the percent used, and the mount point. Pseudo-filesystems such as `proc` and `sysfs` are omitted unless you pass `-a`. The numbers are 1024-byte blocks unless you ask for a unit.

```sh
df
```

People read a row as the size of that directory. The mount point is where the filesystem is attached. The size is the whole filesystem, shared by every directory on it. Two mount points on the same device are one pool of space, printed twice. `df` does not see a filesystem that is not mounted. A path on an unmounted disk is not a row until something mounts it.

## Print sizes people can read
Also asked as: df -h; human readable df; df in gigabytes; powers of 1024; df -h /
`-h` prints sizes in powers of 1024, such as `K`, `M`, and `G`. `1.0G` is 1024 megabytes of 1024 bytes, not 1000 million bytes. The numbers are rounded. A filesystem that is almost a gibibyte can print as `1.0G` used and `1.0G` size while still having space left. `-h` does not change which filesystems are listed.

```sh
df -h
```

People compare `df -h` to a disk sold as 500 GB and think the disk is missing space. The label on the drive is decimal. `-h` is binary. The other gap is filesystem metadata and reserved blocks, which `-h` does not add back. Use `-H` if you want powers of 1000. Use `-h` if you want the same scale as GNU `ls -h` and `du -h`.

## Print sizes in powers of 1000
Also asked as: df -H; df --si; SI units; decimal gigabytes; powers of 1000; df marketing units
`-H` prints powers of 1000. GNU also accepts `--si` for the same scale. `1G` in this mode is 1 000 000 000 bytes, closer to a drive vendor's GB than `-h` is. The column layout does not change. The flag is easy to mix with `-h`, and the two are not equal.

```sh
df -H
```

Use `-H` when you are comparing with a disk label or a network quota counted in decimal bytes. Use `-h` when you are comparing with `du -h`. A script that parses the suffix cannot accept both. The suffix letter is the same. Only the base changes. macOS `df -H` is not guaranteed to be this flag. On GNU it is the SI form.

## Ask about the filesystem that holds a path
Also asked as: df a directory; df a file; which filesystem is this path on; df /var; space left for a folder
A path operand selects the filesystem that contains that path. `df` prints that filesystem's free space, not the sum of the files under the path. Several paths on the same filesystem produce several identical rows. A relative path is resolved from the current directory.

```sh
df -h -- file
```

People run `df dir` to see how much `dir` uses. That number is `du -sh -- dir`. `df` cannot answer it. `df` also cannot answer "how much would this filesystem have if I deleted `dir`" without a `du` of what is actually stored there. A symlink is followed to the target's filesystem. A missing path is an error and a non-zero status, not a zero-size filesystem.

## See why used and available do not add up
Also asked as: df used plus avail is not size; reserved blocks; 5 percent reserved; ext4 reserved space; Use% does not match used over size
On ext-family filesystems the kernel keeps a reserve, often 5 percent, that a normal user cannot fill. `Size` includes it. `Avail` does not. Root can still allocate from the reserve, so a disk at 100 percent for you can still accept a write from root. `Used` plus `Avail` is therefore smaller than `Size`, and the gap is not a hidden directory.

```sh
df -h -- file
```

`Use%` is used space divided by used plus available, not used divided by size. A filesystem can show 100 percent while `Size` still looks larger than `Used`. Deleting files as a normal user frees `Avail`. It does not shrink the reserved percentage. That percentage is a filesystem setting, not a `df` flag. `df` only prints what the filesystem reports.

## Show inode use instead of block use
Also asked as: df -i; out of inodes; IUse%; no space left on device but df shows space; inode table full
`-i` prints inode counts instead of block counts. The columns are how many inodes the filesystem has, how many are used, how many are free, and the percent used. Each file, directory, and symlink consumes an inode. A filesystem can have block space left and zero free inodes. Creating a file then fails with "No space left on device" while `df -h` still shows room.

```sh
df -i
```

People only run `df -h` after that error and conclude the disk is lying. The block pool and the inode pool are separate. Mail spools and caches of tiny files fill inodes first. `-i` does not show which directory holds them. It shows the filesystem. `-h` and `-i` together are not a combined report on GNU df. `-i` replaces the block columns. Use two commands.

## Force one portable line per filesystem
Also asked as: df -P; POSIX df; df output wraps; parse df; 1024-blocks column; do not wrap the mount point
`-P` uses the POSIX layout. Sizes are 1024-byte blocks. The header says `1024-blocks`. Each filesystem is one line, so a long mount point does not wrap onto a second line the way default GNU df can. Scripts that cut fields want this form. It does not change which filesystems are listed.

```sh
df -P
```

Default GNU output is not a stable interface. A wrapped line becomes an extra record, and a human-readable suffix is not a number. `-P` is the form to parse if you must parse `df` at all. The mount point can still contain spaces. Field splitting on blanks still breaks. `--output` is the GNU way to choose columns. It is not POSIX, and macOS df does not have it.

## Print the filesystem type
Also asked as: df -T; filesystem type column; ext4 or xfs; df print type; what type is this mount
`-T` adds a type column, such as `ext4`, `xfs`, `tmpfs`, or `nfs`. The rest of the report is the usual block report. The type is the kernel's type string, not the label on the disk and not a promise about the on-disk format version. It is GNU and common. Do not confuse it with `-t`, which filters.

```sh
df -T
```

People pass `-t` to print the type and get either an error or a filtered list. `-T` prints. `-t` selects. A FUSE mount shows the FUSE type the kernel has, which may be `fuse` or a more specific name. Network filesystems show their client type. `df -T` on a path prints the type of the filesystem that holds that path, still not the type of the file.

## Limit the list to local filesystems
Also asked as: df -l; local filesystems only; skip nfs; df without network mounts; df --local
`-l` lists filesystems the kernel considers local. NFS and other remote mounts are omitted. `tmpfs` and the root disk remain. The flag does not mean "block devices only." A local memory filesystem is local. It is the right filter when a hung network mount would stall a plain `df`.

```sh
df -hl
```

A network mount that is stuck can make plain `df` wait. `-l` avoids asking it. It does not unmount anything, and it does not time out a mount that was explicitly named. `df -l /mnt/nfs` still asks about that path if you named it. People use `-l` to hide `tmpfs` and it does not. Excluding a type is `-x`.

## Skip a filesystem type
Also asked as: df -x; exclude tmpfs; df --exclude-type; hide overlay; skip a type; df without squashfs
`-x` drops filesystems of one type. The type string is the same one `-T` prints. Repeat the flag to drop more than one type. `-x tmpfs` removes memory filesystems of that type from the list. It does not remove a different type that happens to live in memory.

```sh
df -x tmpfs
```

The match is exact. `-x fuse` does not hide a type named `fuse.sshfs` if that is the name the kernel reports. Check `-T` first. `-x` does not exclude a mount point by path. There is no "skip this directory" operand. A path operand still selects the filesystem under that path even if you also excluded its type. The filter applies to the unsolicited list.

## Show only one filesystem type
Also asked as: df -t; only ext4; df --type; filter by filesystem type; list nfs mounts
`-t` keeps filesystems of one type and drops the rest. Repeat it to keep more than one type. The type string matches the column `-T` would print. This is a filter, not a request to print the type. Add `-T` if you want the type column as well.

```sh
df -t ext4
```

People pass a mount point to `-t` and get nothing, or an error, because `-t` wants a type name. `ext4` is a type. `/` is a path. A path goes after the options, not attached to `-t`. `-t` does not discover disks that are not mounted. An unmounted ext4 partition is invisible to `df`.

## Include pseudo and duplicate filesystems
Also asked as: df -a; show all filesystems; include proc and sysfs; dummy filesystems; inaccessible filesystems
`-a` includes filesystems GNU df hides by default. Those are pseudo-filesystems such as `proc`, `sysfs`, and `devpts`, plus duplicates and filesystems it could not stat. The list gets much longer and the sizes on pseudo-filesystems are not disk space. They are kernel tables.

```sh
df -a
```

People run `-a` looking for a missing disk and then try to read the `Size` of `proc`. That number is not capacity you can copy files into. `-a` is for "what is mounted that default `df` hid," not for a free-space check. Inaccessible filesystems appear as rows you still cannot use. The status can be non-zero if a filesystem could not be read.

## Print a grand total
Also asked as: df --total; total free space; sum of filesystems; df grand total; add up df
`--total` adds a final row that sums the listed filesystems. GNU df omits entries that would not add to available space, then prints the total. The flag is GNU. macOS and BusyBox do not have it. The total double-counts a filesystem that is mounted in more than one place if both mounts are in the list.

```sh
df -h --total
```

A bind mount of the same device is the same blocks printed twice. The total adds the rows it kept, not the unique devices. Filter first if you only want local disks or one type. `--total` does not mean "bytes I personally used." It is the sum of filesystem sizes and free space. `du` is still the tool for one tree.

## Choose the columns
Also asked as: df --output; df columns; machine readable df; df output fields; only the available column
`--output` picks columns by name. Useful names are `source`, `fstype`, `size`, `used`, `avail`, `pcent`, `target`, `file`, and the inode names `itotal`, `iused`, `iavail`, and `ipcent`. A comma-separated list with no spaces is the argument. This is GNU. The header is still printed.

```sh
df --output=target,avail -h
```

People parse default `df` with `awk` and the script breaks when a filesystem name contains a space or the line wraps. `--output` is the supported way to drop columns. It is still text. It is not NUL-separated, and `-h` still adds a suffix, so a consumer that wants a raw number should skip `-h` and use `-P` or an explicit block size. An unknown field name is an error and exit non-zero.

## Set the block size
Also asked as: df -B; df -k; block size; df in megabytes; DF_BLOCK_SIZE; 1K blocks
`-B` sets the unit, as in `-BM` for 1 048 576-byte blocks. `-k` is 1024-byte blocks, which is also the GNU default. The number in the column is a count of those blocks, not a count of bytes, unless the size you asked for is 1 byte. `DF_BLOCK_SIZE`, `BLOCK_SIZE`, and `BLOCKSIZE` in the environment change the default. `-h` overrides them with a human scale.

```sh
df -BM
```

`POSIXLY_CORRECT` switches the default to 512-byte blocks, so a script that did not pass `-k` or `-P` can see numbers twice as large on a POSIX-strict box. Pass the unit. `-B` does not change `Use%`. A size suffix of `MB` is decimal and `M` is binary, the same rule as the rest of coreutils. `-k` is the portable "I meant 1024" flag.

## Tell df from du
Also asked as: df versus du; disk free versus disk usage; why df and du disagree; folder size; who is using the space
`df` reads filesystem totals from the kernel. `du` walks a tree and adds up files it can see. They answer different questions. `df` is "how much room is left on this filesystem." `du -sh -- dir` is "how much does this tree add up to." Deleted files that a process still has open count in `df` and not in `du`. Permission-denied directories count in `df` and are missing from `du`.

```sh
df -h -- dir
```

```sh
du -sh -- dir
```

Use `df` for free space. Use `du` for one directory. A number from `du` larger than `df` used space is possible when `du` is adding a tree that crosses into another filesystem, unless you passed `du -x`. A number from `du` much smaller than `df` used space is the usual case. Other directories, reserved blocks, and open-but-deleted files sit in the gap. Neither command deletes anything.

## Read a repeated filesystem
Also asked as: df shows the same disk twice; bind mount; same size on two mount points; duplicate filesystem; df counts twice
Each mount is a row. A bind mount of `/` on `/srv/bind` prints the same size, used, and available as `/`, because it is the same filesystem. Filling one mount fills the other. `--total` will add those rows if both are listed. `df` is not deduplicating by device.

```sh
df -h
```

People add the `Size` column and buy a disk, or panic at a full filesystem that seems to exist twice. Check the source column. The same source on two mount points is one pool. Different sources with the same used percent are two pools. `df -a` shows still more duplicates and pseudo-filesystems. It does not merge them. A path operand prints the mount that contains that path, which may be the bind and not the original mount point.
