# Vi Cheatsheet

## Modes

| Key | Mode |
|---|---|
| `i` | Insert before cursor |
| `a` | Insert after cursor |
| `A` | Insert at end of line |
| `o` | Open new line below and insert |
| `O` | Open new line above and insert |
| `Esc` | Back to normal mode |

## Moving Around

| Key | Where |
|---|---|
| `h j k l` | Left, down, up, right |
| `0` | Start of line |
| `^` | First non-blank character of line |
| `$` | End of line |
| `w` | Next word |
| `b` | Previous word |
| `gg` | Top of file |
| `G` | Bottom of file |
| `42G` | Go to line 42 |
| `Ctrl+d` | Half page down |
| `Ctrl+u` | Half page up |

## Deleting

| Key | What |
|---|---|
| `x` | Delete character under cursor |
| `X` | Delete character before cursor |
| `dd` | Delete (cut) whole line |
| `5dd` | Delete 5 lines |
| `dw` | Delete word |
| `d$` or `D` | Delete to end of line |
| `d0` | Delete to start of line |

Everything you delete goes into a buffer — so `dd` is really "cut", not just "delete".

## Copy (Yank) & Paste

This is the part you keep forgetting. Here it is. Burn it in. 🔥

| Key | What |
|---|---|
| `yy` | Yank (copy) current line |
| `5yy` | Yank 5 lines |
| `yw` | Yank word |
| `y$` | Yank to end of line |
| `p` | Paste after cursor/below current line |
| `P` | Paste before cursor/above current line |

So the classic "move a line" workflow is: `dd` (cut), move to where you want it, `p` (paste).

Copy a line: `yy`, move, `p`.

## Visual Mode (Select, Then Act)

If you can never remember the motion combos, visual mode lets you **see what you're selecting** before you act on it:

| Key | What |
|---|---|
| `v` | Start selecting characters |
| `V` | Start selecting whole lines |
| `Ctrl+v` | Block/column select (rectangular!) |

Once you've selected, press:
- `d` — cut selection
- `y` — yank (copy) selection
- `>` / `<` — indent / unindent
- `~` — toggle case

So: `V`, select a few lines with `j`/`k`, `d` to cut, move, `p` to paste. Done.

## Replace

| Key | What |
|---|---|
| `r` | Replace single character |
| `R` | Replace mode (overwrite until Esc) |

## Search & Replace

| Key | What |
|---|---|
| `/pattern` | Search forward |
| `?pattern` | Search backward |
| `n` / `N` | Next / previous match |
| `:s/old/new/` | Replace first on current line |
| `:s/old/new/g` | Replace all on current line |
| `:%s/old/new/g` | Replace all in file |
| `:%s/old/new/gc` | Replace all, ask each time |

## Undo & Redo

| Key | What |
|---|---|
| `u` | Undo |
| `Ctrl+r` | Redo |
| `.` | Repeat last action (surprisingly useful) |

## File Operations

| Command | What |
|---|---|
| `:w` | Save |
| `:q` | Quit (fails if unsaved changes) |
| `:wq` or `:x` | Save and quit |
| `:q!` | Quit without saving (the "I give up" command) |
| `:w filename` | Save as |
| `:e filename` | Open another file |

## Misc Useful Stuff

| Command | What |
|---|---|
| `:set number` | Show line numbers |
| `:set nonumber` | Hide line numbers |
| `:set paste` | Paste mode (preserves formatting when pasting from clipboard) |
| `:set nopaste` | Back to normal |
| `J` | Join current line with next |
| `>>` / `<<` | Indent / unindent current line |

## The Cheat Code

If all else fails: `vimtutor` in your terminal. It's a 30-minute interactive tutorial that comes with vim. Yes, it's been there all along. No, nobody told you either.
