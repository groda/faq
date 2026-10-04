# faq

Years ago *UNIX for the Impatient*, by Paul W. Abrahams and Bruce R. Larson, lived on my desk. I was a beginner, and it was the book I reached for when I did not want to fight the manual.

A lot has happened since. We all went through the "just Google it, there will be a Stack Overflow answer" years. Now the help sits in the editor: copilots, local models, and the rest of that sorcery. Useful, and also a strange amount of machinery when the question is "which `grep` flag prints the lines around a match?"

`--help` and `man` disappointed me every time I opened them. I kept going back anyway, hoping that this time they would be faster than googling Stack Overflow. They never were. So I made `faq`.

There are already cheatsheet tools. [tldr](https://tldr.sh/) is the short community page. [navi](https://github.com/denisidoro/navi) is the interactive cheatsheet navigator. [cheat](https://github.com/cheat/cheat) is a personal snippet file. They are good at one-liners. This is a book you can search.

## The screen

`faq grep` takes the whole terminal. The questions are a list on the left. The article you are on stays open on the right. The prompt is `ask>`.

You do not submit a search. You type, and the list narrows to the questions that fit. Up and down move through what is left, and the article follows. Esc closes the book and puts you back at the shell. Words after the command, as in `faq grep lines around a match`, start you already narrowed.

The match is against the question and the other ways people ask it, not against every word of the lesson. So "lines around a match", "context", and "grep -C" land on the same page.

If [glow](https://github.com/charmbracelet/glow) is installed, the article on the right is rendered markdown, and the colors follow a light or dark terminal. `FAQ_GLOW_STYLE=light` or `FAQ_GLOW_STYLE=dark` forces one of them. Without glow, the same page is plain text.

## What a chapter is

- One chapter per command, written as the questions people actually ask.
- Each answer is short enough to learn from: what is going on, the command, and the catch.
- An "Also asked as" line on every question, so the five ways of asking it all find the same page.
- Nothing is fetched and no model runs when you ask. The pages were written ahead of time. Search is [fzf](https://github.com/junegunn/fzf).

The chapters are ordinary markdown. Add a command by adding a file. Change a recipe by editing it. Your copy wins, in this order:

1. `$FAQ_HOME`
2. `$XDG_DATA_HOME/faq`, or `~/.local/share/faq`
3. `share/` next to the `faq` script

## Install

You need Python 3 and `fzf`. `glow` is optional.

```sh
install -m 755 faq ~/.local/bin/faq
mkdir -p ~/.local/share/faq
cp share/*.md ~/.local/share/faq/
```

`~/.local/bin` has to be on your `PATH`. `faq --check` should then print one line per chapter.

## Commands

```text
faq grep                         open the grep chapter
faq grep lines around a match    open it already narrowed
faq --read grep                  the whole chapter, in $PAGER
faq --list                       installed chapters
faq --check                      look for a broken chapter
```

Inside the screen: type to narrow, Up and Down to move, Esc to leave.

## A new command

In addition I provide [PROMPT.md](PROMPT.md). Paste it into any capable model, once per command, and save the reply as `share/find.md`, `share/sed.md`, and so on. Then run `faq --check`. The prompt is how a new chapter gets written. It is not part of asking a question. `faq` never calls a model. It only searches the pages you already have.

`grep` is the chapter that ships with this repository. The same prompt is how the next ones get made.

## Credit

I designed it and decided what belonged on each page. I had help from an AI to build it. The idea, and the wish for something that would sit where that old book sat, were mine.
```
