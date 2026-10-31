# awk

`awk` reads records, splits them into fields, and runs a program on each one. It does not edit the file. It writes to standard output. With no file, it reads standard input and waits. A record is a line unless you change `RS`. A field is a run of non-blank text unless you set `-F` or `FS`. `$1` is the first field. `$0` is the whole record. GNU awk, `gawk`, on Linux is assumed here. Debian and Ubuntu often point `awk` at `mawk`, which is enough for these recipes and lacks `gensub`, `FPAT`, and inplace editing. macOS `awk` is BSD nawk and also lacks those. BusyBox awk is smaller again. Exit status is 0 when the program ran. It is non-zero when the program is illegal, a named file cannot be read, or the script calls `exit` with a non-zero value. No matching line is not an error.

Basic form: awk '{ print }' -- file

The program is the first non-option argument unless you pass `-f` or `-e`. Later arguments are files, or `name=value` assignments that apply as those arguments are reached. Single-quote the program so the shell does not expand `$1`. `--` before a file stops a leading dash from looking like a flag. A pattern before an action selects records. No pattern means every record. No action means print the record. `{ print }` is that default, written out.

## Print a column
Also asked as: awk print field; awk '{print $1}'; first column; second field; awk NF
`$1` is the first field, `$2` the second. `NF` is how many fields this record has. `$NF` is the last field. Default splitting treats a run of spaces or tabs as one separator, so two spaces do not make an empty field. `print` writes the fields you name, separated by a space, and then a newline. A record with no fields prints a blank line if you print `$1`, because `$1` is empty.

```sh
awk '{ print $1 }' -- file
```

People write `awk '{ print $1 }' file` and the shell eats nothing only because of the quotes. Without quotes, `$1` is the shell's first argument and awk sees a broken program. A column of numbers is still text. `print $1` does not add them up. A trailing separator does not create a last empty field under the default split. `-F,` does. Know which separator you are in before you count fields.

## Set the field separator
Also asked as: awk -F; awk field separator; split on comma; awk -F,; FS; tab separated
`-F` sets the input field separator for the whole run. `-F,` splits on each comma. Consecutive commas produce an empty field, so `a,,b` has three fields. `-F:` splits on colons. A tab is `-F '\t'`. The separator is a regular expression in awk, so `.` means any character unless you escape it. `FS` is the same variable, set in `BEGIN` if you prefer.

```sh
awk -F, '{ print $2 }' -- file
```

The output separator is not `-F`. `print $1, $2` joins with a space, the default `OFS`. Set `OFS` if the output should be a comma too. People set `-F` and expect `print` to keep the commas. It does not. A CSV file with quotes and commas inside quotes is not split correctly by `-F,`. awk is not a CSV parser. GNU awk 5.3 and later have `--csv`. Do not pretend `-F,` is that.

## Filter lines
Also asked as: awk pattern; awk '/pattern/'; print matching lines; awk condition; drop lines
A pattern with no action prints the matching record. `/pattern/` is a regular expression. `NF == 3` is a field count. `$2 == "pattern"` is an exact field. The test is not a shell glob. A dot matches any character. Anchor with `^` and `$` when the whole field must match. No match prints nothing and exits 0.

```sh
awk -F, '$2 == "pattern"' -- file
```

People write `$2 == pattern` without quotes and awk treats `pattern` as an unset variable, which compares equal to an empty field. Quote the text. `/pattern/` searches the whole record, not one field. A pattern in field 2 only is `$2 ~ /pattern/`. awk prints the record, not the match alone. `grep` is the tool when you only want the matching lines and you do not need fields.

## Add a column of numbers
Also asked as: awk sum; total a column; awk '{s+=$1}'; add numbers; running total
`+=` adds the field to a variable. Unset variables are 0. The addition belongs in the per-line action. The print belongs in `END`, which runs after the last record. Without `END`, the print runs every line and you get a running total instead of one answer. Non-numeric text adds as 0.

```sh
awk '{ s += $1 } END { print s }' -- file
```

People print `$1` and think awk totaled it. `print` does not add. A thousands separator or a `$` in the field makes the value numeric only up to the first bad character, so `1,000` adds as 1. Strip the comma, or the total is quietly wrong. `END` still runs if the file was empty, and the sum is 0. A missing file never reaches `END`. It exits non-zero.

## Count lines and fields
Also asked as: awk NR; awk NF; number of lines; count fields; FNR; records so far
`NR` is the number of records read so far, across every file. `FNR` is the number in the current file. `NF` is the number of fields in this record. `END { print NR }` prints how many records were read. It counts a final record that has no trailing newline. It does not count a completely empty file as one record.

```sh
awk 'END { print NR }' -- file
```

People use `wc -l` and awk on the same file and disagree when the last line has no newline. `wc -l` counts newline characters. awk counts records. `NF` on a blank line is 0. On a line of spaces it is also 0, because the default separator eats the spaces. With `-F,`, a line of commas has fields. `NR` is not the line number inside one file if you passed more than one file. That is `FNR`.

## Change a field and print the line
Also asked as: awk change field; assign $2; rebuild the line; OFS; edit a column
Assigning to `$2` changes that field and rebuilds `$0` using `OFS` between fields. The default `OFS` is a space. The original separators are not kept. Set `OFS` in a `BEGIN` block if the output must be commas or tabs. Printing `$0` after the assignment prints the rebuilt line.

```sh
awk -F, 'BEGIN { OFS = "," } { $2 = "pattern"; print }' -- file
```

People assign `$2` and the commas become spaces, because they set `-F` and not `OFS`. Input separator and output separator are different variables. An assignment to `$1` on a line with one field still prints that field. An assignment to a field past `NF` creates empty fields in between. awk does not quote a field that contains the separator. Rebuilding CSV this way breaks as soon as a value contains a comma.

## Run something before or after the file
Also asked as: awk BEGIN; awk END; header row; skip the header; before reading; after the last line
`BEGIN` runs before any record is read. `END` runs after the last record, or after a fatal input error stops the read. A `BEGIN` block is where you set `FS`, `OFS`, and counters. A pattern of `NR == 1 { next }` skips a header. `next` stops processing this record and starts the next one. `END` is where a total belongs.

```sh
awk 'NR == 1 { next } { print }' -- file
```

People put the header test after an action that already printed. Actions run in order, and an earlier unconditional `{ print }` already fired. `next` in the header rule skips the later actions. `BEGIN` does not see `$1`. There is no record yet. A file that cannot be opened does not skip `BEGIN`, and it may skip `END` if the failure is immediate. Do not use `END` as proof that every file was read.

## Print a fixed format
Also asked as: awk printf; format a column; awk %s; width and precision; do not print extra spaces
`printf` writes the format you give and does not add a newline unless you write `\n`. `%s` is a string. `%d` is an integer. A width pads. Unlike `print`, `printf` does not insert `OFS` between arguments. Unused arguments are ignored. A missing argument for a conversion is a runtime error on GNU awk.

```sh
awk '{ printf "%s\t%s\n", $1, $2 }' -- file
```

People use `print` and then try to line up columns with spaces. The separators are `OFS`, and the columns drift. `printf` is the aligned form. A `%d` on a non-numeric field prints 0 or fails the conversion, depending on the awk. Quote the format so the shell does not eat `%`. `printf` does not print a header. A `BEGIN` block prints the header once.

## Match a field with a regular expression
Also asked as: awk tilde; $1 ~ /pattern/; field matches; awk regex; not match !~
`~` is the match operator. `$2 ~ /pattern/` is true when field 2 contains a match. `!~` is the opposite. The pattern is a regular expression, so `.` and `*` are special. Anchor with `^` and `$` for a whole-field match. A match anywhere in the field succeeds without anchors. Slash-delimited patterns do not need extra quotes inside the program.

```sh
awk '$2 ~ /^pattern$/ { print }' -- file
```

People write `$2 ~ pattern` and awk uses the variable `pattern`, usually empty, which matches everything. A regex in a variable is `$2 ~ pattern` only after you set `pattern` to a string. Case is significant unless you are on GNU awk and set `IGNORECASE = 1`. That variable is a gawk extension. BSD awk and mawk do not honor it. A character class is `[0-9]`, not `\d`, in a portable script.

## Replace text in a line
Also asked as: awk sub; awk gsub; replace in a field; substitute; gensub
`sub` replaces the first match in the target. `gsub` replaces every match. The target defaults to `$0` if you omit it. The replacement is a string. `&` in the replacement is the whole match. A changed `$0` is re-split into fields. `sub` returns how many replacements it made. GNU awk also has `gensub`, which returns the new string and can use `\\1`. `gensub` is not in mawk or BSD awk.

```sh
awk '{ gsub(/pattern/, "pattern"); print }' -- file
```

People expect `sub` to print the line. It only changes it. You still `print`. A replacement written with `$1` inside double quotes is not the field. The field is concatenated outside the quotes, or you build the string another way. `gsub` on `$2` changes that field and then rebuilds `$0` with `OFS`. The same output-separator surprise as an assignment applies.

## Pass a variable in from the shell
Also asked as: awk -v; pass a shell variable; awk var=value; do not interpolate; -v versus ENVIRON
`-v name=value` sets an awk variable before `BEGIN`. The shell should expand it in double quotes, once, outside the program. The program stays in single quotes and uses the name. `ENVIRON["name"]` reads an environment variable and is the other channel. A `name=value` argument between files applies when awk reaches that argument, which is later than `-v`.

```sh
awk -v name='pattern' '{ print name, $1 }' -- file
```

People put `"$name"` inside the single-quoted program and awk sees the literal `$name`, or they close the quotes badly and the shell splits the program. `-v` is the clean boundary. A value that starts with `-` is fine as `-v name=...` and is not a file. A value with a backslash is a string escape to gawk's `-v` in some versions. If the value is arbitrary, `ENVIRON` is the safer carrier.

## Read the program from a file
Also asked as: awk -f; awk script file; program in a file; awk -f prog; shebang awk
`-f` reads the program from a file. The data files come after. The program file is not quoted by the shell, so `$1` in it is awk's field, which is what you want. You can pass more than one `-f`. A syntax error names the program file and the line. That is the reason to leave a long program out of the one-liner.

```sh
awk -f file -- file
```

The two file arguments are easy to swap. The one attached to `-f` is the program. A script that starts with `#!/usr/bin/awk -f` is the same mechanism. It does not receive `-v` unless the caller passed it. `awk -f` with no data file reads standard input and waits on a terminal. It is not an error. It is a missing pipe.

## Skip to the next record
Also asked as: awk next; skip a line; continue in awk; nextfile; drop this record
`next` stops the actions for this record and reads the next one. Later pattern-action pairs do not run for the record you skipped. It is how a header, a comment, or a short line gets out of the way. `nextfile` stops the current file. It is common, but it is not in every old awk. GNU awk and current BSD awk have it.

```sh
awk 'NF < 2 { next } { print $2 }' -- file
```

People use `next` and then wonder why `END` did not run. `next` skips to the next record. It does not skip `END`. `exit` does leave the record loop, and it does run `END`. A `next` in `END` is an error. A short line that you forgot to skip still has empty fields, and `$2` prints blank. The `next` has to be in an earlier rule than the print.

## Split one file from the next
Also asked as: awk FNR; two files; FILENAME; NR versus FNR; per file
`FILENAME` is the name of the file being read. `FNR` resets at the start of each file. `NR` does not. `FNR == 1` is true on the first record of every file, which is the place to reset a per-file total. A `name=value` argument between two filenames does not reset `FNR`. It is an assignment, not a file.

```sh
awk 'FNR == 1 { print FILENAME } { print }' -- file file
```

People store a total in `NR` and the second file continues the count. That is what `NR` is for. Use `FNR` for a line number in the current file. Standard input has a `FILENAME` of `-` if you named `-`, and it may be empty if awk opened stdin itself. `FILENAME` is not updated in `BEGIN`. No file is open yet.

## Print to a different file
Also asked as: awk redirect; print to a file; write from awk; close a file; split output
`print > "file"` writes that file from inside awk. The first write truncates it. Later writes to the same name append. `>>` appends from the first write. The filename is a string, so quote it. `close("file")` flushes and closes it, which matters if another program must read it before awk exits. This redirect is awk's. It is not the shell's `>`.

```sh
awk '{ print > "file" }' -- file
```

The shell does not see the inner `>`. Quoting the program in single quotes is what keeps it that way. A redirection in the shell, `awk '{ print }' > file`, captures all of awk's standard output instead. Those are different. awk will happily create one file per key if the string changes, and it will hit the open-file limit if you never `close`. GNU awk raises the limit. A small awk should close.

## Use a different record separator
Also asked as: awk RS; record separator; paragraph mode; RS blank line; not a line
`RS` is the record separator. The default is a newline. Setting `RS` to the empty string makes a record a paragraph, separated by blank lines, and each line inside the paragraph becomes a field. Setting `RS` to a character splits on that character. GNU awk allows a regular expression as `RS`. That is not portable to every awk.

```sh
awk 'BEGIN { RS = "" } { print NF }' -- file
```

People set `RS` to `\n\n` expecting paragraphs. A blank line is the portable paragraph mode, and it is the empty `RS`, not two newlines. Fields then follow the default field rules inside the paragraph. `ORS` is the separator `print` writes. If you change `RS` and not `ORS`, the output still ends records with newlines. A missing final separator still yields a final record, the same way a missing final newline does.

## What a failed awk means
Also asked as: awk exit status; awk exit code; syntax error; awk failed; exit from awk
A syntax error exits non-zero and names the line it could not parse. A missing file exits non-zero. A program that prints nothing exits 0. `exit 1` from the script is how you report "no match" or "bad data," and `END` still runs when you `exit`. The status you pass to `exit` is the status awk returns, unless a later error overrides it.

```sh
awk 'NF < 1 { exit 1 } { print }' -- file
```

People test `$?` after a filter and treat 0 as "lines matched." It means the program finished. `grep` is the command with a no-match status. Under `set -e`, a syntax error aborts, and an empty filter does not. `exit` in `BEGIN` still runs `END` on GNU awk. A variable you meant to return is not the status. Only `exit` and a real failure set it.
