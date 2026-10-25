# crontab

`crontab` installs and lists the table of jobs the cron daemon will run for you. It does not run a job now, and it does not print output from a job. The daemon reads the table later and runs each due line in a short environment. A line is five time fields and a command. In the old layout the daemon woke every minute and compared the clock to those fields. It still does. Vixie cron, as shipped by cronie on Linux, is assumed here. macOS has `crontab`, and newer macOS prefers launchd. BusyBox `crond` has a thinner `crontab`. Exit status is 0 when the table was read or written. It is non-zero when the file is missing, the user is unknown, or the daemon's spool cannot be updated. A job that fails later does not change that status.

Basic form: crontab -l

`-l` lists your table. `-e` edits it. `-r` removes it. A filename operand installs that file as your table, replacing the old one. There is no `--` that changes this. Quote a command you pass to a shell from cron, inside the table, because a bare `%` is special. The daemon does not expand your login profile. `PATH` in a job is short. The working directory is your home directory.

## List your jobs
Also asked as: crontab -l; show crontab; list cron jobs; what is scheduled; print crontab
`crontab -l` prints your table on stdout. It does not print the system crontab, and it does not print other users. A user with no table gets a message on stderr and a non-zero status. The output is the file you would edit. Comments and blank lines are included. The daemon ignores those.

```sh
crontab -l
```

People run `crontab -l` as root and expect every user's jobs. Root sees root's table. Other users need `-u`, and only root may pass it. People also look in `/etc/crontab` for their lines. That file is the system table, and it has an extra user field. Your table does not. A job that is due does not appear in `crontab -l` as running. The table is the schedule. The process list is the run.

## Edit your jobs
Also asked as: crontab -e; edit crontab; add a cron job; change schedule; open the crontab
`crontab -e` copies your table into a temporary file, opens `$VISUAL` or `$EDITOR`, and installs the result if you save and the editor exits 0. An editor that exits non-zero leaves the old table in place. The first time, the file is empty or a set of comments. You are editing the table, not `/etc/crontab`.

```sh
crontab -e
```

People set `EDITOR` to an editor that forks and returns immediately. `crontab` then installs a half-written or empty file. Use an editor that stays in the foreground. People also paste a command with a `%` in it. In a crontab line, an unescaped `%` ends the command and feeds the rest to the command as stdin. Write `\%` for a literal percent. A bad time field is often installed anyway and then skipped by the daemon, or rejected, depending on the cron. Read the line back with `-l`.

## Install a table from a file
Also asked as: crontab file; load a crontab; crontab path; replace crontab from a file; import jobs
A filename argument replaces your table with that file. `crontab file` is the install. `crontab -l > file` is the backup. There is no merge. The file becomes the whole table. This is the form scripts use. `-` reads the new table from stdin.

```sh
crontab -- file
```

People `crontab -e file` and the editor opens, ignoring the name the way they hoped, or the flag parse fails. The install form has no `-e`. People also install an empty file and wipe the table. Check `crontab -l` after. A file that is not a table still gets installed on many crons, and every line fails later. The status of `crontab` was 0. Keep a copy of the previous `-l` output before you replace it.

## Remove your table
Also asked as: crontab -r; delete crontab; remove all jobs; wipe cron; crontab remove
`crontab -r` deletes your table. There is no trash. Some builds ask if you pass `-i`. A build without `-i` removes it at once. Root with `-u` can remove another user's table. Listing after a remove is a missing-file error.

```sh
crontab -l > file
```

```sh
crontab -r
```

Save the list first. Use the remove only when you mean it. People type `-r` aiming for `-e`, because the keys are adjacent, and the table is gone. The backup file is the way back, with `crontab file`. A remove does not kill a job that already started. It only stops future runs. The daemon notices the missing table on its next scan.

## Edit another user's table
Also asked as: crontab -u; root crontab user; edit a user's cron; crontab for another account; -u user
`-u` names the user. Only root may use it. `crontab -u user -l` lists that user's table. `crontab -u user -e` edits it. The jobs run as that user, with that user's home and that user's short environment. It does not run as root unless the user is root.

```sh
crontab -u user -l
```

People put a username in their own table, copying `/etc/crontab`. In a user table the sixth field is the command. The username is read as part of the command and the job fails. The user field exists in `/etc/crontab` and in `/etc/cron.d/`, not in a table `crontab -e` edits. `-u` is how you pick whose table. A misspelled user is a non-zero status and no edit.

## Write a schedule
Also asked as: cron time fields; five fields; minute hour day; cron expression; what the asterisk means
A job line is minute, hour, day of month, month, day of week, then the command. Ranges are inclusive. `*` is every value. `*/15` is every 15. `1-5` is a range. `1,15` is a list. Day of week is 0-7, and both 0 and 7 are Sunday. A line that is due runs once that minute. It does not run again until the next minute that matches.

```sh
crontab -l
```

People write six time fields, or they put the command where the hour belongs. The command is field six in a user table. People also expect `0 2 * * 1` to mean Monday and also the 1st. If both day of month and day of week are restricted, Vixie cron runs the job when either matches. That is the classic surprise. Use `*` for the one you do not mean to constrain. Test a line against a cron explainer before you depend on it.

## Run something every day, or at reboot
Also asked as: @daily; @reboot; @hourly; @weekly; special cron strings; run once a day
`@daily` is midnight. `@hourly` is on the hour. `@weekly` is Sunday midnight. `@monthly` is the first at midnight. `@reboot` runs once when the daemon starts, which is boot on a normal machine and also a restart of the daemon. These replace the five time fields. The command still follows. They are Vixie extensions. A strict POSIX cron may not have them.

```sh
crontab -e
```

People use `@reboot` for a service and also expect it after a daemon reload only, or not after one. A reload that re-reads tables can run `@reboot` jobs again on some builds, and not on others. Do not use it as a process supervisor. People use `@daily` and assume their timezone. The daemon uses the system's clock and zone. A laptop asleep at midnight runs the job at the next wake only if that cron catches up. Many do not. The missed hour is missed.

## See why a job has no environment
Also asked as: cron PATH; crontab environment; profile not loaded; HOME in cron; why the command is not found
A job does not source `.bashrc` or `.profile`. `PATH` is something short such as `/usr/bin:/bin`. A command that works in your terminal can be "not found" in cron. The working directory is `$HOME`. The shell is `/bin/sh`. Variables you set in the table, above the job, are exported to the job. `PATH=/usr/local/bin:/usr/bin:/bin` is the usual first line.

```sh
crontab -e
```

People rely on an alias or a function. Cron has neither. People also rely on a virtualenv or a version manager that their profile starts. Call the interpreter by its full path, or set `PATH` in the table. `MAILTO` names who gets the output. `SHELL` can replace `/bin/sh`, and a job written for bash then needs that. Percent-escapes and quotes are still the table's rules, not your interactive shell's.

## Capture output, or mail it
Also asked as: cron email; MAILTO; cron output; redirect cron; where does cron print
A job's stdout and stderr are mailed to you, or to `MAILTO`, if there is any output. Empty output sends no mail. `MAILTO=""` disables mail. A redirect inside the command keeps the output in a file instead. `>> file 2>&1` appends both streams. Without a redirect, a chatty job fills a mailbox. The mail is the error report you did not know you had.

```sh
crontab -e
```

People add `> file` and still get mail, because stderr was not redirected. `2>&1` has to follow the stdout redirect. People also truncate the log with `>` every run and lose the failure from last night. Append, or let a log rotator handle it. `MAILTO` is a table variable, not a field on the job line. Set it once at the top. A redirect to a path you cannot write is a mail message about the redirect.

## Escape percent signs
Also asked as: cron percent; \% in crontab; date in cron; stdin from percent; crontab %
In a crontab command, `%` means "end the command, start stdin." The rest of the line is fed to the command as standard input, with `%` turned into newlines. A `date +%F` without an escape becomes `date +` with stdin `F`. Write `\%` for a literal percent. This is the bug behind a large share of "works in the shell" reports.

```sh
crontab -e
```

People copy a `date` format straight from the prompt and the job runs the wrong program with odd stdin. Quote the format and escape the percents: `date '+\%F'`. People also use `%` on purpose to feed a here-document and then add a redirect, and the stdin fight is confusing. Prefer a script file for anything with quotes, percents, or more than one command. The table stays one line per job.

## Run a script, not a long line
Also asked as: cron script; crontab and a shell script; bash in cron; working directory cron; call a file from cron
A script file avoids quoting, percents, and a line you cannot read. Call it by an absolute path. The script's shebang decides the interpreter. Cron's cwd is `$HOME`, so a relative path in the script is relative to home, not to the script's directory. `cd` to the project inside the script, or use absolute paths. Make the script executable, or call `bash /full/path/script`.

```sh
crontab -e
```

People `./script` in the table and the daemon does not find it, because cwd is home and `PATH` does not include `.`. People also edit the script and forget the table still points at the old path. The table holds the command line. It does not watch the file. A script that needs a virtualenv has to activate it itself, or be invoked through that environment's interpreter. Cron will not do it.

## Restrict the day of week and the day of month
Also asked as: cron day of week; day of month OR; Sunday is 0; crontab weekday; both day fields
Day of month and day of week are special. If both are something other than `*`, Vixie cron runs the job when either one matches, not only when both match. `0 0 1 * 1` runs on the 1st and on every Monday. To run only on Mondays, use `0 0 * * 1`. To run only on the 1st, use `0 0 1 * *`. Names like `Mon` work on Vixie cron.

```sh
crontab -e
```

People write both fields, test it once on a Monday the 1st, and ship a job that also runs on other Mondays. The OR rule is the reason. Some other crons differ. The cronie and Vixie behavior is the OR. A range `1-5` is Monday to Friday if Sunday is 0. `7` is also Sunday. A job that must be "Monday and the 1st" needs a test inside the script. The five fields cannot say AND between those two.

## Check that the daemon is running
Also asked as: cron is not running; systemctl cron; crond; jobs never run; is cron enabled
`crontab` only edits the table. The `cron` or `crond` daemon runs the jobs. A stopped daemon means a perfect table and no jobs. On systemd the unit is often `cron` or `crond`. A user crontab is read by that daemon from its spool. Editing the file under `/var/spool/cron` by hand can work and can also be overwritten or ignored. Use `crontab`.

```sh
crontab -l
```

People reload nothing and expect an edit to apply in the same second. The daemon re-reads on its schedule, usually within a minute. People also run jobs on a machine that sleeps. A suspended laptop does not run a 3 a.m. job at 3 a.m. Anacron, or a timer that catches up, is the tool for a machine that is not always on. `crontab` does not install anacron. Check the mail and the log. Silence can mean the daemon never started.

## Read the system crontab
Also asked as: /etc/crontab; /etc/cron.d; system crontab; user field; cron.daily
`/etc/crontab` is not your user table. It has a username field before the command. `/etc/cron.d/` is a directory of the same shape. `/etc/cron.daily/` is a directory of scripts run by a daily job, not a table you edit with `crontab`. Putting a user table line into `/etc/crontab` shifts every field and the command fails, or runs as the wrong user.

```sh
crontab -l
```

People edit `/etc/crontab` to add their job and the extra field eats the command. Use `crontab -e` for your jobs. Use `/etc/cron.d/` for a package, with the user field. A file in `cron.d` that is writable by a non-root user is ignored by cronie. A missing newline at the end of the file can hide the last job. `crontab -l` will not show these. They are system files. The daemon's log shows which it loaded.

## Set the timezone of a job
Also asked as: cron timezone; CRON_TZ; UTC cron; wrong hour; crontab and localtime
The daemon evaluates the time fields in the system's local time, unless `CRON_TZ` is set in that table. `CRON_TZ=UTC` makes that table's fields UTC. It is a Vixie extension. The system zone is what `date` prints. A container with UTC and a host with a local zone run the same fields at different wall times.

```sh
crontab -e
```

People set `TZ` in the table and expect the schedule to move. `TZ` affects the job's view of the clock, not when the daemon decides the job is due. `CRON_TZ` is the schedule knob. A daylight-saving jump can skip a 2:30 a.m. job or run it twice. Cron does not average that away. If a job must run once a day regardless, trigger it from something that catches up, or pick an hour that exists on both sides of the change.

## Log from inside the job
Also asked as: cron log; redirect to a log; date stamp a cron log; see what cron ran; syslog cron
Mail is the default log. A file you append is the durable one. Include a date in the line if the command prints nothing, or you cannot tell runs apart. The daemon may also log starts to syslog or to the journal, without the command's output. That log says the job was attempted. Your redirect says what it printed.

```sh
crontab -e
```

People look at syslog, see a start, and assume the command succeeded. A non-zero exit may be logged and may not. The output file is the record. A redirect that fails because the directory is missing becomes a mail message, if mail works. Create the directory in the script. A shared log needs a lock if two jobs append, or lines interleave. One job, one file, is the simple form.

## Test a job now
Also asked as: run cron job immediately; test crontab line; cron does not run now; trigger a job; debug a cron command
`crontab` has no "run this line." Copy the command into a shell that has the same `PATH` and cwd, or wait for the minute. A practical test is to set the fields to a minute from now and watch the mail or the log. Remember the daemon is on a one-minute tick. A job due at 12:00 runs at 12:00, not at 12:00:30 when you saved the table.

```sh
crontab -l
```

People run the command in their interactive shell, see success, and the cron run fails. The environment was different. Export the short `PATH`, `cd` to `$HOME`, and run under `sh` before blaming the schedule. `@reboot` is a poor test loop. A `*` in the minute field runs every minute, which is a good test and a bad thing to leave in place. Put the real fields back.

## Keep a backup of the table
Also asked as: backup crontab; crontab -l redirect; save cron jobs; version control crontab; restore crontab
`crontab -l` is the whole table. Redirect it to a file and that file is a backup. `crontab file` restores it. There is no built-in history. An editor crash after `-e` can install a partial table if the editor exited 0. A copy from before the edit is the undo. Treat the table like source.

```sh
crontab -l > file
```

People keep the only copy inside the spool and a mistaken `-r` ends it. The redirect is the copy. A backup file can itself be installed by accident if you pass the wrong name. Read it first. Secrets in a crontab, such as a password in a command, are in that backup too. Prefer a script the job calls, with the secret in a file with a tight mode. The table is readable by root and by you.

## What a failed crontab means
Also asked as: crontab exit status; crontab failed; no crontab for user; permission denied crontab; crontab error
`crontab` exits non-zero when there is no table to list, when you may not use `-u`, or when the spool cannot be written. A syntax error in a job is often not a `crontab` failure. The table is installed, and the daemon skips or fails the line later. The message `no crontab for user` is the empty list, not a broken daemon. Installing a file exits 0 if the spool accepted it.

```sh
crontab -l
```

People treat a successful `crontab` as a successful job. The job has not run. Check the next minute, the mail, and the log. A permission error on `-u` means you are not root. A permission error on your own table means the spool or the daemon's directory is wrong, which is an administration problem. Under `set -e`, a `crontab -l` for a user with no table aborts a script. That non-zero is "nothing installed," which may be the answer you wanted to handle, not a crash.
