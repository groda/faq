# free

`free` prints how the kernel is using memory, from `/proc/meminfo`. It does not show which process is using RAM. That is `ps` or `top`. It does not free anything. The `free` column is memory with nothing in it. The `available` column is the estimate of what a new program can have without swapping. They are not the same. GNU `free` from procps on Linux is assumed here. macOS has no procps `free`. Its pager is `vm_stat`. BusyBox `free` is thinner and often has no `available` column. Exit status is 0 when `/proc/meminfo` was read. It is non-zero when it was not. A full-looking `used` column is not a failure.

Basic form: free

There is no file operand. A word after the command is a flag or an error. `-s` takes a delay in seconds. Human units are `-h`. Powers of 1000 are `--si`. The numbers are a snapshot. A second later they move.

## Show memory and swap
Also asked as: how much ram is free; free default; memory usage; show swap; free command
With no flags, `free` prints two rows. `Mem` is RAM. `Swap` is swap. The columns are `total`, `used`, `free`, `shared`, `buff/cache`, and `available`. Units are kibibytes unless you ask otherwise. `shared` is memory used by tmpfs and shared segments. It is not "RAM other people are using."

```sh
free
```

People subtract `used` from `total` and treat the rest as what they can start a job with. Linux uses spare RAM for cache. That cache is in `buff/cache`, and much of it is reclaimable. `available` is the column that already guesses how much can be given to a new program. `free` by itself is the empty part only. A small `free` and a large `available` is a healthy machine, not a full one.

## Print sizes people can read
Also asked as: free -h; human readable memory; free in gigabytes; free -h vs free; powers of 1024
`-h` prints powers of 1024, such as `Mi` and `Gi`. The numbers are rounded. A machine with 16 GiB of RAM can show a total that looks a little under 16. `-h` does not change the meaning of the columns. It only scales them. `--si` uses powers of 1000, so the totals look closer to a vendor's "GB" and do not match `-h`.

```sh
free -h
```

People compare `free -h` to a DIMM label in decimal GB and think the kernel lost memory. The label is decimal. `-h` is binary. Firmware and the kernel also reserve some RAM before `/proc/meminfo` exists. `free` cannot see that. Use `-b` if you want bytes and no rounding. Use `-h` to read the screen.

## Read the available column
Also asked as: free available; MemAvailable; how much can I allocate; available versus free; buff/cache reclaimable
`available` is the kernel's estimate of memory a new program can use without swapping. It starts from `free`, adds cache and buffers that can be dropped, and subtracts watermarks and unreclaimable bits. It is an estimate, not a promise. A program that allocates more than `available` may still start, and it may push the machine into swap.

```sh
free -h
```

People alarm on `used` near `total`. On a machine that has been up, `used` includes cache the kernel will drop. Compare `available` to the job you want to start. If `available` is small and `swap` used is climbing, the machine is actually short. `available` does not include swap. Swap is the second row. A zero `available` with idle CPU is a memory problem, not a load problem.

## See why used is so high
Also asked as: buff/cache; linux ate my ram; used plus free is not total; cache is not leaked; buffers and cache
`used` on the `Mem` row is total minus free minus buffers minus cache, in the default layout. Cache is page cache. It makes disk reads cheap. It is not a leaked process. Dropping it by hand is almost never the fix. The kernel drops it when a program needs the pages. `buff/cache` is that reclaimable-looking pool, though not every page in it can be dropped.

```sh
free -h
```

People run `echo 3 > /proc/sys/vm/drop_caches` to "free RAM" and the number goes up, then back down as soon as files are read. That did not fix a leak. It threw away a cache. A real leak is a process whose resident set grows and never returns. `free` cannot name it. `ps` or `top` can. `used` plus `free` plus `buff/cache` is the reconciliation. `available` is not part of that sum. Do not add it in.

## Show buffers and cache separately
Also asked as: free -w; wide output; buffers versus cache; split buff/cache; free wide
`-w` prints `buffers` and `cache` as two columns instead of `buff/cache`. Buffers are a small block-device cache on modern kernels. Cache is the page cache and is the large number. The `available` column is still there. `-w` is GNU procps. BusyBox `free` often cannot split them.

```sh
free -wh
```

People see a large cache column and start looking for a process named cache. There is not one. It is files the kernel has read, plus other cached pages. A tmpfs lives in memory and is counted in `shared`, and its pages are not a free lunch. Deleting the files in the tmpfs returns the RAM. `free` does not show which mount. `df -h` does.

## Show swap
Also asked as: free swap; swap used; is swap full; swap total; out of memory swap
The `Swap` row is the swap devices and swap files the kernel has on. `used` is pages written out. `free` is unused swap. There is no `available` on that row. Swap used without a small `available` on the `Mem` row can be old pages that have not been pulled back. Swap used with a small `available` and a climbing number is pressure.

```sh
free -h
```

People see any swap use and reboot. Idle swap is not a failure. The kernel swapped a cold page and may never need it. People see zero swap and think they cannot run out of memory. They can. A machine with no swap gets the OOM killer instead of a slow disk. `free` does not show swap I/O. A second sample from `free -s` shows whether the used number is moving. `swapon --show` lists the devices. `free` only totals them.

## Sample over time
Also asked as: free -s; watch free; repeat free; free every second; free -c count
`-s` repeats forever, sleeping the number of seconds you give between samples. `-c` stops after that many samples. The header is printed once. Each sample is a new read of `/proc/meminfo`. This is the way to see a leak or a cache grow. One row is a snapshot.

```sh
free -h -s 2 -c 5
```

People run `free` once during a job and miss the dip. Two samples a few seconds apart tell you if `available` is falling. `-s` without `-c` runs until you interrupt it. In a script, pass `-c`. The delay is not exact under load. `free` sleeps, then reads. It does not average the interval. A `watch` around `free` is the same idea and redraws the screen. `-s` appends lines.

## Print bytes or megabytes
Also asked as: free -b; free -m; free -g; free -k; bytes exactly; megabytes
`-b` prints bytes. `-k` prints kibibytes, the default. `-m` prints mebibytes. `-g` prints gibibytes. These are powers of 1024, and they are rounded down to an integer, so `-g` can show 0 for a machine with under 1 GiB free. `-h` is the rounded human form. `--si` switches the human form to powers of 1000.

```sh
free -b
```

People script `free -m` and compare it to a limit in decimal megabytes. The column is mebibytes. A limit of 1000 MB is not 1000 in the `-m` column. Use `-b` and compare bytes if the limit is exact. `-g` is a bad input to a script because of the rounding. `available` in bytes is the number to compare with a job that knows its own size. `free` does not accept a unit on `-s`. The delay is seconds.

## Add a total line
Also asked as: free -t; total ram plus swap; free total row; mem plus swap; combined memory
`-t` adds a `Total` row that adds `Mem` and `Swap`. The `available` cell on that row is not a meaningful combined pool. RAM and swap are not the same speed, and `available` was never a swap estimate. The total `used` adds RAM used and swap used. It is easy to overread.

```sh
free -ht
```

People quote the total `free` as what a program can allocate. A program does not allocate swap as if it were RAM. It faults, and the kernel may swap. The `Total` row hides that. Use the `Mem` `available` column for "can I start this," and the `Swap` row for "is the machine already paging." `-t` is GNU. It does not change the snapshot. It only adds the sum.

## Tell free from vmstat and top
Also asked as: free versus top; free versus vmstat; who is using memory; process rss; free does not name a process
`free` is system-wide counters. It does not have a process column. `top` and `ps` show resident set and virtual size per process. Those per-process numbers overlap. Shared libraries are in more than one resident set, so the sum of RSS is larger than `used`. `vmstat` shows the rate of swap in and swap out. `free` shows the level, not the rate.

```sh
free -h
```

People add RSS and expect it to equal `used`. It will not. Shared pages are counted once in `free` and once per process in `ps`. A large virtual size is not RAM. The resident set is closer, and it is still not `available`. Use `free` for the machine. Use `ps -o pid,rss,comm` when you need a name. Neither command frees memory. Killing a process does, if that process held the pages.

## Read a small free column without panicking
Also asked as: free column near zero; is the ram full; cache filled the ram; linux memory full; free is low available is high
A `free` column near zero with a large `buff/cache` and a comfortable `available` means the kernel used idle RAM as cache. That is the normal steady state. The number to watch is `available`, then swap used over time. A small `free`, a small `available`, and a rising swap used means the cache cannot save you.

```sh
free -h
```

People buy RAM because `free` showed 200 MiB free on a 64 GiB box that had 40 GiB available. The purchase does not fix the reading. People also ignore a small `available` because `free` still shows a few hundred megabytes. That remainder is the watermark. New allocations will come from cache or from swap. Read both columns. One sample cannot show a leak. `-s` can.

## What a failed free means
Also asked as: free exit status; free exit code; cannot read meminfo; free failed; no proc
`free` exits 0 when it printed the counters. It exits non-zero when `/proc/meminfo` cannot be read. A container that hides proc, or a chroot with no proc mounted, is the usual failure. The message is on stderr. There is no row to parse in that case. A zero in a column is still a successful read.

```sh
free -h
```

People treat a scary `used` as a failed command. The command succeeded. The machine is busy. Parsing `free` in a script is brittle because the header and the units change with `-h`. Use `-b` and read the `available` field by column if you must script it. procps has changed columns before. A parser that assumes `free` is the third number and `available` does not exist is the old layout. Current GNU `free` has `available`. Trust the header, or read `/proc/meminfo` and the `MemAvailable` key.
