# Vim HOWTO: Common Code Editing Operations

!!! Notes "Work in Progress"
    This article is a work-in-progress that aims to eventually replace the article "Vim Tips" next AY.  As such, you will find some overlaps between the two articles.

## Vim as a Source Code Editor

A source code editor is more than a text editor.  A source code editor is designed and optimized for common operations performed by a programmer while programming.  In this article, we summarize some common operations that you will likely perform and how you will perform them in Vim.

## Code Browsing

!!! Quote "Reading vs. Writing"

    “.. the ratio of time spent reading versus writing is well over 10 to 1. We are constantly reading old code as part of the effort to write new code." 

    ― Robert C. Martin, Clean Code: A Handbook of Agile Software Craftsmanship

As a programmer, we spend a significant amount of time reading source code.  This behavior explains why Vim starts in normal mode for reading and optimizes many shortcuts for navigating source code.  

### Navigation Within a File

_You need to go beyond using arrow keys to move character-by-character or line-by-line for better efficiency and productivity._  Some useful operations are:

| Operation       | Cursor Position | Forward        |  Backward   | |
|-----------------|-|----------------|-------------|-|
| Move word-by-word | Anywhere | w | b | |
| Move page-by-page | Anywhere | CTRL-f | CTRL-b | |
| Move block-by-block | Anywhere | { | } |  A block (or paragraph) is defined as consucutive non-empty lines.  This is useful for instance to navigate method-by-method, provided you have blank lines between two methods |
| Next occurance of word | A word | * | # | Useful for looking for where a variable or method is defined/used |
| Matching bracket | (, ), [, ], {, }, <, or > | % | % | Jump to the matching opening/closing bracket.  Useful for quickly jumping to the start/end of a code block or to check for balanced brackets. |
| Beginning/End of the file | Anywhere | G | gg | Useful for jump to the end of the file (e.g., to append a new method) or to the beginning of the file (e.g., to add comments)

!!! Tips "&lt;count&gt;-&lt;movement&gt;" 
    Instead of repeating the commands above $k$ times, you can issue the command $k$ followed by the movement.  For instance, 6w would move forward by six words.  4-CTRL-f would move forward by 4 pages.

To jump to a particular line, issue the command `:linenumber`.  For example, `:42` would jump to Line 42.

### Navigation Between Files

#### Multiple Buffers

Scenario: Suppose you are reading the code about `Simulator` and you find that it uses a class called `Event`.  You want to read the definition `Event.java`.  There are several ways:

- Place the cursor under the identifier `Event`, then type `gf` (go-to file).  Vim will load the file `Event.java`.  After you are done, to go back to the previous file, use CTRL-^ (^ refers to go-back UP).

- Alternatively, you can use the command `:e Event.java` to tell Vim that you want to edit the file named `Event.java` now.  

#### Multiple Windows

If you want to look at `Event.java` without closing `Simulator.java`, you can split the window with `:sp Event.java` or `:vsp Event.java` to split the buffer (horizontally and vertically).

Use CTRL-w CTRL-w to navigate between windows.

#### Multiple Tabs

Or you can open the file in a new tab with `:tabf Event.java`.  Use `gt` to navigate between tabs.


## Moving Things Around

Often we need to move statements/blocks/methods/classes around, either to fix bugs or to tidy up our code.  We can do that in Vim NORMAL or VISUAL mode.  This often involves two steps: (i) cutting/deleting what you want to move at the source, and (ii) pasting it to the destination.

### Swapping lines

Suppose we have two statements that are out of order.  For instance, we might write
```Java
i = 2 * j;
int j = 0;
```
which does not compile.  We should, of course, declare and initialize `j` first before using it.  Placing the cursor on the line `i = 2 * j`, we can perform the two steps above. (i) Type `dd` to delete the line (essentially move it into the clipboard).  The cursor will move to the line `int j = 0`.  (ii) Type `p` to paste it after the current line.

`ddp` essentially swaps two lines.

You can prefix the command with a number $k$ to delete $k$ lines.

### Reordering Multiple Lines

If you want to cut more lines than you can/willing to count, VISUAL mode is a great way to select the lines to cut.  Place the cursor at the beginning of the line you want to cut.  Press SHIFT-V to enter VISUAL-LINE mode.  Now, use any of the movement commands to move and select the lines you want to cut (remember, if possible you should avoid using arrow keys or `j` or `k` to move line-by-line).   Press `d` to delete.

Now navigate to where you want to paste and press `p`.  

Note that you can navigate to a different file to paste.  This is useful if you want to move a method from one class to another.

!!! Tip "Reformatting After Pasting"

    After pasting a block of code, the indentation of the pasted code might be inconsistent.  Make it a habit to use `gg=G` to reindent your code after pasting.

## Changing Names

### Changing the name of one type/variable/method call

Occasionally, we mix up our variable or our method name, and we need to fix it before the code runs correctly or compiles.  Suppose we have

```Java
double perimeter = circle.getArea();
```

and we realize that we should be calling `getPerimeter` instead.  Instead of using `backspace` or `delete` to delete the characters one by one, we can use `cw` to change the word `getArea` into `getPerimeter`.  

To do so, (i) place the cursor at the beginning of `getArea``.  Remember to avoid using arrow keys or `h` or `l` to move letter-by-letter.  You can use `w` or `b` for faster word-by-word navigation.   (ii) type `cw` to remove the word `getArea` and enter INSERT mode.  Now type `getPerimeter` to replace the method name and ESC to return back to NORMAL mode.  

### Changing multiple names on the same line

Sometimes we have multiple occurrences that we wish to change on the same line.  Let's say:

```Java
Shape s = new Shape();
```

and we realize that we should be creating a `Circle` instead of a `Shape`.

One option is to use `cw` twice.  But we could also use the substitute command, like so.

Place the cursor on this line, and type `:s/Shape/Circle/gc`, and then ENTER.  Here is what it does:

- `:` allows us to issue a command to Vim
- `s/<what to substitute>/<substitute with this>/` tells Vim what we want to replace and replace with what.
- `g` stands for `global` and it says that we want to substitute all occurrences 
- `c` is optional, and it tells Vim to confirm every replacement with us.

### Changing multiple occurrences in a block

Let's say that we copy the following method from a class `LeaveEvent.java` to `JoinEvent.java`:

```Java
  void updateTime(int now) {
      if (this.time > 1200) {
          this.time = now;
      } else {
          this.time = 1200;
      }
  }
```

and we realize that, in `JoinEvent` the corresponding field is called `joinTime` instead of just `time`.  Now we need to rename all occurrences of `time`` within this block to become `joinTime`.  

One way to do this is to use the substitute command again.  There are several ways.

If there are only a few lines and you can count the size of the scope within which you want to search and replace, you place your cursor at the beginning of the method and issue the command `:.,+4s/time/joinTime`.  Here `.` refers to the current line; `,` is a separator, `+4` refers to the next four lines.

Suppose your cursor is far away and you have the line number turned on.  Let say the method above appears at Lines 125 to 131.  You can issue the command `:``125,131s/`time/joinTime`.

Alternatively, you can use VISUAL-LINE mode.  Place the cursor at the beginning of the method, and press SHIFT-V.  This enters the VISUAL-LINE mode.  Now, navigate to select the scope within which you want to search and replace (`5j` or `}` works in this case), and press `:`.  You will see that the command prompt is pre-filled with `:'<,'>` to signify the selected range.  Continue typing `s/time/joinTime` and ENTER.

### Changing all occurrences in a file

Let's say that you have a type in the file `Rectangel.java`.  You have named your class `Rectangel` instead of `Rectangle`, and you want to fix all occurrences of this in the file.  You can use `%` to signify that the range of substitution is the entire file.  

The command `:%s/Rectangel/Rectangle/g` should replace all occurances for you.

## Compiling without Leaving Vim

During development, we go through many iterations of the edit-compile-test cycles.  It would save time if we could do so without leaving Vim.  

### TERMINAL mode

It is often useful to split your Vim window to open up a terminal, by the `:terminal` command.  From the terminal, you can run `javac` to compile and `java` to test.  You can either CTRL-w CTRL-w to switch back to edit your code, or CTRL-D to close the terminal.

### Invoking Shell Commands

You can use `:!` to run a shell command without the terminal.  So `:!javac *.java` would let you compile your code without even leaving Vim.
