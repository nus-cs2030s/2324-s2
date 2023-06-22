# The CS2030S Programming Environment

## Java version

Java is a language that continues to evolve.  A new version is released every six months.  For CS2030S, we will _only_ use Java 17, the most recent version with long-term support.  Specifically, we use `openjdk 17 2021-09-14` on Ubuntu 20.04.5.

## PE Hosts

The school has provided a list of computing servers for you to use, with all the required software for CS2030S installed.  You can access them remotely via `ssh`, or Secure SHell.  The hosts are named `pe111`, `pe112`, ..., `pe120`.  (_`pe` stands for "programming environment"_).  We will refer to these servers generally as the _PE hosts._

For this semester, the two servers `pe115` and `pe116` are not available.

You can choose which of the eight hosts to use.  You share the same home directory across all the hosts (_this home directory, however, is different from that of `stu1`_).  If you notice that one host is crowded, you can use another host to spread out the load.

While you can complete the programming assignments on your computers, the practical exams are done in a controlled environment using servers similar to the PE hosts.  It is therefore advisable for you to familiarize yourself with accessing the PE servers via `ssh` and edit your program with either `vim` or `emacs` (_`vim` is recommended and supported_).

## Accessing the PE Hosts

### Basic Requirements

1. You should be familiar with the terms Unix, command-line interface, command prompt, terminal, and shell.  Read this [background article](unix/background.md) if you don't.

1. You need to have an SoC Unix account.  If you do not have one, you can [apply for one online](https://mysoc.nus.edu.sg/~newacct/).

2. Once you have an account, you need to [activate your access to the PE hosts](https://mysoc.nus.edu.sg/~myacct/services.cgi), which are part of the SoC computer clusters.

3. You need a command line `ssh` client.  Windows 10, macOS, and Linux users should already have it installed by default.

4. You need a good [terminal emulator](unix/background.md#what-is-a-terminal).  For Microsoft Windows users, you can use either PowerShell or [Windows Terminal](https://docs.microsoft.com/en-us/windows/terminal/); Mac users can use the default [Terminal](https://support.apple.com/en-sg/guide/terminal/welcome/mac) or [iTerm2](https://iterm2.com/index.html).  

### The Command to SSH

To connect to a remote host, run the following in your terminal on your local computer:

```
ssh <username>@<hostname>
```

Replace `<username>` with your SoC Unix username and `<hostname>` with the name of the host you want to connect to. For instance, I would do:

```
ssh ooiwt@pe112.comp.nus.edu.sg
```

if I want to connect to `pe112.comp.nus.edu.sg`.

After the command above, follow the instructions on the screen.  The first time you ever connect to `pe112.comp.nus.edu.sg`, you will be warned that you are connecting to a previously unknown host.  Answer `yes`.  After that, you will be prompted with your SoC Unix password.  Note that nothing is shown on the screen when your password is entered.

<script id="asciicast-4rtH1KENV6QOdKtlY0T7mce0M" src="https://asciinema.org/a/4rtH1KENV6QOdKtlY0T7mce0M.js" async></script>

### Accessing The PE Hosts from Outside SoC

The PE hosts can only be accessed from __within__ the School of Computing networks.  If you want to access it from outside, you need to connect through SoC VPN.

First, you need to set up a Virtual Private Network (_VPN_) (_See [instructions here](https://dochub.comp.nus.edu.sg/cf/guides/network/vpn)_).  The staff at the IT helpdesk in COM1, Level 1, will be able to help you with setting up if needed.

!!! note "SoC VPN vs NUS VPN"

    Note that SoC VPN is different from NUS VPN.  Connecting to NUS VPN only allows you access to the NUS internal network, but not the SoC internal network.

### Troubleshooting

Some common error messages and what they mean:

1. > `ssh: Could not resolve hostname pe1xx.comp.nus.edu.sg`

    `ssh` cannot recognize the name `pe1xx`. Likely, you tried to connect to the PE hosts directly from outside of the SoC network.

2. > `Connection closed by 192.168.48.xxx port 22`

    You have connected to the PE host, but you are kicked out because you have no permission to use the host.

    Make sure you have activated your access to "SoC computer clusters" here: https://mysoc.nus.edu.sg/~myacct/services.cgi.

3. > `Permission denied, please try again`

    You did not enter the correct password or username.  Please use the username and password 
    of your SoC Unix account which you have created here: https://mysoc.nus.edu.sg/~newacct/.

    Check that you have entered your username correctly.  It is _case-sensitive_.

    If you have lost your password, go here: https://mysoc.nus.edu.sg/~myacct/resetpass.cgi

4. > `Could not chdir to home directory /home/o/ooiwt: Permission denied`

    This error means that you have successfully connected to the PE hosts, but you have no access to your home directory. 

    This should not happen.  Please file a service request with SoC IT Unit at https://rt.comp.nus.edu.sg/. Include the error message, the PE hosts that you connected to, and your username.  The system administrator can reset the permission of your home directory for you.


## Copying Files between PE Nodes and Local Computer

Secure copy, or `scp`, is one way to transfer files between the programming environments and your local computer.  `scp` behaves just like `cp` (_see [Unix: Essentials](unix/essentials.md)_).  The command takes in two arguments, the source, and the destination.  The difference is that we use the `<username>@<hostname>:<filename>` notation to specify a file on a remote host.

Let's say you want to transfer a set of C files from the directory `lab01` to your local computer.  Then, on the local computer, run:

```bash
ooiwt@macbook:~$ scp ooiwt@pe111.comp.nus.edu.sg:~/lab01/*.java .
```

!!! warning
    If you have files with the same name in the remote directory, the files will be overwritten without warning.  I have lost my code a few times due to `scp`.  

The expression `*.java` is a regular expression that means all files with a filename ending with `.java` (see [Advanced Topics on Unix](unix/advanced.md)).
You can copy specific files as well.  For instance, to copy the file `Hello.java` from your local computer to your `~/lab01` directory:

```bash
ooiwt@macbook:~$ scp Hello.java ooiwt@pe111.comp.nus.edu.sg:~/lab01
```

`scp` supports `-r` (_recursive copy_) as well.

Note that we always run `scp` on your local computer in the examples above, since the SSH server runs on the PE host.

## Setting up SSH Keys

The next step is not required but is a time-saver and a huge quality-of-life improvement.  _You need to be familiar with basic Unix commands_, including how to copy files to remote hosts (_using `scp`_) and how to check/change file permissions (_using `ls -l` and `chmod`_).  If you are still not comfortable with these commands, make sure you play with the [basic Unix commands](unix/essentials.md) first.  You can come back and complete this step later. 

Our goal here is to set up a pair of public/private keys for authentication so that you do not need to type your password every time you log into a PE host.

You can use the following command on your local computer to generate a pair of keys:

```
ssh-keygen -t rsa
```

This command will generate two keys, a private key `id_rsa`, and a public key `id_rsa.pub`.  Keep the private key `id_rsa` on your local machine in the hidden `~/.ssh` directory and copy the public key `id_rsa.pub` to your account on PE `pe111`.  

There are two methods to do this:

### Method 1: Using `ssh-copy-id`

If your local machine has `ssh-copy-id` installed, then, run:

```
ssh-copy-id <username>@pe111.comp.nus.edu.sg
```

You will be prompted to enter your password for the PE host.  After this step is completed, your public key will be copied to and configured for password-less login to the PE hosts.

### Method 2: Using `scp`

First, use `scp` to copy the public key `id_rsa.pub` from your local machine to your home directory on PE `pe111`.  

On `pe111`, run

```
cat id_rsa.pub >> ~/.ssh/authorized_keys
```

Make sure that the permission for `.ssh` both on the local machine and on PE is set to `700` and the files `id_rsa` on the local machine and `authorized_keys` on the remote machine are set to `600`.  See the guide on using [`ls`](unix/essentials.md#ls-list-content-of-a-directory) and [`chmod`](unix/essentials.md#file-permission-management) if you are unsure how to do this.

Once set up, you need not enter your password every time you run `ssh` or `scp`.  

## Stability of Network Connection
    
Note that a stable network connection is required to use the PE hosts for a long period without interruption.   If you encounter frequent disconnections while working at home or on campus while connected wirelessly, please make sure that your Wi-Fi signal is strong and that there is no interference from other sources. 

If your connection is disconnected in the middle of editing, `vim` saves the state of the buffer for you.  See the section on [recovery files](vim/tips.md#5-recovery-files) on how to recover your files.

If you find yourself facing frequent disconnection, you can consider running [`screen`](https://en.wikipedia.org/wiki/GNU_Screen).  After logging into a PE host, run:

```
screen
```

You will see some messages, press ++enter++ to go to the command prompt. You can now use the PE host as usual. In case you are disconnected (_e.g., in the middle of editing_), you can log into the same PE host again, and run:

```
screen -r
```

to resume your previous session.
