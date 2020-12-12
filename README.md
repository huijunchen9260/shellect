# shmenu

shmenu is a menu written in POSIX shell.

## Table of content


<!-- vim-markdown-toc GFM -->

* [Preview](#preview)
* [Introduction](#introduction)

<!-- vim-markdown-toc -->

## Preview

[![shmenu](https://asciinema.org/a/IsWFVG2kVyQX1i0AuOlRVpvnv.png)](https://asciinema.org/a/IsWFVG2kVyQX1i0AuOlRVpvnv)

## Introduction

shmenu born from my experience in developing my bibliography manager, [`shbib`](https://github.com/huijunchen9260/shbib), and I built [`shbib`](https://github.com/huijunchen9260/shbib) on the basis provided by [`shfm`](https://github.com/dylanaraps/shfm). I realized that if I do not obey the Unix philosophy and keep adding functions to [`shbib`](https://github.com/huijunchen9260/shbib), [`shbib`](https://github.com/huijunchen9260/shbib) would grow exponentially and eventually become a pain to maintain. Therefore, I isolate out shmenu as an individual menu system that just written in POSIX shell.

shmenu will either accept standard input or assign the display content by `-t` option, i.e., to display all the non-hidden files and directories in your `$HOME` directory,

```sh
printf '%s\n' $HOME/* | shmenu # standard input
shmenu -t "$HOME/*"	       # -t option

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

shmenu [OPTIONS] ([ARGS])

  -h,			Show help options
  -n=[num]		Set numbers of line per entry
  -s=[IFS]		Set IFS
  -t=[text]		Set text to display
  -f=[format]		Set the format to print out text

format detail:
  nldel			delete last nl, equiv to "\${1%\$nl}"
  basename		only print basename, equiv to "\${1##*/}" ;;

  if unset or empty, then equiv to "\$1"
```






