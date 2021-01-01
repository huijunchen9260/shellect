# shellect

shellect is a selection system written in POSIX shell.

## Table of content


<!-- vim-markdown-toc GFM -->

* [Preview](#preview)
* [Introduction](#introduction)
* [Implementation Details](#implementation-details)
	* [Explanation for TUI manipulation](#explanation-for-tui-manipulation)
	* [Overcome the limitation of POSIX shell](#overcome-the-limitation-of-posix-shell)

<!-- vim-markdown-toc -->

## Preview

[![shellect](https://asciinema.org/a/jLJay0bFv0mqSfcnWbAWYiVwu.png)](https://asciinema.org/a/jLJay0bFv0mqSfcnWbAWYiVwu)

## Introduction

shellect born from my experience in developing my bibliography manager, [`shbib`](https://github.com/huijunchen9260/shbib), and I built [`shbib`](https://github.com/huijunchen9260/shbib) on the basis provided by [`shfm`](https://github.com/dylanaraps/shfm). I realized that if I do not obey the Unix philosophy and keep adding functions to [`shbib`](https://github.com/huijunchen9260/shbib), [`shbib`](https://github.com/huijunchen9260/shbib) would grow exponentially and eventually become a pain to maintain. Therefore, I isolate out shellect as an individual selection system that just written in POSIX shell.

shellect will either accept standard input or assign the display content by `-c` option, i.e., to display all the non-hidden files and directories in your `$HOME` directory,

```sh
printf '%s\n' $HOME/* | shellect # standard input
shellect -c "$HOME/*"	       # -c option

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
  -l,			Set live-search
  -n=[num],		Set numbers of line per entry
  -d=[delim],		Set delimiter (IFS, internal field separator)
  -c=[content],		Set content to display
  -f=[format],		Set the format to print out content
  -t=[msg],		Set top status bar message
  -b=[msg],		Set bottom status bar message

format detail:
  nldel			delete last nl, equiv to "\${1%\$nl}"
  basename		only print basename, equiv to "\${1##*/}" ;;

  if unset or empty, then equiv to "\$1"

live-search detail:
  Enter 		confirm
  Backspace 		delete previous character
  Tab 			Tab-completion forward
  Shift-Tab		Tab-completion backward
  control char		ignore
  others		print out
```

## Implementation Details

### Explanation for TUI manipulation

Basically, printing out the raw escape sequence to manipulate the terminal output works in most terminal, and its function is way richer than `tput`.
However, printing out these escape sequence can be daunting, and it is often time-consuming to remember the function of each arbitrary sequence.
Thus, The following `esc` function is steal from [`shfm`](https://github.com/dylanaraps/shfm), with some of my own comment and modification to facilitate the understanding.
All of the resource can be found in the following three resources:

- Reference for color: https://en.wikipedia.org/wiki/ANSI_escape_code#CSI_sequences
- Reference for vt100: https://vt100.net/docs/vt510-rm/contents.html
- Reference for escape sequence: https://github.com/dylanaraps/pure-sh-bible#escape-sequences

```sh
esc() {
    case $1 in
        # vt100 (IL is vt102) (DECTCEM is vt520)
	CUP)     printf '%s[%s;%sH' "$esc_c" "$2" "$3" ;; # cursor to LINES($2), COLUMNS($3)
        CUU)     printf '%s[%sA'    "$esc_c" "$2"      ;; # cursor up
        CUD)     printf '%s[%sB'    "$esc_c" "$2"      ;; # cursor down
        CUR)     printf '%s[%sC'    "$esc_c" "$2"      ;; # cursor right
	CUL)     printf '%s[%sD'    "$esc_c" "$2"      ;; # cursor left
	DECAWM)  printf '%s[?7%s'   "$esc_c" "$2"      ;; # (h: set; l: unset) line wrap
        DECRC)   printf '%s8'       "$esc_c"           ;; # cursor restore
        DECSC)   printf '%s7'       "$esc_c"           ;; # cursor save
        DECSTBM) printf '%s[%s;%sr' "$esc_c" "$2" "$3" ;; # scroll region ($2: top; $3: bottom)
        DECSLRM) printf '%s[%s;%ss' "$esc_c" "$2" "$3" ;; # Set left and right margin
	DECTCEM) printf '%s[?25%s'  "$esc_c" "$2"      ;; # (h: show; l: hide) cursor visible
	ED[0-2]) printf '%s[%sJ'    "$esc_c" "${1#ED}" ;; # clear screen
	    # Erase Display:
	    # 0: From the cursor through the end of the display
	    # 1: From the beginning of the display through the cursor
	    # 2: The complete display
        EL[0-2]) printf '%s[%sK'    "$esc_c" "${1#EL}" ;; # clear line
	    # Erase Line:
	    # 0: from cursor to end of the line
	    # 1: from beginning of the line to cursor
	    # 2: entire line
        IL)      printf '%s[%sL'    "$esc_c" "$2"      ;; # insert blank line
	SGR)     printf '%s[%s;%sm' "$esc_c" "$2" "$3" ;; # colors ($2); attribute ($3)
	    # Color list:
	    # 			FG	BG
	    # Black		30	40
	    # Red		31	41
	    # Green		32	42
	    # Yellow		33	43
	    # Blue		34	44
	    # Magenta		35	45
	    # Cyan		36	46
	    # White		37	47
	    # Bright Black 	90	100
	    # Bright Red	91	101
	    # Bright Green	92	102
	    # Bright Yellow	93	103
	    # Bright Blue	94	104
	    # Bright Magenta	95	105
	    # Bright Cyan	96	106
	    # Bright White	97	107

	    # Attribute list:
	    # Reset					0/''
	    # Bold					1
	    # Faint					2
	    # Italic					3
	    # Underline					4
	    # Slow blink				5
	    # Swap foreground and background colors.	7
	    # Hidden					8
	    # Strike-through				9

        # xterm (since 1988, supported widely)
	screen_alt) printf '%s[?1049%s' "$esc_c" "$2" ;; # (h: to; l: back from) alternate buffer
    esac
}
```

### Overcome the limitation of POSIX shell

POSIX shell is very limited, and quite inefficient compared to compiling language.
Previously, I experimented the efficiency of POSIX shell in terms of passing through all the argument array elements into the key detecting part:

> The efficiency of shellect is highly constraint by the total number of entries and the content that you want to display.
> With `bash`, as I tested, probably only numbers of 5000 is large enough to create significant lag. The command I run is `tree /directory/have/5000/subitems | shellect` or `echo $(seq 1 5000) | shellect`.
> With `dash`, the efficiency is highly depends on both directions. At the number 20000, shellect runs fair efficiency. The command is `tree /directory/have/20000/subitems | shellect`. With the number of 30000, the pointer will not stop if I relieve my key press. However, changing the command to `echo $(seq 1 30000) | shellect`, in my computer, shellect runs with fair efficiency.
> Comparing with `dmenu` and `fzf`, shellect is probably extremely inefficient in terms of large numbers of entry. This is probably the limitation of an interpreting language compared to compiling language.

As an interpreting language, I found a way to avoid such inefficiency.

First, I'll define some terminologies that I'll use through the explanation:

1. argument array: POSIX shell has no array type. However, there's actually one, and only one array in POSIX shell, i.e., the positional parameters, `$1`, `$2`, etc.
To see more information, go to "Working with arrays" section in [Rich’s sh (POSIX shell) tricks](http://www.etalabs.net/sh_tricks.html).
2. selection: the item in argument array that is defined in `$cur`.
3. Length of array: access by `$#`. The length of the total content is `$last`, and the length of a list is `$len`. `$len` is set to 500 if the length of the total content, `$last`, is larger than 500.


```sh
    while key=$(dd ibs=1 count=1 2>/dev/null); do
    ...
    done
```

This `while` loop is the part to detect key press.
This `dd` command has nothing to interact with the argument array.
However, `dd`'s efficiency will be highly affected by argument array that just pass through it.
If the total number of argument array is too high, then `dd` will become laggy when reading the key press, eventually causing the cursor movement is laggy.

To resolve this limitation. I developed a technique to only feed part of the total content to the above while loop:

```sh
key() {
    input_assign ...  # Generate a list which is part of the total content

    set -- $list # let the list to be argument array

    while key=$(dd ibs=1 count=1 2>/dev/null); do
    ...
	othercommand
	return 0 # Go back to main function and stay in the while loop in main function
    ...
	selection key pressed
	return 1 # Go back to main and leave the while loop in main function
    done
}

main() {
    ...
    while [$? -eq 0]; do # If return 0, stay in while loop; others, leave the while loop
	set -- $content	# total content
	key "$@"
    done
}
```

The following steps are to actively switch between `main` function and `key` function.
Within the `$list`, the selection stay in the `while key` loop. If the current selection ever go out of the `$list`, then go back to `main` function, reload the `$content` and generate new `$list`, back to `while key` loop, and process again.
That is to say, user will experience an one-time inaction when reach the boundary of `$list`.
This inaction is to renew `$list` to match current position in the whole `$content`.
Press again, and the selection will move to the next item.
