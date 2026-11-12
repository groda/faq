# cut

`cut` extracts columns from each line of a file. It splits a line on a delimiter and prints the fields you name, or it prints a character range. It does not match patterns, and it does not rearrange whole lines. With no file it reads standard input and waits. GNU cut from coreutils on Linux is assumed here. macOS cut is BSD. It has `-d`, `-f`, `-c`, and `-s`, and it may lack `--complement` or `-z`. BusyBox cut is thinner and usually has `-d`, `-f`, and `-c`. Exit status is 0 when the selected parts were printed. It is non-zero when a named file cannot be read or the options are illegal. A line with fewer fields than you asked for is success and prints what it has.

Basic form: cut -d: -f1 -- file

The first non-option arguments after the flags are files. A name that starts with `-` is a flag unless it comes after `--`. `-d` takes the delimiter as the next argument. Quote a delimiter that the shell would eat. Fields and character positions count from 1. A line that contains no delimiter is printed whole, unless you pass `-s`. Several files are concatenated in the output with no headers.

## Cut a column from a CSV or a colon file
Also asked as: cut -d -f; cut a field; extract a column; cut -d: -f1; get the first field
`-d` sets the delimiter. `-f` selects the fields, counting from 1. `-f1` is the first field. The default delimiter is a tab, not a space and not a comma. A colon-separated file needs `-d:`. A CSV needs `-d,`. Quote the delimiter if it is a shell metacharacter. The selected fields are joined with the same delimiter in the output.

```sh
cut -d: -f1 -- file
```

People pass `-d` and forget `-f`, and cut errors. You need both for fields. People also expect cut to understand quoted CSV. It does not. A comma inside quotes is still a delimiter. Use a real CSV tool for that. A line with no delimiter at all is printed unchanged. That surprises people who wanted only the first column. `-s` drops those lines. The field numbers are 1-based. `-f0` is an error.

## Cut several fields, or a range
Also asked as: cut -f1,3; cut a range; -f1-3; select multiple columns; cut fields list
`-f1,3` prints field 1 and field 3. `-f1-3` prints fields 1 through 3 inclusive. `-f2-` prints from field 2 to the end. `-f-3` prints from the start through field 3. The ranges can be mixed. Output keeps the original order of fields, not the order you listed them in, on GNU cut. A missing field in the middle is skipped.

```sh
cut -d: -f1,3 -- file
```

```sh
cut -d: -f1-3 -- file
```

Use the first for specific columns. Use the second for a contiguous range. People write `-f1-3` expecting to exclude field 3. The range is inclusive. People also list fields out of order and expect that order in the output. GNU cut emits them in ascending position. A line that ends with the delimiter has an empty final field. It counts. A request past the end of a short line prints the fields that exist and does not error.

## Cut by character position
Also asked as: cut -c; cut characters; fixed width; -c1-10; column by position
`-c` selects by character position, counting from 1. `-c1-3` is the first three characters. `-c5-` is from the fifth character to the end. `-c-8` is the first eight. This does not look at delimiters. It is the form for fixed-width data. A multibyte UTF-8 character counts as one character on GNU cut when the locale is UTF-8. Bytes are `-b`.

```sh
cut -c1-10 -- file
```

People use `-c` on a delimited file and get a slice of the text, not a field. The delimiter is ignored. People also expect `-c` to be bytes. In a UTF-8 locale it is characters. `-b` is bytes, and it can split a multibyte character. A short line is printed whole if the range starts within it. A range that starts past the end prints nothing for that line. You cannot combine `-c` and `-f` in one run. Cut requires exactly one of `-b`, `-c`, or `-f`.

## Change the delimiter
Also asked as: cut -d; set delimiter; cut comma; cut tab; delimiter is space
`-d` takes one character as the field separator. `-d,` is a comma. `-d:` is a colon. The default is a tab. A space delimiter is `-d' '`. Only one character is allowed. A multi-character separator is not supported. The same character is used to join the output fields, unless you set `--output-delimiter`.

```sh
cut -d, -f1 -- file
```

People write `-d` with a string and cut errors or uses only the first character. The delimiter is one byte. People also expect whitespace of any kind to be the delimiter. A run of spaces is not collapsed. Each space is a separator, so empty fields appear between them. For whitespace-separated columns, `awk` is the usual tool. Cut is exact. A tab in the data with `-d' '` is not a break. It stays inside the field.

## Drop lines that have no delimiter
Also asked as: cut -s; only delimited; skip lines without delimiter; cut --only-delimited; suppress non-matching
`-s` prints only lines that contain the delimiter. A line with no delimiter is dropped. Without `-s`, that line is printed whole. This is useful when a file mixes comments or headers that do not have the separator. The flag applies to `-f`. It has no meaning with `-c`.

```sh
cut -d: -f1 -s -- file
```

People see a header printed in full and think cut failed to find the field. The header had no colon, so it was passed through. Add `-s`. People also expect `-s` to drop empty fields. It does not. It only cares whether the delimiter appears at least once on the line. A line that is only a delimiter is delimited and is kept. Its fields are empty. The exit status is still 0 when every line was dropped.

## Keep the rest of the line after a field
Also asked as: cut from field to end; -f2-; rest of the line; cut everything after; keep the tail
`-f2-` selects from field 2 through the last field. The trailing dash means "to the end." This is how you drop a leading column and keep everything else, including any delimiters that remain. `-f1` alone would keep only the first. The range form keeps the tail intact.

```sh
cut -d: -f2- -- file
```

People cut field 1 and then try to reconstruct the rest. `-f2-` is the direct form. People also write `-f2` and lose every field after the second. A single number is one field. The dash is required for the range. A line with only one field has nothing from 2 onward, so it prints empty under this selection, unless you also omitted `-s` and the line had no delimiter at all. Check a short line.

## Complement the selection
Also asked as: cut --complement; invert fields; all but these columns; cut everything except; GNU complement
`--complement` prints the fields or characters you did not name. `--complement -f1` is everything except field 1. This is GNU. macOS cut may not have it. The complement is of the selection, not of the line. Delimiters between the kept fields are preserved. It works with `-f`, `-c`, or `-b`.

```sh
cut -d: -f1 --complement -- file
```

People want "all but the first" and reach for a pipeline. On GNU cut the flag is `--complement`. People also expect the complement to reorder or to drop the delimiter. It keeps the relative order and the separators of the remaining fields. A request that complements every field prints nothing for that line. Combine it with `-s` if you also want to drop undelimited lines. The flag is long-only. There is no short form.

## Change the output delimiter
Also asked as: cut --output-delimiter; join with a different separator; output separator; change delimiter on output
`--output-delimiter` sets the string written between selected fields. The default is the input delimiter. Use it when you want to turn colons into commas, or into a space. The string can be more than one character. This is GNU. It does not change how the line is split. Only the joining changes.

```sh
cut -d: -f1,3 --output-delimiter=, -- file
```

People change `-d` and expect the output to use a different character. `-d` is the input split. The output uses the same character unless you set this. People also put the new delimiter in a place cut reads as a field list. The option takes its argument after the equals or as the next word, depending on how you write it. A selected single field has no delimiter to replace. The option only appears between fields.

## Cut bytes instead of characters
Also asked as: cut -b; cut bytes; byte range; -b1-4; binary cut
`-b` selects by byte offset, counting from 1. `-b1-4` is the first four bytes. This can split a multibyte character. Use it for a fixed binary header or when the data is known to be single-byte. `-c` is the character form. You cannot use both in one invocation.

```sh
cut -b1-4 -- file
```

People use `-b` on UTF-8 text and the output is invalid UTF-8 at the cut point. Prefer `-c` for text. People also expect `-b` to understand a delimiter. It does not. It is a pure byte slice of each line, not including the newline. A short line is handled like the character form. The newline that ends the record is not part of the byte count. It is re-added on output.

## Read from a pipe
Also asked as: command | cut; cut stdin; cut a pipeline; no file cut; cut from standard input
With no file, or with `-`, cut reads standard input. It applies the same selection to each line the producer writes. It does not wait for the whole stream before starting. A producer that never closes makes cut wait on the next line. The selection rules are identical to a file.

```sh
command -- file | cut -d: -f1
```

People pipe a command that prints a header and the header passes through or is cut, depending on whether it contains the delimiter. There is no special header handling. People also expect cut on a pipe to exit when they have seen enough. It reads until the producer closes. Use `head` before it if you want a prefix of the stream. A cut of an empty pipe prints nothing and exits 0.

## Handle a trailing delimiter
Also asked as: cut trailing delimiter; empty last field; line ends with delimiter; cut empty field; final colon
A line that ends with the delimiter has an empty field after it. `-f1-` includes that empty field. The output will end with the delimiter if the last selected field is empty. Cut does not strip a trailing delimiter. It treats it as separating a final empty field. This is easy to miss when you count columns by eye.

```sh
cut -d: -f1- -- file
```

People count the visible columns and ask for one more than exists, then wonder about a trailing delimiter in the output. The empty field is real. People also want cut to drop empty fields. It does not. An empty field is preserved. A tool that collapses separators is `awk` with a different field separator, or a dedicated CSV parser. Cut is literal about the characters you named.

## Cut one character from each line
Also asked as: cut a single character; -c1; first character of each line; cut one column; extract a letter
`-c1` prints the first character of each line. `-c5` prints the fifth. A single number is a range of length one. The newline is not counted. A line shorter than the position prints nothing for that line. This is the character form of a one-field cut.

```sh
cut -c1 -- file
```

People use `-f1` for this and get a whole field instead of one character. `-c` is positions. `-f` is delimited fields. People also expect `-c1` to skip a leading space. It does not. Position 1 is the first byte or character, whatever it is. A blank line has no character to print, so the output line is empty. The line count is preserved. You still get one output line per input line.

## Use it on /etc/passwd style files
Also asked as: cut passwd; cut -d: /etc/passwd; extract username; colon separated; system file columns
A file like `/etc/passwd` is colon-separated. `-d: -f1` is the username. `-f3` is the uid. `-f7` is the shell. The same pattern works for any colon- or comma-separated system file that does not quote fields. Cut does not understand the meaning of the columns. It only counts them.

```sh
cut -d: -f1 -- /etc/passwd
```

People cut `/etc/passwd` and a line with fewer colons shifts the fields. Cut does not validate the format. People also expect it to skip comments. A comment line with no colon is printed whole, or dropped with `-s`. There is no comment syntax. A NIS or a systemd user line that looks different will be cut the same way. Check the field count on a sample line before you script it.

## Combine with sort or uniq
Also asked as: cut and sort; extract and unique; cut | sort -u; column then sort; pipeline cut
Cut emits one field per line when you select a single field. That stream is ordinary text. `sort -u` deduplicates it. `sort | uniq -c` counts it. The cut does not sort. The later command does. Quote nothing special beyond the usual. The pipeline status is the last command unless `pipefail` is set.

```sh
cut -d: -f1 -- file | sort -u
```

People expect cut to unique the values. It does not. It preserves duplicates and order. People also cut several fields and then sort, and the sort key is the whole remaining line. Set the sort key if you meant only one of the remaining columns. A cut that drops the delimiter between fields changes what sort sees. Keep the delimiter if the next tool needs columns.

## Cut a fixed-width record
Also asked as: cut fixed width; -c for columns; character columns; extract positions; fixed format
When the data has no delimiter and the columns are at known character positions, `-c` is the tool. `-c1-8` is the first column, `-c10-20` is the next, if you run cut twice or accept that one cut takes one selection. One invocation prints one selection. To extract two non-contiguous character ranges you can run cut twice and paste, or use a different tool.

```sh
cut -c1-8 -- file
```

People try to pass two ranges to `-c` and get them concatenated with nothing between. `-c1-8,10-20` prints those characters in order, jammed together. There is no delimiter inserted because there was no field split. People also count screen columns and the data has tabs. A tab is one character to `-c` in terms of the count, but it displays wider. Count characters in the file, not columns on the screen.

## Suppress the whole line when the field is missing
Also asked as: cut missing field; fewer fields; short line cut; field does not exist; cut past the end
A request for a field that does not exist on a line prints nothing for that field. If you asked for only that field, the output line is empty. The line is still emitted. Cut does not drop the line. `-s` drops only lines that lack the delimiter entirely. A line with the delimiter but too few fields is still delimited.

```sh
cut -d: -f5 -- file
```

People expect a missing field to omit the line. It omits the content and keeps the line. People also expect an error. The status is 0. A later tool that reads the output will see blank lines. Filter them if you need to, or choose a field that is always present. Counting fields with `awk -F: 'NF < 5'` is the way to find the short lines. Cut will not report them.

## Read NUL-separated records
Also asked as: cut -z; zero terminated; cut --zero-terminated; filenames with newlines; NUL records
`-z` uses a NUL byte as the record separator instead of a newline. A newline inside a record is ordinary text. This is GNU. It pairs with `find -print0` and other NUL streams. Without `-z`, a newline ends the record. macOS cut may not have `-z`. The field and character selection then applies inside each NUL-terminated record.

```sh
cut -z -d: -f1 -- file
```

People pass `-z` on cut and not on the producer. The records disagree and the output contains raw NULs or split lines. Both sides need the same separator. `-z` does not change the field delimiter. You still set `-d` for fields. A missing final NUL is still a record, the way a missing final newline is still a line. Do not mix the output with a line-oriented tool without converting.

## Cut the first field and keep the delimiter style
Also asked as: cut preserve delimiter; output looks like input; same separator; cut and join; field separator out
By default the output delimiter is the input delimiter. Selecting `-f1,2` from a colon file produces colon-separated output. You do not need an extra join. `--output-delimiter` changes it only when you want a different one. A single field has no join to do.

```sh
cut -d: -f1,2 -- file
```

People pipe cut into `paste` or `tr` to put the delimiter back. It was never removed between the fields you kept. People also select non-contiguous fields and expect the original intervening fields' delimiters to remain as gaps. They do not. Only the selected fields are emitted, joined by the output delimiter. The skipped field is gone, delimiter and all.

## What a failed cut means
Also asked as: cut exit status; cut failed; cut illegal option; cannot open; cut error
A missing file exits non-zero. An illegal option, or combining `-f` and `-c`, exits non-zero. A delimiter that is more than one character exits non-zero on GNU cut. A successful cut exits 0 even when it printed nothing, and even when every line lacked the field. Cut does not have a "whether the column existed" status. The output is the report.

```sh
cut -d: -f1 -- file
```

People test `$?` to see if any lines matched the shape. It does not say that. An empty result can be an empty file, a high field number, or `-s` dropping everything. People also forget `--` before a file named `-f` and cut reads it as a flag. The status is a usage error. Under `set -e`, a missing file aborts a script. A cut that produces blank lines does not. Check the output if a later step assumes every line has content. A usage error prints a message on stderr and leaves stdout empty.
