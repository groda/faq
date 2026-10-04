# jq

`jq` reads JSON and writes JSON. The first argument is a filter, not a file. `.` is the filter that copies the value. With no file, `jq` reads standard input and waits. Several files are several JSON values, one after another, not one array. A missing key is `null`, not an error. jq 1.6 is assumed here. It runs on Linux and macOS. BusyBox does not include it. Exit status is 0 when the filter ran and the input parsed. A compile error is non-zero, often 3. A parse error is non-zero, often 4. `-e` changes the status again. No match is not a parse error. It is an empty output or `null`, and without `-e` the status is still 0.

Basic form: jq . -- file

Quote the filter. The shell eats `$`, quotes, and brackets. Single quotes are the usual wrapper. `--` before a file stops a leading dash from looking like a flag. `-` is standard input. The filter is not a path. `jq file` tries to compile `file` as a program and fails.

## Pretty-print a JSON value
Also asked as: jq pretty print; jq .; format json; jq a file; colorize json
`.` copies the input and pretty-prints it. Objects and arrays are indented. Object key order is preserved on jq 1.6 unless you pass `-S`. `-c` prints one compact line. `-C` forces color. `-M` turns color off. A terminal gets color by default. A pipe does not.

```sh
jq . -- file
```

People run `jq` with no filter and get a usage error. The filter is required. `.` is the identity. People also pipe colored output into a file and the color codes become junk. `-M` is the plain form. `jq` does not accept a trailing comma in the JSON. That is a parse error, status non-zero, and no value. A JSON text may contain several values. `jq` processes each one.

## Read a field
Also asked as: jq .foo; get a key; jq field; object key; missing key is null
`.foo` is the value of the key `foo`. `.foo.bar` walks nested objects. `.["foo"]` is the same lookup when the key has a space or a dash. A missing key yields `null`. It does not fail. `.foo.bar` on a missing `foo` also yields `null`, because `null` propagates through a lookup.

```sh
jq '.pattern' -- file
```

People write `jq .foo file` without quotes and the shell may expand `.foo` as a glob, or leave it. Quote the filter. A key that is not an identifier needs the bracket form. `.foo-bar` is subtraction, not a key. Use `.["foo-bar"]`. A field that is an object prints as an object. `-r` does not stringify it into a useful line. `-r` is for strings.

## Print a string without quotes
Also asked as: jq -r; raw output; strip quotes; jq tostring; shell variable from jq
`-r` prints a string as raw text, without JSON quotes and without escaping the contents into quotes. A number prints as digits either way. An object still prints as JSON. `-r` on a non-string walks the value and raw-prints any strings inside it, which surprises people who expected one line. Use `-r` when the next tool wants the text, not a JSON string.

```sh
jq -r '.pattern' -- file
```

People capture `jq '.name'` and the variable includes the quote characters. `-r` drops them. A string that contains a newline stays multi-line. A `null` prints the word `null` even with `-r`, unless you filter it. `@sh` is the formatter when the next step is a shell word. `-r` alone does not quote for the shell. A value with a space splits if you leave it unquoted in the shell.

## Iterate an array
Also asked as: jq .[]; array elements; walk an array; jq items; one value per element
`.[]` streams the elements of an array, or the values of an object. Each element is a separate output. `jq` prints them one after another. `. [0]` is the first element. The index is zero-based. A negative index counts from the end. `. [0]` on an empty array is `null`.

```sh
jq '.pattern[]' -- file
```

People write `.[].name` and expect one array back. They get one output per element. `[.pattern[].name]` collects them into an array. `map` does that on an array input. `.[]` on an object streams values, not keys. `keys` streams the keys if you iterate it. A JSON array of one object is still an array. Forgetting `[]` prints the whole array as one value.

## Select some elements
Also asked as: jq select; filter an array; jq map select; where; only matching objects
`select` keeps the input when its argument is true. `map` applies a filter to each element of an array and collects the results. `map(select(.pattern == "pattern"))` is the usual "keep some objects" form. `==` is equality. `!=` is the other. A select that keeps nothing produces no output, not `null`, when it is used on a stream.

```sh
jq 'map(select(.pattern == "pattern"))' -- file
```

People `select` an array and get nothing, because `select` saw the array, not each element. Iterate first, or `map`. `select(.foo)` keeps values that are truthy. `0`, `false`, `null`, and empty string are not. A missing field is `null` and fails that test. `==` does not coerce. `"1"` and `1` are not equal. The status is still 0 when nothing is selected, unless you passed `-e`.

## Build an object
Also asked as: jq construct object; jq {name}; make json; project fields; jq object literal
An object literal builds a value. `{name: .pattern}` uses `name` as the key and the filter as the value. `{pattern}` is shorthand for `{pattern: .pattern}` when the key matches the field. String keys need brackets or quotes in the key position. The result is one JSON object per input.

```sh
jq '{name: .pattern}' -- file
```

People write `{.foo: .bar}` and the parser rejects it. The key is not a filter unless you parenthesize it. `( .pattern ): .other` uses a filter as the key. A trailing comma is a syntax error. The output is still JSON. `-c` makes it one line, which is the form to embed in a log. Building an object does not remove the input. You named the fields you wanted. The others are dropped because you did not copy them.

## Pass a variable from the shell
Also asked as: jq --arg; --argjson; shell variable into jq; jq \$var; do not interpolate
`--arg name value` sets `$name` to a JSON string. `--argjson name value` sets `$name` to a parsed JSON value. The filter uses `$name`. It does not use the shell's `$name` if the filter is single-quoted. `--arg` is the one for text, so a value that looks like a number stays a string. `--argjson` is the one for `true`, numbers, and objects.

```sh
jq --arg name 'pattern' '.pattern == $name' -- file
```

People close the single quotes to insert a shell variable and the JSON breaks on the first quote in the value. `--arg` is the boundary. A missing `--arg` makes `$name` a compile error. `--arg` does not become a number. Compare with a string, or use `--argjson` for a number. The variable is visible to the whole filter, including `select`.

## Read several JSON values as one array
Also asked as: jq -s; slurp; ndjson to array; json lines; multiple documents
`-s` reads every JSON value in the input into one array, then runs the filter on that array. Newline-delimited JSON becomes one array of objects. Without `-s`, the filter runs once per value, and a filter that expects an array sees each object alone. Slurp holds the whole input in memory.

```sh
jq -s 'map(.pattern)' -- file
```

People `jq '.[0]'` on a stream of objects and get `null`, because each object has no index 0. `-s` makes the array first. A huge log slurped into an array can exhaust memory. Streaming the filter, `jq '.pattern'`, prints one result per line and does not hold the file. `-s` does not mean "compact." That is `-c`. Both can be set.

## Read lines that are not JSON
Also asked as: jq -R; raw input; jq from text lines; split a line; inputs
`-R` reads each line as a JSON string instead of parsing it as JSON. The newline is stripped. `-r` is the output flag. `-R` is the input flag. Together they turn a text file into text lines and back. `split(",")` cuts a line. `/pattern/` splits with a regex. This is not a CSV parser. Quotes in the line are characters.

```sh
jq -R 'split(",") | .[0]' -- file
```

People `jq -R . file` on JSON and get a string containing braces, not an object. `-R` disables the parser. Leave it off for JSON. A line with no newline at the end of the file is still a value. `inputs` is the filter form of "each input," useful with `-n`. `-n` means there is no input value, so a filter that expects one does nothing unless it uses `inputs` or `input`.

## Default a missing field
Also asked as: jq //; alternative operator; default value; null then default; missing key default
`//` yields the right-hand filter when the left is `null` or `false`. `.foo // "pattern"` is the default. It also replaces `false`, which is the surprise. `//` does not replace `0` or `""`. Those are present. For a missing key only, `if .foo == null then "pattern" else .foo end` is the stricter form.

```sh
jq '.pattern // "pattern"' -- file
```

People use `//` to mean "or" for numbers and a real `0` becomes the default. Use `== null` when zero is valid. `//` does not create the key in the input. It only changes the output of this filter. An error is not a missing key. A type error still fails. `.foo.bar // "pattern"` defaults when `.foo` is null, because the lookup yielded null. It does not default when `.foo` is a number. Indexing a number is an error.

## Add a column of numbers
Also asked as: jq add; sum an array; map numbers; reduce; total a field
`add` sums an array of numbers. `map(.pattern) | add` collects a field and sums it. `add` of an empty array is `null`. A string in the array is a type error. There is no silent skip. `tonumber` converts a numeric string. It errors on text that is not a number.

```sh
jq 'map(.pattern) | add' -- file
```

People `jq '.n'` on a stream and see one number per line, not a total. The total needs `map` or `-s` and `add`. A missing field becomes `null` in the `map`, and `add` then fails or yields null depending on the values around it. `map(.pattern // 0)` treats missing as zero. It also treats `false` as zero. A JSON number is not currency. jq uses IEEE floats. Large integers can lose low bits.

## Sort and unique
Also asked as: jq sort; jq unique; sort_by; order by field; dedupe
`sort` sorts an array. Numbers sort as numbers. Strings sort as strings. `sort_by(.pattern)` sorts objects by a field. `unique` sorts and drops duplicates. `unique_by(.pattern)` drops duplicates by a field. These want an array. A stream needs `[.]` or `-s` first. The sort is stable enough for a report and is not a locale collation you can set.

```sh
jq 'sort_by(.pattern)' -- file
```

People `sort` an object and get keys or an error, depending on the filter. Collect the values into an array, then sort. `unique` without `_by` compares the whole value. Two objects that differ by an unused field are not duplicates. `sort` does not print one element per line unless you iterate afterwards. `| .[]` streams the sorted array.

## Group rows
Also asked as: jq group_by; group by field; count by key; bucket objects; jq grouping
`group_by(.pattern)` sorts an array and splits it into arrays of equal key. The input must be an array. Each group is an array of the original elements. `map({key: .[0].pattern, count: length})` is the usual summary after a group. `group_by` reorders. It is not a streaming fold.

```sh
jq 'group_by(.pattern) | map({key: .[0].pattern, n: length})' -- file
```

People `group_by` a stream of objects and jq errors, because the filter received one object. `-s` or `[inputs]` builds the array. A missing key groups as `null`. All of those rows land in one bucket. `group_by` holds the whole array. A multi-gigabyte log is the slurp warning again. Count in a tool that streams if the file does not fit.

## Join a string
Also asked as: jq join; implode; array to string; @csv; @tsv; output a line
`join(",")` concatenates an array of strings with a separator. Numbers need `map(tostring)` first, or `join` errors. `@csv` formats an array as a CSV row, quoting fields. `@tsv` uses tabs. `@sh` formats shell words. These are output formats. `-r` is required or you get a JSON string with quotes around the whole row.

```sh
jq -r '[.pattern, .pattern] | @tsv' -- file
```

People `join` without `-r` and the commas are inside a JSON string. The next tool sees the quotes. `@csv` quotes a field that contains a comma. It is the formatter to use, not `join(",")`, when the data may contain the separator. `@sh` is still not a reason to `eval` untrusted JSON. It quotes. It does not make the content safe to run.

## Update a value
Also asked as: jq assign; set a field; .foo = 1; update object; jq rewrite
`.pattern = "pattern"` sets a key and passes the whole object on. `.pattern += 1` adds. `|=` updates in place of a filter. The input is the whole value, so you still print the object, not only the field. Assignment in jq does not edit the file. It edits the value on the way to standard output.

```sh
jq '.pattern = "pattern"' -- file
```

People expect the file to change. Redirect to a new file and rename it. `jq file > file` empties the file, the same trap as any redirect. A set on a missing key creates it. A set on a missing parent fails, because there is no object to put the key on. Build the parent, or use a filter that creates the path. `+=` on `null` fails. Default first.

## Use a filter file
Also asked as: jq -f; filter from a file; jq program file; long filter; jq -f script
`-f` reads the filter from a file. The data files come after. The filter file is not JSON. It is the program. You do not single-quote the contents. The shell never sees them. `-f` can be combined with `--arg`. A syntax error names the filter file.

```sh
jq -f file -- file
```

The two file arguments are easy to swap. The one attached to `-f` is the program. `jq -f file` with no data file reads JSON from standard input and waits on a terminal. It is not a usage error. Comments in a filter file are `#` to the end of the line, which is how a long program stays readable. A `#` inside a string is a character.

## Exit non-zero when the result is empty
Also asked as: jq -e; exit status; jq in a test; false is failure; null exit code
`-e` sets the status from the last output. `null` and `false` exit 1. No output exits 1. Any other value exits 0. This is the flag for a script that must fail when the field is missing. Without `-e`, a missing key prints `null` and exits 0. A parse error is still a parse error. `-e` does not replace that.

```sh
jq -e '.pattern' -- file
```

People use `-e` on a filter that prints several values. The status is the last one. An earlier `null` does not fail the run if the last value is a string. `select` that keeps nothing produces no output, and `-e` then exits 1. That is the "no match" test. `grep` is still clearer for a text log. `-e` is for a JSON test. Under `set -e`, this aborts the script on a missing field. That is what you asked for.

## What a failed jq means
Also asked as: jq exit code; parse error; compile error; jq failed; invalid numeric literal
A filter that does not compile exits non-zero, often 3, and names the token. A file that is not JSON exits non-zero, often 4, and names the line. `jq` stops at the first bad value. Later values in that stream are not processed. A missing key is not this failure. It is `null`. `jq -e` is the way a missing key becomes a status.

```sh
jq . -- file
```

People see `null` and think the file failed to parse. The file parsed. The key was absent. People see a parse error on a file that is newline-delimited JSON with a blank line. A blank line is not a JSON value. Strip it, or use a stream the producer actually wrote. Status 0 with `-c` output is a successful compact print, not a test that a field matched. The output is the result. The status is the parse and the `-e` rule.
