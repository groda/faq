## Change the prompt
Also asked as: PS1; change bash prompt; customize prompt; export PS1; prompt with user and directory; bash prompt
`PS1` is the interactive prompt. A non-interactive script does not print it. Set it in `~/.bashrc`, which interactive bash reads. Single quotes keep the special escapes for later. `\u` is the user. `\h` is the hostname. `\w` is the working directory. `\$` is `$` for a user and `#` for root. `\n` is a newline. Double quotes expand variables once, when you set it, which is usually too early.

```sh
PS1='\u@\h:\w\$ '
```

People set `PS1` in `~/.bash_profile` and a new terminal is unchanged. Interactive non-login shells read `~/.bashrc`. People also put a command substitution in double quotes and the output is frozen at assignment time. Use single quotes, or `PROMPT_COMMAND`, which runs before each prompt. Color codes that are not wrapped in `\[ \]` make the cursor lie about the line length. A prompt does not change `set -e` or the status of the last command. `$?` in the prompt is the status of the previous command, which is what people want when they embed it.
