# Contributing

Issues and pull requests are both welcome. An issue is the right place to report a wrong answer, a missing command, or a bug in the app. A pull request is the right place to fix it.

## Code

Open an issue if you are not sure the change is wanted, or if you cannot run the app locally. Open a pull request if you already have the fix.

- Keep the change small and about one thing.
- Run the tests and the typecheck before you push: `npm test` and `npm run typecheck`.
- Do not commit secrets, local databases, or generated `node_modules`.

A pull request should say what changed and how you checked it. A screenshot helps for UI work. An issue should say what you expected and what happened.

## FAQs

The searchable chapters live as Markdown. You can add a command you actually use, or correct one that is already here. Everyone has pet peeves. That is fine. A chapter that answers a real question is more useful than a complete man page.

There is no separate shared FAQ repository yet. Until there is one, contribute the `.md` files in this repo. If a shared repository appears later, these chapters should be easy to move: one command per file, no app code mixed in.

### Chapter shape

The first line is the title: `# name`. A short lesson comes before any `##` heading. It says what the command does, what it does not do, the basic form on its own line starting `Basic form: `, how arguments and quoting work, and the exit status. Name the implementation you assume, and say when macOS or BusyBox differs.

Each section is a question someone would type:

```
## Question the way a person asks it
Also asked as: other wording; another wording; the flag names

Teaching prose. The section stands alone. Restate the flag and why it is there.
Say what the command prints, what people confuse it with, and the gotcha that
makes the obvious answer wrong.

```sh
command -- 'placeholder'
```
```

`Also asked as:` is the search index. Each piece separated by a semicolon is matched on its own, so write a phrase a person would type, not a bag of loose words. The prose and the code fences are not searched.

- Twenty to thirty sections is enough. Cover jobs people search for, not obscure flags.
- Every section has at least one fenced `sh` block. The command is copy-pasteable. Single-quote patterns. Put `--` before a path the reader will replace.
- Prefer the boring correct flag. Show a second fence only for a real second form.
- Every flag you use must exist in the implementation you named. If it is GNU-only, say so. If you are not sure a flag exists, drop the section.
- Do not invent long options. Do not wrap the file in an extra code fence.

### Review

Check the command against the real binary when you can. A wrong exit status or a flag that means something else on macOS is a useful review comment. Style nits are not.

