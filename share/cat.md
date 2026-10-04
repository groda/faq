# cat

`cat` reads files and writes them to standard output, one after another. It does not search, it does not number lines unless you ask, and it does not create a file unless the shell redirects the output. With no file, or with `-`, it reads standard input and waits until that input ends. Several files are concatenated in the order you named them. A directory operand is an error. GNU cat from coreutils on Linux is assumed here. macOS cat understands `-n`, `-b`, `-s`, `-v`, `-e`, and `-t`. Its long options are thinner. BusyBox cat usually copies files and stdin and may lack `-A` and `-s`. Exit status is 0 when every named file was read. It is non-zero when a named file is missing or unreadable. Output already written is not taken back. A binary file is not a failure. It is bytes, and the terminal will show them.

Basic form: cat -- file

Operands are paths. A name that starts with `-` is a flag unless it comes after `--`. `-` alone is stdin. The shell expands globs before `cat` sees them. Quote a name with spaces. `cat` does not know about lines until you use a flag that counts them. The bytes go out as they are, including NULs.

## Print a file
Also asked as: how to show a file; cat a file; print file contents; dump a file; type a file
`cat` writes the file to standard output and adds nothing. No header, no line numbers, no trailing newline that was not already there. A short file scrolls by. A missing file prints an error on stderr and exits non-zero. If you named several files, GNU cat still prints the ones it can read.

```sh
cat -- file
```

People use `cat` to page a long file. It does not page. The terminal scrolls, and the top is gone. `less -- file` is the viewer. `cat` is the right tool when the next stage is a pipe or a redirect. A file with no final newline is printed without one, so the next shell prompt can sit on the same line. That is the file, not `cat` eating a character.

## Concatenate files
Also asked as: cat file1 file2; join files; concatenate; combine text files; cat several files
Each operand is written in order, with nothing inserted between them. The result is one stream. `cat` does not add a newline at the boundary. If the first file lacks a trailing newline, the first line of the second file continues it. Directories in the list are errors and are skipped.

```sh
cat -- file file
```

People expect a separator or a header with the file name. `cat` does not print one. `tail` and a loop do, if that is what you want. A glob such as `*.txt` is expanded by the shell in alphabetical order, not in the order the files were created, and it skips dotfiles. `cat -- *.txt` fails the way any command fails when the glob matches nothing and the shell leaves the literal star.

## Read standard input
Also asked as: cat no arguments; cat stdin; cat a pipe; cat -; copy stdin to stdout
With no operand, `cat` reads standard input until end of file and writes it to standard output. `-` means the same thing when you also pass files. A terminal does not end until you send end of file. A pipe ends when the producer closes it. This is how `cat` copies a stream.

```sh
cat -- -
```

People run `cat` with no file in a script and the command sits there. It is waiting for stdin, not reading the current directory. `cat file - file` writes the first file, then stdin, then the second file. Data from the pipe does not go back and fill in the first file. `cat` is not required in a pipe. `cmd | other` already connects the stream. `cmd | cat` only copies.

## Write a file with a redirect
Also asked as: cat > file; create a file with cat; cat overwrite; here document versus cat; save stdin to a file
The shell opens the redirect before `cat` runs. `cat > file` truncates `file`, then reads stdin and writes it there. `cat >> file` appends. `cat` itself does not have an output flag. A here document is the way to feed a short body without typing on the terminal.

```sh
cat > file <<'EOF'
pattern
EOF
```

`cat file > file` empties the file. The shell truncates it before `cat` reads it. That is the classic data loss. Write to a new name and rename, or use a tool that understands in-place edits. `cat` will not ask before an overwrite. The redirect is the shell's. A permission error on the output is from the shell, and `cat` may not start.

## Number the lines
Also asked as: cat -n; number lines; cat -b; number nonblank lines; nl versus cat
`-n` prints a number before every line, including blank lines. The number starts at 1 for the first line of the whole output, not at 1 for each file. `-b` numbers only non-blank lines and overrides `-n` if both are given. The text of the file is unchanged. The numbers are on standard output with the file.

```sh
cat -n -- file
```

```sh
cat -b -- file
```

Use `-n` when blank lines should count. Use `-b` when they should not. A line is a stretch of bytes ending in a newline. A file with no trailing newline still gets a number on that last partial line under GNU cat. `-n` does not wrap, and it does not know about screen width. `nl` is the separate command when you need a different number format. `-n` is not "print nothing but the count." It is the file, annotated.

## Show ends of lines and tabs
Also asked as: cat -A; cat -E; show line endings; cat -T; see tabs; trailing spaces
`-E` prints `$` at the end of each line, so a trailing space becomes visible before the `$`. `-T` prints tabs as `^I`. `-A` is both, plus `-v` for other non-printing bytes. These marks are added to the output. They are not in the file. A CRLF file shows `^M$` at the end of the line under `-A`.

```sh
cat -A -- file
```

People look at a file that "looks the same" as another and miss a trailing space or a carriage return. `-A` is the quick view. It is a bad thing to pipe into a parser, because the `$` and `^I` are extra bytes. `-E` does not convert line endings. `--` is still required before a name that starts with a dash. macOS cat spells the same ideas `-e` and `-t`, and `-e` there includes non-printing characters, not only the end mark.

## Show non-printing characters
Also asked as: cat -v; show nonprinting; caret notation; binary as text; cat -vET
`-v` prints non-printing bytes in caret notation. A NUL can show as `^@`. Bytes with the high bit set use `M-` notation. Newlines stay newlines, and tabs stay tabs unless you also asked for `-T`. `-A` is `-vET`. The output is a display, not a hex dump.

```sh
cat -v -- file
```

People `cat` a binary file to a terminal and the terminal beeps or changes character set. `-v` is how you look without letting every control byte act. It is not a complete dump. `od -An -tx1 -- file` is the byte view. `-v` does not stop at the first NUL. GNU cat will print the whole file. A terminal can still struggle with a huge result. Redirect it.

## Collapse blank lines
Also asked as: cat -s; squeeze blank lines; suppress repeated empty lines; collapse whitespace lines; single-space a file
`-s` replaces a run of empty lines with one empty line. A line that contains spaces is not empty. The other lines are copied as they are. The flag does not trim trailing spaces, and it does not join wrapped lines. It only squeezes repeated empty output lines.

```sh
cat -s -- file
```

People pass `-s` to mean "ignore blank lines" the way `diff -B` does. `cat` still prints one blank line where the run was. A file that uses a line of spaces as a separator will not shrink. `-s` counts lines after the files are concatenated, so a run that starts at the end of one file and continues at the start of the next becomes one blank line in the output.

## Copy a file
Also asked as: cat source > dest; copy with cat; cat versus cp; duplicate a file; redirect copy
`cat file > other` is a copy only in the weak sense that the bytes land in a new file the shell created. The new file's mode is what the shell's umask produced, not the mode of `file`. Sparseness, owner, timestamps, and hard links are not copied. `cp -- file other` is the command that copies a file.

```sh
cp -- file file
```

Use `cp` to copy. Use `cat` when you want the bytes on standard output or when you are joining streams. `cat file > file` empties the source, as the redirect truncates first. `cp` to the same name fails instead of wiping it. A binary file survives `cat` into a redirect. It does not survive being pasted through a terminal.

## Add a line at the end of a file
Also asked as: append a line; cat >> file; add text to a file; echo versus printf append; do not wipe the file
`>>` appends. `>` truncates. `printf` is the reliable way to write one line, because `echo` flags differ and a value that starts with a dash can be read as an option. The redirect is on `printf`, not a feature of `cat`. `cat` belongs here when the new bytes come from another file.

```sh
printf '%s\n' 'pattern' >> file
```

```sh
cat -- file >> file
```

Use `printf` for a line you typed. Use `cat` to append a file. The second form appends `file` to `other` only if you name two different paths. `cat file >> file` reads and appends at the same time and can grow the file without end. Do not append a file to itself. `>>` creates the destination if it is missing. It does not create parent directories.

## Look at the start of a file without a pager
Also asked as: cat a short file; head versus cat; first lines; do not dump a huge file; cat or head
`cat` reads the whole file. It has no count flag. `head` prints the first lines and stops. `head -n 20 -- file` is the command when you want a prefix. `cat` of a multi-gigabyte file writes all of it to the terminal or the pipe.

```sh
head -n 20 -- file
```

People `cat` a log and then reach for Control-C. The process stops. The terminal is full of the middle of the file. `head` is the short view. `tail` is the end. `cat` is the whole stream, which is what a consumer that must see every byte wants. A pipe does not make `cat` faster to finish. It still reads the input to the end unless the reader exits and the pipe breaks.

## What a failed cat means
Also asked as: cat exit status; cat exit code; cannot open file; cat failed; missing file status
`cat` exits 0 when it read every operand. A missing file is a message on stderr and a non-zero status. GNU cat keeps going with the remaining operands, so you can see a partial concatenation and still get a failure status. A directory operand is a failure for that operand. A permission error is a failure.

```sh
cat -- file
```

People check the output for emptiness and treat it as success. An empty file is success and prints nothing. A missing file prints nothing on stdout and fails. Check the status. `2>/dev/null` hides the reason and does not change the status. `cat` of a file you cannot read is not an empty file. It is an error. In a pipeline the status of `cat` is lost unless `pipefail` is on, because the status you see is the last stage.
