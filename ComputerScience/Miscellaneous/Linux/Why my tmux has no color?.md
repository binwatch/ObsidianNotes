I have installed `tmux` and `oh-my-bash` on my Ubuntu 22.04. However I found that the text in my tmux is all white which is not the same as my colorful bash theme (even if the original terimal). I tried `ls` command, the output text is still all white, but `vim` works normal. Based on these I guess it's a profile issue.

It really confused me because I only know *.bashrc* before. Then I found key to my problem in @[Chris Laidler](https://askubuntu.com/users/1110654/chris-laidler)'s answer to the [Tmux Colors Not Working](https://askubuntu.com/questions/925881/tmux-colors-not-working) question:

> This is a bit of an old one. I resolved this issue by creating a *.bash_profile* file and sourcing my *.bashrc* file.
> To do this, add `source ~/.bashrc` to your *.bash_profile* file.
> The problem in my case was due to my bashrc settings not being sourced via tmux. **Tmux runs as a login shell so it looks for a *.bash_profile* file or a *.bash_login* file**. My colours were being set in my *.bashrc*  file.

So I appended my *.bashrc* file to *.bash_profile*:

```shell
cat ~/.bashrc >> ~/.bash_profile
```

and solved this problem. Now my tmux also uses my bash theme.

````ad-note
title: **\[Extension\]** ⛏ Login Shell v.s. Non-Login Shell & Interactive Shell v.s. Non-Interactive Shell

From `man bash`:

> A  login shell is one whose first character of argument zero is a -, or one started with the --login option.

From the definition of **login shell** above, you can use command:
 
```shell
echo $0
```
 
to see the first character of argument zero, obviously you will get `-bash` in a login shell and `bash` in a non-login shell.
More, if you try to execute `logout` in a non-login shell:

```shell
logout
```
 
You will see the following prompt:

> bash: logout: not login shell: use `exit'
 
If you enter `tmux` (which runs as login-shell) then use `logout` you will find your tmux session exited just like `exit` command.
`logout` is an internal command of the shell. So it is shell-dependent. From `man bash`:

> logout – Exit a login shell.

From `man zsh`:

> logout [ n ] – Same as exit, except that it only works in a login shell.ell
 
So they're completely the same, but logout will simply refuse to work for non-login shells.

**But why we need login shell?**

From @[Stephen Kitt](https://unix.stackexchange.com/users/86440/stephen-kitt)'s answer to [What is the intended purpose of a login shell?](https://unix.stackexchange.com/questions/435964/what-is-the-intended-purpose-of-a-login-shell):

```ad-quote
Login is handled by tools other than the shell, _e.g._ `login` itself, or your desktop manager (with the help of PAM and various other tools).

The _purpose_ of a login shell **isn’t to handle login**, it’s to **behave appropriately as the first shell in a login session**: mainly, that means **processing startup files** which should only be processed once per login session (you will see these files later), and **protecting the login session from unwanted interaction with certain system features** (job suspension in particular).
```

From `man bash`:

> An interactive shell is one started without non-option  arguments  (un‐
less  -s  is  specified) and without the -c option whose standard input
and error are both connected to terminals (as determined by isatty(3)),
or  one  started  with  the -i option.  PS1 is set and $- includes i if
bash is interactive, allowing a shell script or a startup file to  test
this state.

Here the definition of interactive shell seems confusing, tThe following explains how these different kinds of `bash` behave at startup, **please pay attention to the last paragraph**:

```ad-quote
The  following paragraphs describe how bash executes its startup files.
If any of the files exist but cannot be read, bash  reports  an  error.
Tildes  are expanded in filenames as described below under Tilde Expan‐
sion in the EXPANSION section.

When bash is invoked as an interactive login shell, or as a  non-inter‐
active  shell with the --login option, it first reads and executes com‐
mands from the file /etc/profile, if that file exists.   After  reading
that file, it looks for ~/.bash_profile, ~/.bash_login, and ~/.profile,
in that order, and reads and executes commands from the first one  that
exists  and  is  readable.  The --noprofile option may be used when the
shell is started to inhibit this behavior.

When an interactive login shell exits, or a non-interactive login shell
executes  the  exit  builtin  command, bash reads and executes commands
from the file ~/.bash_logout, if it exists.

When an interactive shell that is not a login shell  is  started,  bash
reads  and  executes  commands  from /etc/bash.bashrc and ~/.bashrc, if
these files exist.  This may be inhibited by using the  --norc  option.
The  --rcfile  file option will force bash to read and execute commands
from file instead of /etc/bash.bashrc and ~/.bashrc.

When bash is started non-interactively, to run a shell script, for  ex‐
ample,  it  looks for the variable BASH_ENV in the environment, expands
its value if it appears there, and uses the expanded value as the  name
of  a  file to read and execute.  Bash behaves as if the following com‐
mand were executed:
       if [ -n "$BASH_ENV" ]; then . "$BASH_ENV"; fi
but the value of the PATH variable is not used to search for the  file‐
name.
```

It's clear, when you run a shell script, a **non-interactive shell** is started that _runs the commands_ in the script, and _then exits_ when the script finishes.

Thus, an **interactive shell** is simply any shell process that you use to _type commands_, and _get feedback_ from those commands. This is, [Read–eval–print loop (REPL)](https://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop).
````
