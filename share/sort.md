# sort

`sort` reads lines and writes them in order on standard output. It does not edit the file unless you pass `-o`, it does not search inside fields for a pattern, and it does not assume the input is already ordered. With no file, or with `-`, it reads standard input and waits for it. Several files are concatenated and sorted as one stream, not sorted one file at a time. The comparison is the whole line unless you set a key. Locale rules decide alphabetic order, so the same bytes can sort differently on another machine.

Basic form: sort file

Operands are paths. A name that starts with `-` is a flag unless it comes after `--`. Quote nothing for `sort` itself. The shell still expands globs before `sort` sees them. GNU sort from coreutils on Linux is assumed here. macOS ships BSD sort. It has `-n`, `-r`, `-u`, `-k`, `-t`, `-o`, `-c`, `-f`, and `-s`, and it does not have GNU `-h`, `-V`, `-g`, `-z`, or `--debug`. BusyBox sort is smaller again. Numeric, reverse, unique, and a simple key are usually there. Human sizes, version order, and NUL records often are not. Exit status is 0 when the sort finished, including when the input was empty. It is 1 when `-c` or `-C` finds a line out of order. It is 2 when a file cannot be read or the flags are illegal. Writing the sorted lines is not a statement that they were unique.

## Sort the lines of a file
Also asked as: how to sort a file; sort alphabetically; sort text lines; sort whole line; sort does not edit the file
With no key, `sort` orders complete lines and prints the result on standard output. The input file is left as it was. Order is the locale's collating order, not a promise about ASCII. On a terminal you see the sorted lines and nothing else. Empty input prints nothing and exits 0.

```sh
sort -- file
```

People expect the file to change on disk. It does not. They also expect `A` and `a` to group together, or `_` to sort in byte order. In a typical UTF-8 locale, case and punctuation follow the locale, and the help text on GNU sort warns about that. `LC_ALL=C sort -- file` is the byte-value order, and it is also the fast one. A header line is just another line. `sort` will move it unless you peel it off first.

## Sort numbers
Also asked as: sort -n; numeric sort; sort numbers not alphabetically; 10 before 2; sort -nr
`-n` compares a leading number instead of the character codes. `2` then comes before `10`. Blanks before the number are ignored. The number is the integer or decimal at the start of the key. Text after the number is not part of the numeric value. Without another key, the key is the whole line, so the number has to begin the line.

```sh
sort -n -- file
```

```sh
sort -nr -- file
```

Use `-n` for low to high. Use `-nr` for high to low. `-r` alone reverses the alphabetic order. It does not turn on numeric comparison. A line like `item 10` sorts as the number 0, because `-n` stops at the first character that is not part of a number, and the last-resort compare then sorts the text. Put the number in a field and select it, or the order will look alphabetic again. Units such as `K` and `M` are not numbers to `-n`. That is `-h`.

## Sort sizes such as 2K and 1G
Also asked as: sort -h; human numeric sort; sort 1G 2K 10M; sort file sizes; GNU sort -h
`-h` compares human sizes. A leading number may be followed by a suffix such as `K`, `M`, or `G`. `2K` comes before `10M`, which comes before `1G`. The suffix is 1024-based, the same scale GNU `ls -h` prints. It is GNU. macOS and BusyBox usually do not have it.

```sh
sort -h -- file
```

People pass `-n` to a listing of `du -h` or `ls -h` and still get `1G` before `2K`, because `1` is less than `2` as a plain number and the letter is ignored. `-h` is the flag that understands the suffix. It does not understand every unit. A bare word with no leading number compares as zero. Decimal points follow the locale, so a comma is not a decimal mark in the C locale.

## Sort version numbers inside the text
Also asked as: sort -V; version sort; natural sort; a1 a2 a10; sort filenames with numbers; GNU version-sort
`-V` compares runs of digits by their numeric value and compares the other characters as text. `a1`, `a2`, and `a10` come out in that order. Alphabetic sort puts `a10` before `a2`. This is GNU. It is the same idea as `ls -v`, applied to whole lines or to a key.

```sh
sort -V -- file
```

`-n` is the wrong tool once the number is not a single value at the start of the key. `-n` on `a10` and `a2` sees no leading number and falls back to text. `-V` is also not a promise about every versioning scheme. Leading zeros and mixed suffixes still need a look. It does not ignore case unless you also pass `-f`.

## Ignore letter case
Also asked as: sort -f; case insensitive sort; sort ignore case; fold case; A and a together
`-f` folds lowercase to uppercase before comparing, so `a` and `A` compare equal and group together. The printed line keeps the case it had in the input. `-f` does not translate the file. It only changes the comparison.

```sh
sort -f -- file
```

Equal keys are not a stable group. Without `-s`, lines that compare equal are still ordered by the whole line as a last resort, so `A` can land before or after `a` depending on that tie-break. People expect `-f` to mean "dictionary order" or "ignore punctuation." Punctuation is still significant. That is `-d`. Locale rules still apply to letters that are not simple ASCII case pairs.

## Sort on one column
Also asked as: sort -k; sort by column; sort second field; sort -k2; key starts at 1; sort by field
`-k` selects the sort key. Fields are numbered from 1, not from 0. A blank run is the default separator, and `-k2` means "from field 2 through the end of the line," not "field 2 only." Global `-n` applies to that key. The rest of the line is not ignored. It is part of the key when you leave the end open.

```sh
sort -k2,2n -- file
```

The usual mistake is `sort -k2 -n` and a belief that only column 2 matters. `-k2` runs to the end, so later columns change the order. `-k2,2` stops at the end of field 2. The `n` attached to the key, as in `-k2,2n`, is the numeric comparison for that key only. A key option replaces the global ordering flags for that key. `--debug` on GNU sort underlines the bytes it actually compared. Use that when the column looks right and the order is wrong.

## Change the field separator
Also asked as: sort -t; sort comma separated; sort colon separated; field separator; sort csv column; sort -t,
`-t` sets the field separator to a single character. `-t,` splits on commas. `-t:` splits on colons. Fields are then the text between separators, and an empty field is a real field. `-k` still counts from 1. Quote or escape the separator when the shell would eat it. A tab separator is `-t $'\t'`.

```sh
sort -t, -k2,2n -- file
```

The default separator is not "every space." It is the transition from non-blank to blank, and the blanks at the start of a field stay in the field unless you pass `-b`. Consecutive spaces are one separator by default. With `-t`, each comma is its own separator, so `a,,b` has an empty second field. `-t` does not understand quotes. A comma inside a quoted CSV field still splits. `sort` is not a CSV parser.

## Drop duplicate lines
Also asked as: sort -u; unique lines; sort and uniq; remove duplicate lines; sort unique; distinct lines
`-u` prints one line from each run of equal keys. With no `-k`, the key is the whole line, so you get the distinct lines, sorted. This is the usual replacement for `sort | uniq`. `uniq` alone only collapses duplicates that are already adjacent. `sort -u` does both steps.

```sh
sort -u -- file
```

With a key, equality is the key, not the whole line. `-u` turns off the last-resort full-line compare, so two lines that share a key collapse even when the rest differs. The line you keep is the first of that equal run, which is not a promise about input order unless you also pass `-s`. People use `-u` to mean "unique by column 2, and keep the rest." They get one survivor per column 2 and lose the other rows. That is what the flag does.

## Keep the original order of ties
Also asked as: sort -s; stable sort; preserve input order; equal keys stay in order; sort stable
`-s` disables the last-resort comparison. Lines whose keys compare equal stay in the order they were read. Without `-s`, equal keys are still ordered by the whole line, so a later column can reshuffle rows that agreed on the key you named.

```sh
sort -s -k2,2n -- file
```

People sort on a column, see rows with the same column swap, and think the sort is broken. The tie-break is the rest of the line, and it is on by default. `-s` is how you turn that off. `-u` already disables the last-resort compare. Adding `-s` to `-u` is what makes the survivor the first input line of that key, rather than whichever line the merge left at the front of the run.

## Write the sorted lines back to the file
Also asked as: sort -o; sort in place; sort output file; replace the file with sorted lines; sort -o file file
`-o` names the output file. `sort` writes a temporary result and replaces that file when the sort finishes, so `-o file file` is the safe way to sort a file onto itself. Redirecting with `sort file > file` is not safe. The shell truncates `file` before `sort` reads it.

```sh
sort -o file -- file
```

People use `-o` as if it selected a column. It does not. The column flag is `-k`. `-o` is only the destination. A missing input is still exit 2, and `-o` does not create a sorted file from a path that could not be read. Several inputs can be written to one `-o`. They are merged into one sorted stream, not written back into each input.

## Check that the input is already sorted
Also asked as: sort -c; sort -C; check sorted; is this file sorted; disorder; sort exit code 1
`-c` reads the file and does not sort it. If a line is out of order under the same flags you would have used to sort, it prints the first bad line on stderr and exits 1. If the file is in order, it prints nothing and exits 0. `-C` is the quiet form. It uses the same status and prints no "disorder" line.

```sh
sort -c -n -- file
```

```sh
sort -C -n -- file
```

Use `-c` when you want the first offending line. Use `-C` when a script only wants the status. The check uses the same comparison as a real sort. `sort -c` on a numeric file accepts an alphabetic order that `-n` would reject, and `sort -c -n` does the opposite. `-c` is not a checksum of the file. A missing file is exit 2, not exit 1. Exit 1 means "readable, and not in this order."

## Merge files that are already sorted
Also asked as: sort -m; merge sorted files; sort merge; combine sorted inputs; do not re-sort
`-m` merges files that are already sorted into one sorted stream. It does not sort an unordered file. Each input is assumed to be ordered under the same flags you pass to the merge, such as `-n`. The result is written to standard output, or to `-o`.

```sh
sort -m -n -- file file
```

People use `-m` as a faster `sort` on arbitrary input. An unsorted input is not corrected. The merge walks the files as if each one were ordered, and the output can be wrong with exit 0. Sort each file first, or drop `-m` and let `sort` do a full sort. `-m` is the right tool when the pieces are known to be sorted and you only want them combined.

## Sort month names
Also asked as: sort -M; month sort; sort Jan Feb Mar; sort by month name; abbreviated month
`-M` orders month abbreviations. Unknown text comes first, then `JAN` through `DEC`. The match is case-insensitive, so `jan` and `MAR` sort as January and March. It looks at the month token in the key, not at a date in the middle of a sentence, unless that token is the key.

```sh
sort -M -- file
```

`sort` does not parse `2020-03-01` as a month because of `-M`. That string is not a month name. ISO dates sort correctly as plain text, or as a year-month-day key if you split them. `-M` is the wrong flag for those. A full name such as `January` is not the abbreviation `-M` expects. GNU sort wants the three-letter form.

## Force byte order instead of locale order
Also asked as: LC_ALL=C sort; sort byte order; traditional sort order; locale changes sort; ASCII sort
GNU sort says the locale controls the order. `LC_ALL=C` selects byte values, so uppercase letters come before lowercase letters and the result does not depend on the language settings. It is also the order to ask for when you want the same output on every machine. The variable has to be set on the `sort` command, not only in the script's later environment.

```sh
LC_ALL=C sort -- file
```

People set `LANG` and still get a surprise because `LC_COLLATE` is set on its own. `LC_ALL` overrides the category `sort` actually uses. Byte order is not numeric order. `10` still comes before `2` until you add `-n`. In the C locale, the decimal point for `-n` is `.`. A comma is not part of the number, so `1,2` compares as `1`.

## Read lines from standard input
Also asked as: sort stdin; sort a pipe; sort with no file; sort -; sort command output
With no file, `sort` reads standard input until end of file. `-` means the same thing when you also pass filenames. A pipe closes that input. Typed input does not end until you send end of file. The sorted lines are written to standard output, which is a different stream from the input.

```sh
sort -n -- -
```

People run `sort` with no file in a script and the command sits there. It is waiting for stdin, not sorting the current directory. `sort` never lists a directory. If you pass a directory path, it fails. Combining files and stdin is `sort -- file -`. The `-` has to be after `--` only when you need to stop a leading dash from looking like a flag. Here `-` is the stdin operand and may sit with the other paths.

## Sort several files as one list
Also asked as: sort multiple files; sort file1 file2; concatenate and sort; sort more than one file; one sorted stream
Every file operand is read and the lines are sorted together. The output is one sequence, not a sorted block per file. There is no header between files. A failure to open any named file is exit 2.

```sh
sort -n -- file file
```

People want each file sorted into itself. One `sort` invocation does not do that. `-o` names a single destination. To sort files separately, run `sort` once per file. Order of the operands does not matter for a full sort, because all lines are compared. It does matter for `-m`, which trusts each file's existing order.

## Separate records with a NUL
Also asked as: sort -z; sort --zero-terminated; sort null delimited; sort find -print0; newline in the record; GNU sort -z
`-z` uses a NUL byte as the record separator instead of a newline. A newline inside a record is then ordinary data. The output records are also NUL-terminated. GNU sort has this. macOS and BusyBox often do not. Pair it with a producer that writes NULs, such as `find -print0`.

```sh
sort -z -- file
```

Without `-z`, a newline ends the record, so a filename that contains a newline becomes two records and the sort is wrong while looking successful. `-z` does not change the comparison. You still want `LC_ALL=C` for raw bytes, and `-n` or `-V` only if the records are numbers or versions. Do not mix `-z` output with a tool that reads lines. The NUL is invisible in a terminal and the records look glued together.

## Ignore punctuation while comparing
Also asked as: sort -d; dictionary order; ignore punctuation; sort alphanumerics only; blanks and letters only
`-d` compares only blanks and alphanumeric characters. Other characters are ignored for the comparison. The printed line is unchanged. Hyphens and similar marks no longer decide the order by themselves. Locale still decides which characters count as alphanumeric.

```sh
sort -d -- file
```

Lines that differ only by punctuation compare equal, and the last-resort compare can still separate them unless you pass `-s`. People use `-d` to mean "human dictionary, case insensitive." Case is still significant unless you add `-f`. `-d` is not field splitting and it is not `-t`. It does not drop the characters from the output.

## Sort scientific notation
Also asked as: sort -g; general numeric sort; sort 1e2; sort floats with exponent; sort -g versus -n
`-g` compares numbers the way a general numeric conversion does, including an exponent. `3`, `20`, and `1e2` sort by value, so `1e2` comes after `20`. It is GNU. `-n` does not treat `e` as an exponent. It stops at the first character that is not part of a plain number.

```sh
sort -g -- file
```

`-g` is slower and looser than `-n`. Text that is not a number becomes a zero or a conversion failure and then ties. Do not use it as the default numeric flag. Use `-n` for ordinary integers and decimals, `-h` for `K`/`M`/`G`, and `-g` when the file really contains exponents. The decimal point still follows the locale.

## Shuffle lines
Also asked as: sort -R; random sort; shuffle lines; sort random order; sort versus shuf
`-R` assigns each distinct key a random weight and sorts by that weight. Identical keys stay together. The output is a shuffle of the keys, not a cryptographically serious shuffle, and it is not stable. GNU `shuf` is the tool that shuffles lines without the "group equal keys" rule.

```sh
sort -R -- file
```

People run `sort -R` twice and expect a different grouping of duplicate lines. Equal lines share a key, so they move as a group. `shuf -- file` is the command that permutes lines independently. `-R` also makes `-u` less meaningful, because uniqueness still collapses equal keys and the random order is among the survivors. Do not use `-R` when you need a reproducible order unless you control `--random-source`.

## Ignore leading blanks in the key
Also asked as: sort -b; ignore leading blanks; leading spaces change the field; sort -b -k; blanks in the key
`-b` ignores leading blanks in the key. With the default separator, the blanks that begin a field are part of the field, so a character offset such as `-k2.3` can land on a space on one line and a digit on another. `-b` skips those spaces before the key is taken. Attached to a key, `-k2b` applies only there.

```sh
sort -b -k2,2n -- file
```

`-n` already skips blanks when it parses a number, so `-b` is not what fixes a numeric column that looks wrong. The usual failure is a character position inside the field, or a belief that every space is a separator. `-t` is the fix when the separator is a specific character. `-b` is the fix when the default blank separator left padding inside the key and you are counting characters.

## See why a key did not match the bytes you meant
Also asked as: sort --debug; which bytes does sort compare; debug sort key; underline the key; sort field looks wrong
`--debug` is GNU. It writes the sorted lines to standard output and, on stderr, underlines the part of each line that was the key. It also warns about a global flag that a key option overrode, and about locale numeric rules. It does not change the order.

```sh
sort --debug -k2,2n -- file
```

Use it when `-k` is silently using the wrong span. The underline is the answer. A key of `-k2` underlines from field 2 to the end of the line. A key of `-k2,2` underlines field 2 only. `--debug` output is not for a pipeline. The notes go to stderr, but the extra annotation is easy to confuse with the data if you merge the streams. Drop it once the key is right.
