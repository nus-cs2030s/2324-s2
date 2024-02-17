# Unix CLI: Background

This article is a replication of the notes from the [Unix@Home Workshop](https://nus-unix-workshop.github.io/2021-s1) held in August 2020.

!!! abstract "Learning Objectives"

    Students should

    - understand the basic terms: CLI, terminal, shell, command prompt.
    - appreciate why CLI is more efficient.
    - aware of the commonly used terminal control sequences.

## What is Command-Line Interfaces?

The command-line interface, or just CLI for short, is an important interface that we, as computing professionals, interact with the computer for most of our day-to-day tasks.

In contrast to graphical user interfaces where users use a mouse to click/drag on menus and windows to interact with a computer, the command-line interface uses a keyboard and text.  The users would type a command to instruct the computer to do something, and the computer would respond by displaying the reply to the user.

CLI evolves from teletypes machines where users would interact with the computer through a typewriter-like machine (see [Figure 2.2.](#figure2_2) of this article for an example).  Users would type a command on the keyboard, and the typewriter would print out, line-by-line, the output on a piece of paper.  This is the era before monitors and mice.  Again, driven by constraints and necessity, CLI interfaces are designed to be simple and economical.  _The commands are short and fast to type; the responses are succinct._

<br><div align="center" id="figure2_1">
<a title="Bubba73 (Jud McCranie) / CC BY-SA (https://creativecommons.org/licenses/by-sa/3.0)" href="https://commons.wikimedia.org/wiki/File:ASR_33.jpg"><img width="256" alt="ASR 33" src="https://upload.wikimedia.org/wikipedia/commons/thumb/1/18/ASR_33.jpg/256px-ASR_33.jpg"></a>
<br>Figure 2.1.: A teletype device (Model 33 ASR) to interact with a computer.
</div><br>

## Why CLI over GUI?

Since CLI is designed to be economical, CLI is much more efficient and productive to use, in particular when we are interacting with a remote computer over the network &mdash; sending text back and forth is much more efficient than sending graphical elements over the network.  Each character takes up to two bytes, but each pixel alone takes up 3 bytes of data.

Another reason why using CLI is faster and more productive is that user can keep their hands on the keyboard at all times and does _not_ need to switch frequently between keyboard and mouse._  While research has shown that GUI and mouse are great for casual users, for software developers that need to type on the keyboard most of the time, having to switch between keyboard and mouse is a productivity-killer.

Further, CLI commands typically provide a host of options that is accessible directly (in contrast to clicking through preference dialogues) from the command line, making these commands flexible and customizable.

Finally, since these commands are just text, we can put together a sequence of commands easily as a _script_, to automate highly repetitive tasks.


## What is a Terminal?

With the advances in Cathode-ray tube (CRT), the teletype machine is replaced with _computer terminals_ in the late 1970s.  Instead of printing the output on paper, the output from CLI is now printed on a monitor supporting 24x80 characters on screen in black and white (or green).

<br><div align=center id="figure2_2">
<a title="Jason Scott / CC BY (https://creativecommons.org/licenses/by/2.0)" href="https://commons.wikimedia.org/wiki/File:DEC_VT100_terminal.jpg"><img width="512" alt="DEC VT100 terminal" src="https://upload.wikimedia.org/wikipedia/commons/thumb/9/99/DEC_VT100_terminal.jpg/512px-DEC_VT100_terminal.jpg"></a>
<br>Figure 2.2.: The VT100 Computer Terminal.
</div><br>

In modern days, operating systems still use similar underlying functionalities to read in keyboard inputs and print the output as text to show to the users, but instead of these clunky special purpose devices, the functionality of a terminal is replaced by programs called _terminal emulator_ or _virtual terminal_.  Examples include `Terminal` and `iTerm2` on macOS; `Windows Terminal` on Microsoft; `xterm` and `konsole` on Ubuntu, etc.  Many legacy control commands on these teletype machines remain in today's computing environment, such as the terminal control sequence.

## What is a Shell?

The term CLI refers to a type of user interface.  To realize this interface, Unix computing environments rely on another type of program called _shell_.

A shell usually works closely with a terminal to get inputs from the users, interpret the meaning of the inputs, execute the tasks (perhaps through the invocation of other programs), and returned the output to the user through the terminal.

Note that a shell can run on its own without a terminal (it can read input from a file, and write the output to a file, for instance).

There are many shells available, each with different bells and whistles to help improve our productivity.

The most popular shell that comes as default on many Unix systems is `bash` or Bourne Again Shell.  This is the shell that we will use in this workshop and as default in the SoC Unix computing environment.

Other popular shells are [`fish`](https://fishshell.com/) and [Oh-my-zsh](https://ohmyz.sh/) (`zsh`).

## Command Prompt

A shell has a _command prompt_. It typically looks something like this, but will be different depending on the default configuration on your machine:

```
ooiwt@pe111:~$
```

The prompt is where you type in a command for the shell to interpret and execute.

In `bash`, the command prompt can be customized to include information such as the username, hostname, time, current working directory, etc.  It is customary to use the `$` sign as the final character of the prompt.  In our examples, we will just show `$` to indicate the command prompt.

Depending on the habit, sometimes you are asked to type in a command "into the terminal", "into the shell", or "into bash".  They all mean the same thing: type in the command at the command prompt of the shell.

## Terminal Control Sequence

On the old teletype machines, a user can send special commands to the teletype machines to control their operation.  Many of these special commands still exist today, and can be triggered by hitting a combination of ++control++ and another key (i.e., a control sequence).

The following lists some of the most useful control sequences to know:

- ++control+d++
:   signal the end of input to a program.  This is also used to exit from a shell (by telling the shell that you have no more input to send, and you are done with it).

- ++control+z++
:   suspend the current running program.  This _pauses_ the execution of the program (but not terminates it).  In the `bash` shell, the most recently suspended program can resume executing in the background with the command `bg` or be brought back to execution in the foreground again with the command `fg`.   

- ++control+c++
:   terminate the current running program.

- ++control+s++
:   freeze the terminal.  This is a legacy control command that pauses the output printing of a teletype machine.  You shouldn't need to use this control sequence.

- ++control+q++
:   resume the terminal.  This is a legacy control command that resumes the printing of a teletype machine.  You shouldn't need to use this control sequence unless you accidentally hit ++control+s++


!!! warning "++control+z++ vs. ++control+c++"
    A common mistake for new students is to hit ++control+z++ frequently if something goes wrong with their program &mdash; this behavior could lead to multiple suspended programs (which still occupy resources such as memory on the computer).  The right sequence to use is ++control+c++ &mdash; which terminates the program (and frees up the resources).

!!! warning "++control+s++ accidents"
    Since ++control+s++ is used as the "save" shortcut in non-Unix environment, many students accidentally hit this control sequence, causing their terminal to freeze.  Don't panic if this happens.  Just hit ++control+q++ and things will be back to normal.

## References

- [The Art of Unix Usability: Command Line Interfaces](http://www.catb.org/~esr/writings/taouu/taouu.html#id3017631), by Eric Steven Raymond
