# Vim Tips

I collected some tips on `vim` that I find helpful for students. 

!!! warning "Prerequisite"
    You have gone through the basic [quick lessons](quick-lessons.md) and have [set up your vim](setup.md) in your PE account.

!!! abstract "Learning Objectives"

    Students should

    - be able to compare files (_e.g., input/output matching for correctness_).
    - be able to recover from `.swp` file in `vim`.

## 1. Useful Configuration

### Showing Line Numbers

If you prefer to show the line number on every line in `vim`, add

```
set number
```

to your `~/.vimrc`.

## 2. Navigation

### Faster Navigation

If you find yourselves typing too many ++h++++j++++k++++l++ to navigate around your code, check out the following shortcuts to navigate around:

To move word-by-word:

- ++w++ jump to the beginning of the next word
- ++b++ ump to the beginning of the previous word (reverse of `w`)
- ++e++ jump to the end of the word (or next word when pressed again)

To search:

- ++f++ char: search forward in the line and sit on the next matching char
- ++t++ char:  search forward in the line and sit on one space before the matching char

++0++ would move you to the beginning of the line, but when coding, it is sometimes useful to jump to the first non-blank character instead.  To do so, use ++shift+6++ (_i.e._, `^`).

In coding, we have many pairs of `[]`, `{}`, `()` and `<>`.  You can use ++shift+5++ (_i.e._, `%`) jump between matching parentheses.

### Jump to a Line

If the compiler tells you there is an error on Line $x$, you can issue `:<x>` to jump to Line $x$.  For instance, `:40` will go to Line 40.

## 3. Editing Operations

### Undo and Redo

Since we are on the topic of correcting mistakes, ++u++ in command mode undo your changes.  Prefix it with a number $n$ to undo $n$ times.  If you want to undo your undo, ++control+r++ will redo.

### Navigation + Editing

`vim` is powerful because you can combine _operations_ with _navigation_.  For instance ++c++ to change, ++d++ to delete, ++y++ to yank (copy).  Since ++w++ is the navigation command to move over the current word, combining them we get:

- ++c++++w++ change the current word (delete the current word and enter insert mode)
- ++d++++w++ delete the current word
- ++y++++w++ yank the current word (copy word into buffer)

Can you guess what each of these does:

- ++d++++f++++shift+0++ 
- ++d++++f++++shift+0++ 
- ++c++++shift+4++
- ++y++++0++

If you repeat the operation ++c++, ++d++, and ++y++, it applies to the whole line, so:

- ++c++++c++ change the whole line
- ++d++++d++ delete the whole line
- ++y++++y++ yank the whole line

You can add a number before an operation to specify how many times you want to repeat an operation.  So ++5++++d++++d++  deletes 5 lines, ++5++++d++++w++ deletes 5 words, etc.

See the article [Operator, the True Power of `Vim`](http://whileimautomaton.net/2008/11/vimm3/operator) for more details.

### Swapping Lines

Sometimes you want to swap the order of two lines of code, in command mode, ++d++++d++++p++ will do the trick.  ++d++++d++ deletes the current line, ++p++ paste it after the current line, in effect swapping the order of the two lines.

### Commenting blocks of code

Sometimes we need to comment out a whole block of code in C for testing purposes. There are several ways to do it in `vim`:

- Place the cursor on the first line of the block of code you want to comment on.
- ++0++ to jump to the beginning of the line
- {++ ++ctrl+v++ enter visual block mode (if your terminal intercepts ++ctrl+v++ as paste, try ++ctrl+q++). ++}
- Use the arrow key to select the block of code you want to comment on.
- ++shift+i++ to insert at the beginning of the line (here, since we already selected the block, we will insert at the beginning of every selected)
- ++slash++++slash++ to insert the C/Java comment character (you will see it inserted in the current line, but don't worry)
- ++escape++ to escape from the visual code.

To uncomment,

- Place the cursor on the first line of the block of code you want to comment.
- ++0++ to jump to the beginning of the line
- ++control+v++ enter block visual mode
- Use the arrow key to select the columns of text containing `//`
- ++x++ to delete them

## 4. Other Useful Commands

### Search and Replace in `vim`

```
:%s/oldWord/newWord/gc
```

`:` enters the command mode.  `%` means apply to the whole document, `s` means substitute, `g` means global (otherwise, only the first occurrence of each line is replaced). `c` is optional &mdash; adding it cause `vim` to confirm with you before each replacement  

### Shell Command

If you need to issue a shell command quickly, you don't have to exit `vim`, run the command, and launch `vim` again.  You can use `!`,

```
:!<command>
```

will issue the command to shell.  E.g.,

```
:!ls
```

You can use this to compile your current file, without exiting `vim`.

```
:!make
```

`make` is a builtin command for `vim`, so you can also simply run

```
:make
```

### Terminal

You can open an interactive shell from within `vim` with:

```
:terminal
```

This command splits the window and add a terminal, within which you can compile or run your code.

### Abbreviation

You can use the command `ab` to abbreviate frequently typed commands.  E.g., in your `~/.vimrc`,

```
ab sop System.out.println
```

Now, when you type `sop `, it will be expanded into `System.out.println`

### Auto-Completion

You can use ++control+p++ or ++control+n++ to auto-complete.  By default, the autocomplete dictionary is based on the text in your current editing buffers.  This is a very useful keystroke saver for long function names and variable names.

### Auto-Indent the Whole File

You can ++g++++g++++equal++++shift+g++ in command mode (_i.e._, type out `gg=G`) to auto-indent the whole file.  ++g++++g++ is the command to go to the beginning of the file.  ++equal++ is the command to indent.  ++shift+g++ is the command to go to the end of the file.

### Split `vim`'s Viewport

- `:sp file.c` splits the `vim` window horizontally
- `:vsp file.c` splits the `vim` window vertically
- ++control+w++++control+w++ moves between the different `vim` viewports

Alternatively, run `vim -O file1 file2` to immediately open both files in two different viewpoints.

### Compare two files

You can compare two files with `vim`, using the `-d` flag.  For instance,

`vim -d file1 file2`

would open up two files for line-by-line comparison.  This is most useful if you want to compare the output of your program with the expected output.

## 5. Recovery Files

Vim automatically saves the files you are editing into temporary _swap_ files, with the extension `.swp`.  These files are hidden, so you don't normally see them when you run `ls`.  (You need to run `ls -a` to view the hidden files)

The swap files are useful if your editing session is disrupted before you save (_e.g._, the network is disconnected, you accidentally close the terminal, your OS crashes, etc).

When you launch `vim` to edit a file, say, `foo.c`.  `vim` will check if a swap file `.foo.c.swp` exist.  If it does, `vim` with display the following

```
Found a swap file by the name ".foo.c.swp"
          owned by: elsa   dated: Sat Aug 21 15:01:04 2021
         file name: ~elsa/foo.c
          modified: no
         user name: elsa   host name: pe116
        process ID: 7863 (STILL RUNNING)
While opening file "foo.c"
             dated: Mon Jul 12 18:38:37 2021

(1) Another program may be editing the same file.  If this is the case,
    be careful not to end up with two different instances of the same
    file when making changes.  Quit, or continue with caution.
(2) An edit session for this file crashed.
    If this is the case, use ":recover" or "vim -r a.c"
    to recover the changes (see ":help recovery").
    If you did this already, delete the swap file ".a.c.swp"
    to avoid this message.

Swap file ".a.c.swp" already exists!
[O]pen Read-Only, (E)dit anyway, (R)ecover, (Q)uit, (A)bort:
```

The messages above are self-explanatory.  Read it carefully.  Most of the time, you want to choose "R" to recover your edits, so that you can continue editing.  Remember to remove the file `.foo.c.swp` after you have recovered.  Otherwise, `vim` will prompt you the above messages every time you edit `foo.c`.

!!! warning
    If `foo.c` is newer than the state saved in `.foo.c.swp`, and you recover from `.foo.c.swp`, you will revert to the state of the file as saved in the swap file.  This can happen if (i) you edit the file without recovery, or (ii) you recover the file, continue editing, but did not remove the `.foo.c.swp` file after.
