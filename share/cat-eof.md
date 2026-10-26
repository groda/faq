# cat << EOF

A here document feeds a command several lines of input from inside the script. The shell reads until it sees a line that is only the marker, and that text becomes the command's standard input. `cat <<EOF` is the form people paste. It does not edit a file by itself, and it is not a string variable until you capture it. Bash on Linux is assumed here. The same syntax works in dash, in BusyBox ash, and in macOS bash. zsh accepts it too. A POSIX `sh` has here documents. Exit status is the status of the command on the left of `<<`, not of the document. `cat` exits 0. A missing command exits non-zero, and the document is still read.

Basic form: cat <<EOF

The marker sits against `<<`. A space is allowed. The closing marker is a line by itself, with no leading spaces and no trailing spaces, unless you use `<<-` and indent with tabs. Leave the marker unquoted when the body should expand variables. That is the normal form. Quote it, `<<'EOF'`, only when the body must be literal. The body is not a command. It is input.

## Feed several lines to a command
Also asked as: cat <<EOF; how does cat << EOF work; here document; heredoc; multiline input; bash <<
`<<EOF` tells the shell to read the following lines until a line that is exactly `EOF`, and to give those lines to the command as stdin. `cat` prints them. Any command that reads stdin can sit there instead. The marker is not special to `cat`. It is shell syntax. The command runs with that stream attached.

```sh
cat <<EOF
pattern
pattern
EOF
```

People think `EOF` is a flag of `cat`. It is the end marker, and you can use any word. `END`, `SQL`, `PY`, and 🐸 are fine. The closer must be alone on its line. A space before it, or a space after it, means it is not the closer, and the shell keeps reading. That is the usual "my script hung" bug. In an unquoted here document, `$` and backticks still expand. Other quotes in the body are just characters.

## Keep the text literal
Also asked as: <<'EOF'; quoted heredoc; no expansion; literal here document; single quote the marker
Quote the marker and the body is literal. Variables are not expanded. Command substitutions are not run. Backticks are characters. The quotes are not part of the marker. The closer is still a bare `EOF`. Use this only when the body contains a `$` you do not want expanded. The everyday form leaves it unquoted.

```sh
cat <<EOF
pattern
EOF
```

People quote the closer, or they quote each line, and the closer never matches. Only the opener is quoted, and only if you need a literal body. People also leave the opener unquoted and wonder why `$HOME` became a path. Unquoted means expand. That is what the form above does. A backslash in an unquoted body can escape `$`, a backtick, or a newline.

## Expand variables in the body
Also asked as: unquoted heredoc; variable in heredoc; expand in <<EOF; parameter expansion; bash here document variable
An unquoted marker expands `$var`, `${var}`, and `` `command` `` in the body before the command sees it. The expansion happens in the current shell, not in a subshell, for a plain `<<`. Quotes inside the body do not protect a `$`. There are no shell quotes in the body. It is not parsed as code.

```sh
cat <<EOF
pattern $HOME
EOF
```

People wrap a variable in double quotes inside the body and still see it expand, which is what they wanted, and then they try single quotes and it still expands. The quotes are characters. The marker decides. People also put a space or a glob in a variable and expect word splitting. Here-document expansion does not split words and does not glob. The value is inserted as text. A `*` in a variable stays a star.

## Indent the closer
Also asked as: <<-EOF; dash heredoc; indented here document; tabs before EOF; leading tabs
`<<-EOF` strips leading tabs from every line of the body, including the closer. Spaces are not stripped. Only tabs. This is how a here document sits indented inside a function without the closer failing. The marker may still be quoted. The unquoted form expands variables and strips tabs.

```sh
cat <<-EOF
	pattern
	EOF
```

People indent with spaces and the closer is not found. The shell reads to the end of the file. Use a real tab, or do not indent the closer. Editors that expand tabs to spaces break `<<-`. People also expect the indentation of the content to disappear entirely. Only leading tabs go. Spaces you typed stay. A mix of tab and space leaves the spaces.

## Run a command with the text as stdin
Also asked as: command <<EOF; heredoc not just cat; mysql <<EOF; python <<EOF; feed a program
`cat` is only the demonstration. The command on the left reads the document. `python -`, `mysql`, `ssh host bash`, and `tee file` are common. The document replaces stdin. A command that does not read stdin ignores it. A command that also needs a terminal, such as a password prompt, does not get your keyboard. Stdin is the document.

```sh
command -- file <<EOF
pattern
EOF
```

People put arguments after the closer. The closer is not an argument. Arguments belong on the command. `command <<EOF arg` is legal, and `arg` is an argument. The document still follows. People also pipe into a command that has a here document. Both want stdin. The here document wins, and the pipe is unused, or the shell rejects the mix. Pick one source of stdin.

## Write a file from a script
Also asked as: cat <<EOF > file; heredoc to a file; write a config; generate a file; tee a heredoc
Redirect the command. `cat <<EOF > file` writes the body to the file, truncating it. `>>` appends. `tee` does the same and can be `sudo tee` when the file is not writable by you. The redirect is the shell's. It is not part of the marker. An unquoted marker expands variables into the file.

```sh
cat <<EOF > file
pattern
EOF
```

People `> file` and the file is empty, because the closer never came and the redirect already truncated it. The truncate happens when the command starts. Fix the closer, then run again. People also expect `cat <<EOF > file` to keep a backup. It does not. `>` replaces the file. A `sudo cat > file` fails for the same reason `sudo` and `>` always fail. The shell opens the file as you. `sudo tee file` is the form that opens it as root.

## Catch the text in a variable
Also asked as: variable=$(cat <<EOF); heredoc to a variable; read into a string; bash multiline variable; command substitution heredoc
Command substitution captures the document. `var=$(cat <<EOF` ... `EOF` `)` sets `var` to the body, minus the trailing newlines that `$()` strips. Quote `"$var"` when you use it. An unquoted marker expands variables at capture time, not when you later print `var`. This is a string, not a file.

```sh
var=$(cat <<EOF
pattern
EOF
)
```

People are surprised the last newline disappeared. `$()` deletes trailing newlines. `printf '%s\n' "$var"` puts one back. People also put the closer and the `)` on the wrong lines. The closer is a line by itself. The `)` closes the substitution after that. Quote the marker only if you wanted the dollars captured as dollars.

## Use a marker that appears in the text
Also asked as: EOF in the body; choose a marker; heredoc delimiter; END marker; collision with closer
The closer is whatever word you used. If the body contains a line that is exactly that word, the document ends there. Pick a marker that is not in the body. `EOF` is a habit, not a requirement. `PY`, `SQL`, `CUT`, and 🐸 are fine. There is no escape for a closer line. Change the marker.

```sh
cat <<🐸
pattern
🐸
```

People indent the accidental closer and, without `<<-`, it is not a closer, so the document runs on. With `<<-`, a tab-indented marker is a closer. A marker with spaces in it needs quotes on the opener. The closer line has no quotes. A marker that includes a `$` is a bad idea. Use a plain word, or a frog.

## Put the document in a function or a test
Also asked as: heredoc in a function; heredoc in if; bash function multiline; indent script; here document scope
A here document is syntax at the command it follows. Inside a function it still feeds that command. It does not create a local by itself. Variables expand from the function's scope when the marker is unquoted. The closer must still be a line the shell sees as the closer, which is why indentation and `<<-` matter in a nested block.

```sh
func() {
	cat <<-EOF
	pattern
	EOF
}
```

People paste a here document into an `if` and the `fi` is eaten as body, because the closer was indented with spaces. The shell is still reading the document. Move the closer to column zero, or use tabs and `<<-`. A here document in a pipeline runs in a subshell in Bash, so a variable set in that pipeline is not visible after it. The document itself is not the subshell. The pipeline is.

## Pass a script to another shell or to ssh
Also asked as: ssh <<EOF; bash <<EOF; remote heredoc; sudo bash heredoc; python <<EOF
`ssh host bash <<EOF` runs a script on the remote side. An unquoted marker expands `$` locally before the text is sent. That is useful when you want a local value inserted. `bash <<EOF` is the same idea locally. `python - <<EOF` runs Python on the body. The remote or the interpreter sees the text, not the marker.

```sh
bash <<EOF
pattern
EOF
```

People leave a `$HOME` in the body and it becomes the local home before `ssh` runs. That is the unquoted rule. Quote the marker if the remote shell should see the dollar. People also `ssh host 'bash -s' <<EOF` and mix quotes until the remote shell gets the document as arguments. The document is stdin. `bash -s` reads commands from stdin. That pairing works. A `ssh` that asks for a password needs a terminal, and stdin is the document. Set keys up first, or allocate a tty on purpose and accept the fight for stdin.

## What a failed here document means
Also asked as: heredoc hung; unexpected end of file; closer not found; bash syntax error near unexpected eof; here document failed
If the closer never appears, Bash reads to the end and reports a syntax error, often `unexpected EOF` or `here-document delimited by end-of-file`. The command may already have started, and a `>` redirect may already have truncated the file. A closer with a trailing space is not the closer. A closer indented with spaces is not the closer unless you used `<<-` and tabs. The status is the shell's syntax error, non-zero, before a normal run.

```sh
cat <<EOF
pattern
EOF
```

People kill a hung script and the hang was the shell waiting for `EOF` on the terminal, because the script was being typed or the closer was missing. Type the closer or Control-C. A command that fails, such as `false <<EOF`, still consumes the document. The status is the command's status. `set -e` aborts on that status. It does not abort on a here document that merely expanded to empty. Check the closer first when the error mentions end of file. The marker in the message is the one Bash was waiting for.
