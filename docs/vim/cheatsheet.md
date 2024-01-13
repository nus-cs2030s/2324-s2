# VIM Cheat Sheets

## Cursor Movement

In `NORMAL` mode.

| Keys | Description |
| ---- | ----------- |
| ++h++ &nbsp;&nbsp; (_or_ ++left++) | Move cursor to the left (_you may use arrow keys if_ `.vimrc` _is set correctly_) |
| ++j++ &nbsp;&nbsp; (_or_ ++down++) | Move cursor down (_you may use arrow keys if_ `.vimrc` _is set correctly_) |
| ++k++ &nbsp;&nbsp; (_or_ ++up++) | Move cursor up (_you may use arrow keys if_ `.vimrc` _is set correctly_) |
| ++l++ &nbsp;&nbsp; (_or_ ++right++) | Move cursor to the right (_you may use arrow keys if_ `.vimrc` _is set correctly_) |
| |
| `n` ++h++ &nbsp;&nbsp; (_or_ `n` ++left++) | Move cursor `n` steps to the left (_you may use arrow keys if_ `.vimrc` _is set correctly_) |
| `n` ++j++ &nbsp;&nbsp; (_or_ `n` ++down++) | Move cursor `n` steps down (_you may use arrow keys if_ `.vimrc` _is set correctly_) |
| `n` ++k++ &nbsp;&nbsp; (_or_ `n` ++up++) | Move cursor `n` steps up (_you may use arrow keys if_ `.vimrc` _is set correctly_) |
| `n` ++l++ &nbsp;&nbsp; (_or_ `n` ++right++) | Move cursor `n` steps to the right (_you may use arrow keys if_ `.vimrc` _is set correctly_) |
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
| ++ctrl++ + ++r++ | Redo |
| ++equal++ | Auto-indent current line |
| |
| `n` ++y++ ++y++ | Yank (_i.e., copy_) `n` lines |
| `n` ++d++ ++d++ | Delete (_i.e., copy_) `n` lines |
| `n` ++equal++ ++equal++ | Auto-indent `n` lines |
| ++g++ ++g++ ++equal++ &nbsp;&nbsp;&nbsp; ++shift++ + ++g++ | Auto-indent all lines |

The last keys can be split into 3 components:

1. ++g++ ++g++: Go to the first line
2. ++equal++: Auto-indent
3. ++shift++ + ++g++: Go to the last line

This reads as (1.) go to the first line, then (2.) auto-indent (3.) until the last line.

## Search and Replace

In `NORMAL` mode.

| Commands | Description |
| -------- | ----------- |
| `/<pattern>` | Search for `<pattern>` |
| `?<pattern>` | Search backward for `<pattern>` |
| `:%s/<old>/<new>/gc` | Replace all `<old>` with `<new>` (_this will prompt options_) |
| `:<sn>,<en>s/<old>/<new>/gc` | Replace all `<old>` with `<new>` in the given line range from `<sn>` to `<en>` (_this will prompt options_) |

Once a search is performed.

| Keys | Description |
| ---- | ----------- |
| ++n++ | Continue search forward |
| ++shift++ + ++n++ | Continue search backward |

Options for replace

| Option | Description |
| ------ | ----------- |
| `y` | Yes, replace the current one |
| `n` | No, skip this one |
| `a` | Yes to all |
| `l` | Yes to just this one |
| `q` | Quit |

### Comment and Uncomment

We can quickly comment on a specific range of lines using the replace functionality.  The idea is to replace the _beginning of a line_ with `//` (_i.e._, to start a single-line comment).  We need to "escape" the `/` character by writing it as `\/\/`To match the beginning of the line, the character is `^`.

Comment out Lines 4 to 17.

```
:4,17s/^/\/\//gc
```

We can also quickly uncomment a specific range of lines by replacing `//` with _nothing_.

Uncomment Lines 4 to 17.

```
:4,17s/\/\///gc
```

Simply replace `4` and `17` with the range that you need.

## Split Screen

### Commands

| Keys | Description |
| ---- | ----------- |
| `:sp <filename>` | Open file name `<filename>` in _horizontal_ split screen |
| `:vsp <filename>` | Open file name `<filename>` in _vertical_ split screen |
| `:e <filename>` | Open the file name `<filename>` in the current screen |

### Navigation

| Keys | Description |
| ---- | ----------- |
| ++ctrl++ + ++w++ &nbsp;&nbsp;&nbsp; ++ctrl++ + ++w++ | Change screen |
| ++ctrl++ + ++w++ &nbsp;&nbsp;&nbsp; ++h++ | Change to the _right_ screen |
| ++ctrl++ + ++w++ &nbsp;&nbsp;&nbsp; ++j++ | Change to the _down_ screen |
| ++ctrl++ + ++w++ &nbsp;&nbsp;&nbsp; ++k++ | Change to the _up_ screen |
| ++ctrl++ + ++w++ &nbsp;&nbsp;&nbsp; ++l++ | Change to the _left_ screen |

## Java

| Keys | Description |
| ---- | ----------- |
| `:!javac <filename>.java` | Compile the java file `<filename>.java` |
| `:!java <classname>` | Run the class `<classname>` |
| `:!jshell <filename> <filename> ...` | Start JShell with the given `<filename>`s |