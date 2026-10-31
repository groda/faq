# head

`head` prints the start of a file. The default is the first 10 lines. It does not search, and it does not follow a file. That is `tail`. With no file it reads standard input and waits. GNU head from coreutils on Linux is assumed here. macOS `head` has `-n` and `-c`, and it does not accept a negative count. BusyBox `head` usually has `-n` and `-c`. Exit status is 0 when the lines were printed. It is non-zero when a named file cannot be read. A short file is success and prints the whole file.

Basic form: head -- file

Operands are files. A name that starts with `-` is a flag unless it comes after `--`. `-n` takes a count. A leading `-` on that count, as in `-n -2`, means "all but the last 2" and is GNU. Quote a name with spaces. Several files get a header each, unless you pass `-q`. A directory operand is an error.

## Print the first lines
Also asked as: head a file; first lines; head -n; first 10 lines; top of a file
With no flags, `head` prints the first 10 lines. `-n 20` prints the first 20. A file shorter than the count is printed whole. The lines are in order. `head` does not add a header for a single file. It does not modify the file. It stops reading once it has enough, which matters on a slow pipe.

```sh
head -n 20 -- file
```

People write `head -20` and it works on GNU and on many other builds, as a legacy form. Prefer `-n 20`. People also expect `head` to wait for the rest of a growing file. It prints and exits. A file with no trailing newline still prints its last partial line if that line is within the count. A binary file is printed as bytes and will look wrong in a terminal. `head` is a text tool unless you meant `-c`.

## Print all but the last lines
Also asked as: head -n -1; drop the last line; all but the last; GNU head negative; head minus
`-n -1` prints every line except the last. `-n -5` drops the last five. The minus is GNU. macOS `head` rejects it. A file shorter than the drop count prints nothing and exits 0. This is how a script strips a trailer without `sed`. It has to read the whole file, because it cannot know the last lines until the end.

```sh
head -n -1 -- file
```

People write `head -n -1` on a Mac and get a usage error. The negative form is GNU. `sed '$d'` is the portable "drop the last line." People also expect this to be fast on a huge file. `head` of the first N lines can stop early. `head` that must drop the last lines cannot. It reads to the end. A pipe works. The producer runs to completion.

## Print the first bytes
Also asked as: head -c; first bytes; head --bytes; first N characters; start of a binary
`-c 100` prints the first 100 bytes. `-c -100` prints all but the last 100 bytes, and is GNU. Bytes, not characters. A multibyte UTF-8 character can be split. This is the form for a header, a magic number, or a fixed-width record. A multiplier such as `K` means 1024 on GNU head. `kB` means 1000.

```sh
head -c 100 -- file
```

People use `-c` expecting characters and a multibyte file looks truncated mid-character. Count bytes, or use `-n` for lines. People also combine `-c` and `-n`. GNU head rejects the combination. A short file prints the whole file and exits 0. `-c` does not follow. There is no follow flag on `head`. Use `tail -f` if the file is growing and you want new bytes.

## Read from a pipe
Also asked as: command | head; head stdin; first lines of a pipeline; head a stream; no file
With no file, or with `-`, `head` reads standard input. It prints the first N lines and exits. The producer then gets a broken pipe if it is still writing, and may exit with a signal status. That is normal. `head` does not wait for the producer to finish once it has its lines. This is the cheap preview.

```sh
command -- file | head -n 20
```

People pipe a command into `head` and the producer reports a broken pipe, and they treat it as a failure of the producer. `head` closed the pipe early on purpose. Under `set -o pipefail`, that signal status can fail the pipeline. People also `head` a command that prompts. The prompt may be among the first lines, and the rest never runs. A producer that does not handle `SIGPIPE` can look noisy. The preview is still the first lines.

## Print several files
Also asked as: head file file; multiple files; head headers; head -q; quiet head
`head file file` prints the first lines of each, with a header naming the file. `-q` suppresses the headers. `-v` forces a header even for one file. The count applies to each file, not to the combined stream. A missing file is an error for that name. The others are still printed. The status is non-zero if any could not be read.

```sh
head -n 5 -q -- file file
```

People expect one merged head of both files. `head` does not merge. It walks the operands. People also want the headers in a script and they clutter the parse. `-q` is the off switch. A glob that matches nothing is left literal by bash, and `head` looks for a file named with a star. `nullglob` changes that. There is no follow for several files. `head` is a one-shot print.

## Skip a byte-order mark or a fixed header
Also asked as: head skip bytes; drop a BOM; head -c and tail; fixed header; cut the front
`head` keeps the front. To drop a fixed front and keep the rest, use `tail -c +N` or `tail -n +N`. To keep a fixed front and drop the rest, use `head`. A three-byte UTF-8 BOM is `tail -c +4` to remove it, not `head`. `head -c 3` would print only the BOM. Know which end you want.

```sh
head -c 4 -- file
```

People `head -c 3` a file to "strip the BOM" and instead print the BOM and discard the file. Stripping the front is `tail` with a plus count. People also chain `head` into `tail` to window a file, `head -n 100 | tail -n 10`, and that works and reads only the first 100 lines. It is the right shape for a window near the start. A window near the end is `tail` alone.

## Use it to peek at a CSV
Also asked as: head a csv; first rows; preview a table; head -n 5 csv; see the header
`-n 5` shows the header and four rows. It does not parse commas. A header with an embedded newline is more than one line to `head`. A quoted field that contains a newline breaks the count. For a real CSV, a parser is the tool. For a peek at a clean file, `head` is enough.

```sh
head -n 5 -- file
```

People judge a file's columns from `head` and the later rows differ. `head` only saw the start. People also `head` a file with Windows line endings and the `^M` shows. `head` does not convert them. A huge first line, one record with no newline, makes `head -n 1` print the whole record. A line is a newline, not a screen row. `head` will print a gigabyte if the first newline is that far.

## NUL-separated records
Also asked as: head -z; zero terminated; head --zero-terminated; filenames with newlines; NUL records
`-z` uses a NUL byte as the record separator instead of a newline. A newline inside a record is ordinary text. This is GNU. It pairs with `find -print0` and `sort -z`. Without `-z`, a newline ends the record. macOS head has no `-z`. The count is then a count of NUL-separated records.

```sh
find . -type f -print0 | head -z -n 5
```

People pass `-z` on `head` and not on the producer. The counts disagree and the output contains raw NULs. Both sides need the same separator. `-z` does not imply `-q`. Headers are still a thing for named files. A missing final NUL is still a record, the way a missing final newline is still a line. Do not mix the output with a line-oriented tool without converting.

## Stop a producer early
Also asked as: head closes the pipe; SIGPIPE; head and pipefail; preview without the rest; early exit
`head` reads N lines and closes its input. A producer still writing receives `SIGPIPE` and typically exits. That saves the work of producing lines nobody will print. In a script with `pipefail`, the producer's signal exit can fail the pipeline even though `head` succeeded. Turn `pipefail` off for that pipeline, or accept the status, if a preview is allowed to stop the producer.

```sh
command -- file | head -n 20
```

People set `pipefail` globally and a preview pipeline starts failing. The producer was killed on purpose. People also want `head` to pause the producer and resume later. It cannot. The unread output is gone. If you need the rest, do not use `head` in the middle. Write the output to a file and `head` that. A producer that ignores `SIGPIPE` keeps running and gets an error on the next write. The preview is still correct.

## What a failed head means
Also asked as: head exit status; head failed; cannot open; no such file; head error
A missing file exits non-zero. A directory exits non-zero. A permission error exits non-zero. A short file exits 0 and prints what it has. An empty file exits 0 and prints nothing. A negative count on macOS is a usage error. On GNU it is the "all but the last" form and exits 0 when the file could be read.

```sh
head -- file
```

People treat empty output as failure. An empty file, or a count of 0, prints nothing and exits 0. People also `head` a binary and read the garbage as an error. The status was 0. The content is bytes. Under `set -e`, a missing file aborts the script at `head`, which is often what you want before a later step parses it. A broken pipe on the producer is not a `head` failure. `head`'s own status is 0 if it printed what it was asked for.
