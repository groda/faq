# Prompt: generate one faq chapter

Paste the prompt below into a capable model once per command. Save the reply, with nothing around it, as `share/<command>.md`. Then run `faq --check`. Do not use a model at question time. `faq` only searches this file.

Replace `grep` in the first line with the command you want (`find`, `sed`, `awk`, `xargs`, `tar`, `ssh`, `git`, `jq`, …). Generate one command per run. Do not generate a bundle.

---

Write one FAQ chapter for the Unix command `grep`. The chapter is both a thing a person can read from top to bottom to learn the command, and the search index for a terminal tool. Output only the Markdown file. No preamble, no closing note.

Format, exact:

- Line 1 is `# grep` (the command name, nothing else).
- Then a short lesson, before any `##` heading. Cover what the command actually does, what it does not do, the basic form on its own line starting exactly `Basic form: `, how arguments and quoting work, and the exit status if the command has one. Name the implementation you assume (GNU, on Linux) and say when macOS or busybox differs in general. Plain prose. No table of every flag.
- Then one section per common question. A section is:

```
## Question the way a person asks it

Also asked as: other wording; another wording; the flag names; synonyms

Teaching prose. Each section must stand alone. Do not say "as above" or "the same flags". Restate the flag and why it is there. Two short paragraphs is the usual length. Say what the command prints, what people confuse it with, and the one gotcha that makes the obvious answer wrong.

```sh
command -- 'placeholder'
```

A second fence only when there is a real second form (portable version, or the other flag people mix up). Say when to use which.
```

Rules for the questions:

- Twenty to thirty questions. The head of the distribution: jobs people actually search for. Not a rewritten man page. Not obscure flags.
- The `##` line is a question or a task, not a flag. Good: "Show lines surrounding each match". Bad: "-C".
- The `Also asked as:` line is the search index. Each semicolon-separated piece is its own key, matched on its own, so write each piece as a question or a short phrase a person would actually type. Not a bag of loose words. Include the flag letters as their own phrase (`grep -C`). The prose and the code are not searched. If the words are not in the heading or in one of these phrases, the search will not find the section.
- `Also asked as:` is the first line of the section, immediately under the heading.
- Every section has at least one fenced `sh` block. The command is copy-pasteable. Single-quote patterns. Put `--` before a pattern that a user will replace. Use obvious placeholders (`file`, `pattern`, `.`).
- Prefer the correct boring flag over a clever pipeline. Show the pipeline when it is the actual answer (an AND of two patterns, a portable `find`).
- Every flag you use must exist in the implementation you named. If a flag is GNU-only, say so in that section. If you are not sure a flag exists, drop the section. Do not invent long options.
- Call out behavior that surprises people: basic versus extended regular expressions, exit codes, directories, stdin when no file is given, binary files, shells eating quotes and braces, word characters, counts of lines versus counts of matches.
- Do not mention this prompt, fuzzy search, or language models. The chapter is a FAQ.
- Do not wrap the file in a code fence. The first character of the reply is `#`.
