# VIM Cheat Sheets

## Cursor Movement

In `NORMAL` mode.

| Keys | Description |
| ---- | ----------- |
| ++h++ | Move cursor to the left |
| ++j++ | Move cursor down |
| ++k++ | Move cursor up |
| ++l++ | Move cursor to the right |
| |
| `n` ++h++ | Move cursor `n` steps to the left |
| `n` ++j++ | Move cursor `n` steps to the right |
| `n` ++k++ | Move cursor `n` steps down |
| `n` ++l++ | Move cursor `n` steps up |
| |
| ++g++ ++g++ | Go to the first line |
| ++shift++ + ++g++ | Go to the last line |
| |
| ++brace-right++ | Jump to next paragraph |
| ++brace-left++ | Jump to previous paragraph |
| ++colon++ `n` | Go to line number `n` |

## Edit

In `NORMAL` mode.

| Commands | Description |
| -------- | ----------- |
| `:e` | Edit file (_i.e., open_) |
| `:w` | Write file (_i.e., save_) |
| `:q` | Quit file (_only works if file is unchanged_) |
| `:q!` | Force quit (_even if file is changed_) |
| `:wq` | Write file and quit |

| Keys | Description |
| -------- | ----------- |
| ++y++ ++y++ | Yank (_i.e., copy_) a line |
| ++d++ ++d++ | Delete (_i.e., copy_) a line |
| ++p++ | Paste after cursor |
| ++shift++ + ++p++ | Paste before cursor |
| ++u++ | Undo |
| ++ctr++ + ++r++ | Redo |
| ++equal++ | Auto-indent current line |
| |
| `n` ++y++ ++y++ | Yank (_i.e., copy_) `n` lines |
| `n` ++d++ ++d++ | Delete (_i.e., copy_) `n` lines |
| `n` ++equal++ ++equal++ | Auto-indent `n` lines |
| ++g++ ++g++ ++equal++ &nbsp;&nbsp;&nbsp; ++shift++ + ++g++ | Auto-indent all lines |

The last keys can be split into 3 components:

- ++g++ ++g++: Go to the first line
- ++equal++: Auto-indent
- ++shift++ + ++g++: Go to the last line

This reads as (i) go to the first line, then (ii) auto-indent (iii) until the last line.

## Search and Replace

In `NORMAL` mode.

| Commands | Description |
| -------- | ----------- |
| `/<pattern>` | Search for `<pattern>` |
| `?<pattern>` | Search backward for `<pattern>` |
| `:%s/<old>/<new>/g` | Replace all `<old>` with `<new>` |

Once a search is being performed.

| Keys | Description |
| ---- | ----------- |
| ++n++ | Continue search forward |
| ++shift++ + ++n++ | Continue search backward |

## Split Screen

## Java

