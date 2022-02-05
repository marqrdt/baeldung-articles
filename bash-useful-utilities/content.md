# Bash Scripting :: Useful Utilities

In this segment, we'll look at some very useful Linux command-line utilities and dive into how they work, exploring Linux's powerful command-chaining functions.

## Basic setup
This article assumes that you are running a fairly recent version of Linux, such as CentOS 7+ or Ubuntu 18+. Also, it assumes your default shell is **/bin/bash**. Some users have reasons to run legacy shells such as **csh** or **sh** (it will work in Bourne shell), but this utility should also work in newer shells such as **zsh**.

## Directory size inspector

A common problem in managing disk space on desktop and network servers is understanding how disk space is allocated within a directory. This one-liner script will list the sizes of each subdirectory, sorting them from largest to smallest. It can help identify which directories are consuming the most amount of disk space, and the results can be surprising.

Change your home directory:

```
me@myhost:/maybe/not/my/home$ cd
```

and run the following command:

```
me@myhost:~ find . -maxdepth 1 -exec du -sh {} \;
```

### A closer look

This should output a listing of all directories in your home directory, listing the size of the directory along with its name. We'll take a moment to analyze what is happening in this command. First, it employs Linux's **find** command, which will list all files in a given directory, in this case, the current directory, indicated by the dot (.). We use the **-maxdepth** parameter and set it to **1** to only show the top-level directories. It then uses the **-exec** option, which is allows us to send each listing to another command, sometimes expressed as 'piping the output' to another command. The **find** command originates in very early versions of Linux and Unix flavors, so, unlike many other commands, the 'long' options use a single dash '-'.

Each directory name from **find** is passed to the **du -sh** command, which displays the amount of space the directory consumes. The **-sh** option string is actually two options squeezed together. The **-s** option displays a summary of the top directory. Without it, **du** would descend into each subdirectory, which in many Linux home directories, numbers in the 1000's, and not very useful here. The **-h** option displays the output in **h**uman-readable form, which list output as **G**igabytes, **M**egabytes, etc.

The output of our command might look like this:

```
...
14M     ./.vim
503M    ./Videos
4.0K    ./.Xauthority
5.0G    ./.thunderbird
129M    ./.emacs.d
4.9G    ./Downloads
373M    ./virtualenvs
999M    ./.config
12K     ./.fltk
2.1G    ./.cache
...
```

Wait! You mean my **.thunderbird** folder is 5GB? And what about **.cache**? I did mention this command can be very useful.

### Improving our output

The command above is useful, but it can be difficult to inspect for unusually large folders if the directory contains a lrage number of sub-folders. Here, we'll tailor the command to sort the output using the **sort** command. The **sort** command takes several options. We'll look at two very useful ones. First, we'll add **-g**. Like the same option in the **du** command, this option performs a **h**uman-readable sort, and can properly sort file sizes, sorting **373M** before **5.0G**, etc. Here's what the previous listing would look like when we pipe through **sort -h**:

```
# output slightly truncated...
me@myhost:~ find . -maxdepth 1 -exec du -sh {} \; | sort -h
...
4.0K    ./.Xauthority
12K     ./.fltk
14M     ./.vim
129M    ./.emacs.d
373M    ./virtualenvs
503M    ./Videos
999M    ./.config
2.1G    ./.cache
4.9G    ./Downloads
5.0G    ./.thunderbird
...
```

This offers much a much clearer metric of our home directory disk usage. We can also add the **-r** option to **sort** to reverse the output listing, showing the list in descending order of size:

```
# output slightly truncated...
me@myhost:~ find . -maxdepth 1 -exec du -sh {} \; | sort -h
...
5.0G    ./.thunderbird
4.9G    ./Downloads
2.1G    ./.cache
999M    ./.config
503M    ./Videos
373M    ./virtualenvs
129M    ./.emacs.d
14M     ./.vim
12K     ./.fltk
4.0K    ./.Xauthority
...
```

### Make it into a reusable utility

So far, we've been recycling the same, somewhat verbose, command, either by cut-n-paste, or, more easily, using the  <UP-ARROW> key in your terminal. If you find this utility useful, it might be more efficient to convert it to a truly usuable tool. We will employ the **alias** tool in the Bash language. Though **alias*** appears to be a command, there is no **alias** executable, as with **find** or **du**, which exist in **/usr/bin**. Rather, it is an internal function with Bash itself. In our shell session, we will alias our **find** utility to a short command **inspect-dir**.

```
me@myhost:~ alias inspect-dir="find . -maxdepth 1 -exec du -sh {} \; | sort -h"
```

The basic syntax for **alias**is:
```
alias <custom-command>="<actual-command>"
```

The double-quotes in the **alias** command are essential. Once added as an alias, it becomes a new command in our arsenal of tools. Let's use it to get to the next level. If you remember, the **.thunderbird** folder in my home directory was 5GB. It was also a 'dot-file', meaning the filename began with a '.' and was hidden from normal view.

### Persisting our disk inspection utility across our whole environment