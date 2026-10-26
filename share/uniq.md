# uniq

`uniq` collapses adjacent identical lines. It does not find duplicates that are not next to each other. Sort first, or the second copy of a line survives. It does not sort. With no file it reads standard input and waits. GNU uniq from coreutils on Linux is assumed here. macOS `uniq` is BSD. It has `-c`, `-d`, `-u`, and `-i`, and it has no `-z` or `--group`. BusyBox `uniq` usually has `-c`, `-d`, and `-u`. Exit status is 0 when the filter ran. It is non-zero when a named file cannot be read or an option is illegal. A file with no adjacent duplicates is success and prints every line.

Basic form: uniq -- file

Operands are an optional input and an optional output. One name is the input. Two names are input and output. `--` before a path stops a leading dash from looking like a flag. Quote a name with spaces. The comparison is the whole line, after any fields or characters you skip. A line is text ending in a newline. The last line still counts if the file has no trailing newline.

## Collapse adjacent duplicates
Also asked as: uniq a file; remove duplicate lines; uniq adjacent; collapse repeats; unique lines
With no flags, `uniq` prints a line once for each run of identical neighbors. A line that appears again later is printed again. The first of each run is the one that survives. Order is otherwise unchanged. This is not a set. It is a run-length squeeze.

```sh
uniq -- file
```

People run `uniq` on an unsorted file and the duplicates remain. The help text says this, and it is still the common surprise. `sort file | uniq` is the form that means "each distinct line once." `sort -u` does that in one process. People also expect `uniq` to ignore case. It does not, unless you pass `-i`. `Foo` and `foo` are different lines. Trailing spaces are part of the line. They are not trimmed.

## Count how many times each run appears
Also asked as: uniq -c; count duplicates; uniq --count; frequency of lines; how many repeats
`-c` prefixes each output line with the number of adjacent copies it collapsed. The number is padded. The count is of the run, not of the line in the whole file. An unsorted file still splits one value into several counts. Sort first if you want a real frequency.

```sh
sort -- file | uniq -c
```

People `uniq -c` and then `sort -n` to rank frequencies. That works after the input was sorted, so each value is one run. People also read the count as a total across the file when the input was not sorted. Two runs of the same line become two rows. `sort | uniq -c | sort -n` is the frequency table. The padding means a string sort of the count is wrong. Sort numerically.

## Print only the lines that were duplicated
Also asked as: uniq -d; only duplicates; uniq --repeated; lines that appear more than once; show repeats
`-d` prints one copy of each run that occurred more than once. Unique lines are omitted. `-D` prints every line of those runs, not one sample. `--all-repeated` is the long form of `-D`. A line that appears twice, far apart, is not a run. Sort first or `-d` misses it.

```sh
sort -- file | uniq -d
```

People want "every line that is a duplicate somewhere" and pass `-d` to an unsorted file. Only neighbors count. People also confuse `-d` with `-D`. `-d` is one representative. `-D` is every member of the run, which is how you see the original repeated rows. `-u` is the opposite filter. `-d` and `-u` together are not a useful combination. A run of length one is not a duplicate.

## Print only the lines that appeared once
Also asked as: uniq -u; only unique; uniq --unique; lines that appear once; suppress duplicates
`-u` prints runs of length one and drops runs that were longer. After a sort, that is the lines that occurred exactly once in the file. Without a sort, it is the lines that were not equal to their neighbor. A line that appears twice, not adjacent, is printed twice under `-u` if you did not sort.

```sh
sort -- file | uniq -u
```

People use `-u` expecting the same result as plain `uniq`. Plain `uniq` keeps one of each run, including runs that were long. `-u` keeps only the singleton runs. People also want "unique" to mean case-insensitive. Add `-i`. The sort has to use the same idea, `sort -f`, or the case folding disagrees and the runs do not form. `-u` does not sort. It only filters.

## Ignore case
Also asked as: uniq -i; ignore case; case insensitive uniq; uniq --ignore-case; Foo and foo
`-i` compares lines without regard to case. `Foo` and `foo` are one run if they are adjacent. The line that is printed is the first of the run, with its original case. `-i` does not rewrite the line to lowercase. Sort with `-f` if you need those lines adjacent first. GNU and BSD both have `-i`.

```sh
sort -f -- file | uniq -i
```

People `uniq -i` without sorting and the mixed-case copies are still apart. The fold only affects the comparison. People also expect the output to be lowercased. It is not. Use the first line's text, or pipe through `tr` if you wanted a normalized form. Locale can change what "case" means. A UTF-8 file is fine. A binary file is not a line-oriented input.

## Skip the first fields
Also asked as: uniq -f; skip fields; ignore the first column; uniq --skip-fields; compare the rest of the line
`-f N` skips N fields before comparing. A field is a run of blanks, then non-blank characters. The skipped part still prints. Only the comparison ignores it. This is how you collapse lines that differ only in a leading id or a count. The fields are whitespace-separated. A comma is not a separator.

```sh
sort -k2 -- file | uniq -f 1
```

People skip a field and forget to sort on the same key. The runs never form, and nothing collapses. Sort on the part you compare. People also expect `-f` to drop the field from the output. It does not. The whole line is printed. A field count past the end of a short line compares the empty remainder, so short lines can collapse together. Check the data. `-f` is not `-s`. Fields first, then characters, if you pass both.

## Skip the first characters
Also asked as: uniq -s; skip chars; ignore a prefix; uniq --skip-chars; compare after column N
`-s N` skips N characters before comparing. Tabs and spaces count. The skipped characters still print. Use this for a fixed-width prefix. Use `-f` for whitespace-separated fields. `-w N` is the other end. It compares at most N characters after the skip. A long line that differs only past that window collapses.

```sh
uniq -s 10 -- file
```

People count columns by eye and the tabs are one character each, not the visual stop. The skip is bytes of the line as stored, in the C locale. A multibyte character can split under a non-C locale if you count wrong. Prefer `-f` when the layout is fields. `-s` does not remove the prefix. A later cut does that. Adjacent-only still applies. Sort the same way you skip, or the neighbors are not the lines you think.

## Separate groups
Also asked as: uniq --group; group identical lines; blank line between groups; show groups; GNU uniq group
`--group` prints every line and inserts an empty line between runs of identical lines. It does not drop anything. It makes the runs visible. This is GNU. macOS uniq does not have it. The input still needs to be grouped already. Sort first if the equals are scattered.

```sh
sort -- file | uniq --group
```

People expect `--group` to find duplicates across the file. It only punctuates adjacent runs. People also want the blank line in a count. `-c` already gives the size of the run. `--group` is for reading. A file that is already one run gets no internal blank line. An empty input prints nothing. The separator is an empty line, which a later `uniq` will not treat as special beyond being a line of its own.

## Read NUL-separated records
Also asked as: uniq -z; zero terminated; uniq --zero-terminated; filenames with newlines; NUL records
`-z` uses a NUL byte as the record separator instead of a newline. A newline inside a record is ordinary text. This is GNU. It pairs with `find -print0` and `sort -z`. Without `-z`, a newline ends the record, and a filename that contains one becomes two records. macOS uniq has no `-z`.

```sh
sort -z -- file | uniq -z
```

People pass `-z` on one side of a pipeline and not the other. `sort` without `-z` still splits on newlines, and the NULs become garbage in the lines. Both sides need the same separator. `-z` does not imply `-d` or `-c`. Add those if you wanted them. A missing final NUL is still a record, the way a missing final newline is still a line. Do not mix the output with a line-oriented tool.

## Compare uniq with sort -u
Also asked as: sort -u versus uniq; unique sort; sort unique; do I need uniq; deduplicate a file
`sort -u` sorts and prints each distinct line once. `sort | uniq` does the same thing in two processes. `uniq` alone does not sort, and it keeps a later run of a line you already saw. Use `sort -u` when you want the set. Use `uniq` when the file is already grouped and you want counts, only-duplicates, or only-singletons.

```sh
sort -u -- file
```

People `uniq` a log and ship a report that still has duplicates. The lines were not adjacent. `sort -u` is the one-word fix when order does not need to stay. When order must stay and you still want a full-file unique, `uniq` cannot do it. `awk '!seen[$0]++'` is the usual order-preserving form. `uniq` will not become that. Counts require the `uniq -c` form, which needs the sort.

## Write the result to a file
Also asked as: uniq input output; uniq to a file; redirect uniq; two arguments; overwrite carefully
The second operand is the output file. `uniq file out` reads `file` and writes `out`. Redirecting is the same idea. Do not use the input path as the output path. `uniq` reads as it writes, and a same-file redirect truncates the input first. Write to a new name and rename.

```sh
uniq -- file > file.new
```

People `uniq file > file` and the file is empty. The shell opened the redirect before `uniq` ran. A second operand that names the input is the same trap. `-` as the input means stdin. An output of `-` is not special on every build. Use a redirect if you want stdout. The status is still 0 when the write succeeded. A permission error on the output is non-zero and the input is untouched only if you had not already truncated it yourself.

## What a failed uniq means
Also asked as: uniq exit status; uniq failed; uniq extra operand; cannot open; illegal option
A missing input exits non-zero. An unknown flag exits non-zero. Too many operands exits non-zero. A successful filter exits 0 even when it changed nothing, and even when every line was a duplicate. `uniq` does not have a "whether there were duplicates" status. The output is the report. `-d` that prints nothing is still 0.

```sh
uniq -- file
```

People test `$?` to see if the file had duplicates. It does not say that. Count the lines, or use `-d` and test whether it printed anything. Under `set -e`, a missing file aborts a script. An empty result does not. A pipe that closes early can make `uniq` exit non-zero with a broken pipe, depending on the shell. The adjacent rule is not an error. It is the definition. If the result looks wrong, sort the input before you blame the filter.
