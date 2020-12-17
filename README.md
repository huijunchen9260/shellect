# shellect

shellect is a selection system written in POSIX shell.

## Table of content


<!-- vim-markdown-toc GFM -->

* [Preview](#preview)
* [Introduction](#introduction)
* [Limitation](#limitation)

<!-- vim-markdown-toc -->

## Preview

[![shellect](https://asciinema.org/a/Cmfl0fJzjY4x3gRGvj5muRgUB.png)](https://asciinema.org/a/Cmfl0fJzjY4x3gRGvj5muRgUB)

## Introduction

shellect born from my experience in developing my bibliography manager, [`shbib`](https://github.com/huijunchen9260/shbib), and I built [`shbib`](https://github.com/huijunchen9260/shbib) on the basis provided by [`shfm`](https://github.com/dylanaraps/shfm). I realized that if I do not obey the Unix philosophy and keep adding functions to [`shbib`](https://github.com/huijunchen9260/shbib), [`shbib`](https://github.com/huijunchen9260/shbib) would grow exponentially and eventually become a pain to maintain. Therefore, I isolate out shellect as an individual selection system that just written in POSIX shell.

shellect will either accept standard input or assign the display content by `-t` option, i.e., to display all the non-hidden files and directories in your `$HOME` directory,

```sh
printf '%s\n' $HOME/* | shellect # standard input
shellect -t "$HOME/*"	       # -t option

```

The keybindings are:

```
k/↑/Ctrl-p - up
j/↓/Ctrl-n - down
l/→ - right
h/← - left
Ctrl-f/PageDown - PageDown
Ctrl-u/PageUp - PageUp
g/Home/Ctrl-a - go to top
G/End/Ctrl-e - go to bottom
/ - search
? - show keybinds
q - quit
```

Command-line option:

```
Usage:

shellect [OPTIONS] ([ARGS])

  -h,			Show help options
  -i,			Set case-insensitive search
  -n=[num],		Set numbers of line per entry
  -d=[IFS],		Set IFS
  -c=[content],		Set content to display
  -f=[format],		Set the format to print out content
  -t=[msg],		Set top status bar message
  -b=[msg],		Set bottom status bar message

format detail:
  nldel			delete last nl, equiv to "${1%$nl}"
  basename		only print basename, equiv to "${1##*/}" ;;

  if unset or empty, then equiv to "$1"
```

## Limitation

The efficiency of shellect is highly constraint by the total number of entries and the content that you want to display.

With `bash`, as I tested, probably only numbers of 5000 is large enough to create significant lag. The command I run is `tree /directory/have/5000/subitems | shellect` or `echo $(seq 1 5000) | shellect`.

With `dash`, the efficiency is highly depends on both directions. At the number 20000, shellect runs fair efficiency. The command is `tree /directory/have/20000/subitems | shellect`. With the number of 30000, the pointer will not stop if I relieve my key press. However, changing the command to `echo $(seq 1 30000) | shellect`, in my computer, shellect runs with fair efficiency.

Comparing with `dmenu` and `fzf`, shellect is probably extremely inefficient in terms of large numbers of entry. This is probably the limitation of an interpreting language compared to compiling language.

If there's any method to improve the efficiency of shellect, feel free to open a issue or pull request. I'll be more than happy to work with you.
