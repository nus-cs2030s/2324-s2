# The Philosophy of `vim`

This article is adapted from the notes of the [Unix@Home Workshop](https://nus-unix-workshop.github.io/2021-s1) 
held in August 2020.

!!! abstract "Learning Objectives"

    Students should

    - appreciate the usefulness of learning `vim` and using it as the main source code editor.
    - appreciate the efficiency and philosophy of using `vim`.
    - have experience navigating around a text buffer and manipulating text in `vim`
    - be aware of how to learn more about using `vim`.

## Background

To edit our code, we need a proper editor.  Remember that, ideally, we want to keep our hands on the keyboard and keep ourselves "_in the zone_" with only the terminal, the keyboard, and ourselves, so we will use a terminal-based editor: no windows, no mouse, no arrow keys, no function keys.

There are only two respectable, widely available text editors in Unix &mdash; `vim` and `emacs`.  Which one is better has been an ongoing religious war, but for us in SoC, we use `vim`.

!!! info "Reading Keyboard Keys"
    Keyboard keys are enclosed in a box like the following letter ++a++.  All keyboard keys are written in uppercase.  You are to press exactly that key without any other key.  So the letter ++a++ should give you the _lowercase_ "a" character.
    
    If a _sequence_ of keys is to be pressed, you will see the keys in successions.  For instance, if you see ++g++ ++g++, it means you should press ++g++ twice.
    
    On the other hand, if multiple keys are to be pressed at the same time, you will see a plus sign (_i.e., +_) written in between the keys.  For instance, if you see ++shift+z+z++, it means you should press ++shift++ key and ++z++ at the same time.  Since we cannot press two different ++z++ at the same time, the second ++z++ is pressed after your finger has been moved from the first ++z++.  However, you should still be pressing the ++shift++ key.
    
    If the intention is for the ++shift++ key to not be pressed for the second ++z++, it will be written as ++shift+z++ ++z++.

## Basic of `vim`
### Minimizing Hand Movements

`vim`, like the shell, aims to minimize hand movements.  Frequently used commands are positioned in convenient places on the keyboard.  Let me give you a few examples.

- To exit vim, type ++shift+z+z++.  Notice that this is located in the bottom left corner of your keyboard.  For normal typing, your left hand is supposed to be placed over the keys ++a++ ++s++ ++d++ ++f++, so you just need to move slightly your left pinky to ++shift++ and left ring finger to ++z++ and hit them.

- To move the cursor, instead of using the arrow keys, `vim` uses ++h++ to move left, ++l++ to move right, ++j++ to move down, and ++k++ to move up.  For normal typing, your right hand is supposed to be placed on ++j++ ++k++ ++l++ ++semicolon++, so these arrow key alternatives are located very near to where your right hand should be!

I have a few more things to say about using ++h++ ++j++ ++k++ ++l++ to replace the arrow keys:

- It is not uncommon for applications to re-map other keys for movement.  Many first-person shooting games use ++w++ ++a++ ++s++ ++d++ for movement, for the same reason as `vim` &mdash; it is close to the resting position of the left hand on the keyboard.

- The use of ++h++ ++j++ ++k++ ++l++ for movement is more ubiquitous than you think.  In the Web version of Gmail, Facebook, and Reddit, for instance, you could use ++j++ and ++k++ to move up and down across posts.

### Multi-modal Editor

`vim` is a multi-modal editor.  While most other editors make no distinction between reading and editing, `vim` makes an explicit distinction between the two.  `vim` has two basic modes:

- `NORMAL` mode: where you read, navigate and manipulate the text.
- `INSERT` mode: where you insert the text

As a programmer, having a separate `NORMAL` mode makes sense since we spend much time reading and navigating around source code.  Thus, allowing the editing commands to be optimized.

In the `NORMAL` mode, you can use any of these keys ++i++ ++s++ ++a++ ++o++ (with or without ++shift++) to switch to `INSERT` mode.  To go back to `NORMAL` mode, press ++esc++.  The keys ++i++ ++s++ ++a++ ++o++ have different meanings, which you will learn later.

Note that most of the time you will be in `NORMAL` mode.  So a habitual `vim` user would insert some text and immediately switch back to normal mode by hitting ++esc++.

### Tell `vim` What You Want To Do; Don't Do It Yourself

In `NORMAL` mode, you can manipulate text in `vim` by issuing commands to `vim`.  These commands are like a programming language.  It is also not unlike the Unix commands, in that each command does a small thing but can be composed together to perform complex text manipulation.

Let me give an example here.  Suppose you have a sentence:

```
Wherever there is light, there is also a shadow.
```

You want to remove `also a` from the sentence.

What would you do in a typical text editor?  You move your hand away from the keyboard, find your mouse, move your mouse cursor to highlight the text, and then hit ++delete++.  Or you could move the cursor (by mouse or by repeatedly hitting the keyboard) to place the cursor after `a`, and then press ++delete++ six times.

In addition to being tedious, this is error-prone.  You might highlight one additional or one less space, or hit ++delete++ one too many times.

What we are used to doing is performing the action of deleting the words ourselves.  For `vim`, we do it differently.  We need to look for the word `also` and delete two words.  This translates to the command ++slash++ ++a++ ++l++ ++s++ ++o++ ++enter++ ++d++ ++2++ ++w++.

- ++slash++ triggers a search.  This is an almost universal command &mdash; try ++slash++ on Facebook (web) or on this page.
- ++a++ ++l++ ++s++ ++o++ ++enter++ tells `vim` what you want to search.
After pressing ++enter++, your cursor should be placed at the beginning of `also`.
- ++d++ ++2++ ++w++ tell `vim` to "delete two words".

Instead of worrying about the actual actions to perform the deletion, we issue higher-level commands to describe what we want to do.  This is powerful since this is how our brain thinks &mdash; "I need to insert this here, change this word to that, remove two lines, etc."  All these maps into commands in `vim`.  As a result, once you master `vim` basics, you can type as fast as you think[^3]!

A common pattern for `vim` command consists of three parts:

1. Place the cursor.
2. Perform an action.
3. Move to the new placement of the cursor.

In the example above,
++slash++ ++a++ ++l++ ++s++ ++o++ ++enter++ places the cursor, ++d++ is the action (delete), and ++2++ ++w++ is the movement (move the cursor forward by two words).

Another common command that students use is ++g++ ++g++ ++equal++ ++shift+g++.  This command is used to indent the source code in the current file.  ++g++ ++g++ is the command to place the cursor at the top of the file.  ++equal++ is the action (indent), and ++shift+g++ is the command to place the cursor on the last line of the file.

### Be A Good Unix Citizen
Not only do the basic commands `vim` adhere to the Unix principles of composability, `vim` plays well with Unix shells, which adds additional power to `vim`.  For instance, if you want to have the standard output from a command paste into the file you are editing, you can run:

```bash
:r! <command>
```

++colon++ triggers the `vim` command line.  ++r++ asks `vim` to read something and paste it into the current cursor location.  At this point, you can pass in, for instance, another file name.  But here, we enter
++exclam++, which tells `vim` to run a shell.  We then pass the `command` to the shell.  Whatever the command writes to the standard output, will be read and inserted into `vim`.

Want to insert today's date?

```bash
:r! date
```

Want to insert a mini calendar?

```bash
:r! cal
```

Want to insert the list of all JPG pictures?

```bash
:r! ls *jpg
```

You can even pass a chunk of text from `vim` to the standard input of another program, and replace it with what is printed to the standard output
by that program.

## Other Reasons To Learn `vim`

Besides enabling you to type as fast as you think with as few hand movements as possible, there are other reasons to use `vim`:

- `vim` is installed by default in almost any Unix environment.  Imagine if you get called to a client-side to debug a Linux server, and you need to edit something &mdash; you can rest assured that `vim` is there.

- `vim` is the only source code editor you need to learn and master.  It works for almost any programming language.  If you use IDE, you have to learn IntelliJ for Java, IDLE for Python, Visual Studio C++ for C++, etc.  This reason is also why VS Code has gained significant popularity in recent years.

- `vim` is extensible and programmable.  It has been around for almost 30 years, and tons of plugins have been written.  Whatever feature you need, there is likely a native `vim` command or a `vim` plugin for that.

The only downside to using `vim` is that it is terminal-based (some consider it ugly) and it has a steep learning curve.  But, in our experience, students will build up their muscle memory and are comfortable with `vim` after 2-3 weeks of usage.

For CS2030S, there is another practical reason to learn and gain familiarity with `vim`.  The practical exams are conducted in a sandboxed environment, which you can only access through `ssh` via a terminal.  You only have a few choices (`emacs`, `micro`, `vim`) and `vim` is the only reasonable choice. 

[^3]: The book _Practical Vim_ by Drew Neil has the subtitle "Edit text at the speed of thought".
