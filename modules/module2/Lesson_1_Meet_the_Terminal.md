# Lesson 2.1: Meet the Terminal

**Module 2: Working in the Linux Terminal**

## What you will need

- Your terminal open (Ubuntu on Windows, or Terminal on a Mac)
- The `Training` folder in your home folder (from [Module 0](../module0/Getting_Started.md))

## What you will learn

By the end of this lesson you will be able to:

- Read a terminal prompt and say where you are
- Describe the parts of a command
- Move around the file system with `pwd`, `ls` and `cd`
- Tell absolute paths from relative paths
- Use Tab completion, the history, and `Ctrl+C` to work faster and recover from mistakes
- Get help with `--help` and `man`

## Why this matters

Everything in the rest of the course happens in the terminal, and almost every bioinformatics tool is used there. The terminal has no menus and no undo button. It does exactly what you type, which makes it powerful, and makes it important to know **where you are** and **how to stop** when something goes wrong. This lesson gives you both.

## How to read the commands in this course

Commands are shown after a dollar sign. **Do not type the `$`.** The lines underneath are the output you should expect.

```
$ pwd
/home/yourname
```

Here you type `pwd` and press **Enter**. Your output will show your own username in place of `yourname`. Always press **Enter** after each command.

## Step 1: Open the terminal and read the prompt

Open Ubuntu from the Start menu (Windows), or Terminal (Mac). You see a line like this:

```
yourname@YOUR-PC:~$
```

This is the **prompt**. It is the terminal telling you it is ready for a command.

| Part | Meaning |
|---|---|
| `yourname` | Your username |
| `@YOUR-PC` | The name of the computer |
| `:` | A separator |
| `~` | Where you are now. The `~` (tilde) is shorthand for your **home folder** |
| `$` | "Ready for your command" |

If you have created the `bioinfo` environment, your prompt may start with `(bioinfo)` or `(base)`. That is fine; we cover it in the next lesson.

## Step 2: The parts of a command

Every command follows the same pattern:

```
command   -options   arguments
```

> The exception here is `pwd`, which has no options or arguments. It is a command that does one thing: print the current folder. Another exception is the `mv` command, which has two arguments: the source and the destination. For example:

For example:

```
ls -lh Training
```

- **`ls`** is the command (what to do).
- **`-lh`** is a set of **options** (how to do it). Options usually start with `-`.
- **`Training`** is the **argument** (what to do it on).

The spaces matter. `ls -lh Training` works. `ls-lh Training` does not.

> **Linux is case-sensitive.** `Training` and `training` are different names. If a command cannot find something you can see, check the capital letters first.

## Step 3: Where am I? `pwd`

`pwd` stands for **print working directory**. It tells you the folder you are currently in.

```
$ pwd
/home/yourname
```

You are in your home folder. The text is called a **path**: the address of a folder, written from the top of the file system. `/home/yourname` means "inside `home`, inside `yourname`".

A **directory** is simply a folder. The words are used interchangeably.

## Step 4: What is here? `ls`

`ls` lists what is in the current folder.

```
$ ls
Training
```

Right now you see only `Training`. (If you have already done other things, you may see more.) Add options to see more detail:

```
$ ls -l
total 4
drwxr-xr-x 4 yourname yourname 4096 Nov  8 18:01 Training
```

`-l` gives the **long** listing. Reading across: permissions, owner, group, size in bytes, date, and name. The `d` at the very start of `drwxr-xr-x` tells you `Training` is a **directory**. A file starts with `-`.

Now add `-h` for **human-readable** sizes (K for kilobytes, M for megabytes):

```
$ ls -lh Training
total 8.0K
drwxr-xr-x 2 yourname yourname 4.0K Nov  8 18:01 long_reads
drwxr-xr-x 4 yourname yourname 4.0K Nov  8 18:01 short_reads
```

Options can be combined: `-lh` means `-l` and `-h` together.

`ls` also accepts a folder name as an argument, so you can look inside a folder without going there:

```
$ ls Training/long_reads
barcode57.fastq
barcode58.fastq
sample3.fastq
```

Finally, `-a` shows **hidden** files, which begin with a dot. On a fresh account you will see at least these:

```
$ ls -a
.  ..  .bash_logout  .bashrc  .profile  Training
```

(Your list will be longer if you have installed Miniforge, because it adds some hidden items.) The single dot `.` means "this folder" and the double dot `..` means "the folder above".

## Step 5: Moving around. `cd`

`cd` stands for **change directory**. Give it the name of a folder to go there.

```
$ cd Training
$ pwd
/home/yourname/Training
$ ls
long_reads  short_reads
```

Notice that `cd` prints nothing when it works. The only way to know it worked is to look at your prompt, or to type `pwd`.

Go further in, one folder at a time:

```
$ cd short_reads
$ ls
paired  unpaired
$ cd paired
$ pwd
/home/yourname/Training/short_reads/paired
$ ls -lh
total 108M
drwxr-xr-x 2 yourname yourname 4.0K Nov  8 18:01 SRR1553607
-rwxr-xr-x 1 yourname yourname  54M Jan 16  2026 SRR1553607_1.fastq
-rwxr-xr-x 1 yourname yourname  54M Jan 16  2026 SRR1553607_2.fastq
-rwxr-xr-x 1 yourname yourname  236 Jun 26 14:00 test.rotimi
```

These are the two paired Ebola read files, 54 MB each. The folder `SRR1553607` and the small file `test.rotimi` are extras in the bundle that we will not use; you can ignore them.

### Going back up

`..` means "the folder above this one".

```
$ cd ..
$ pwd
/home/yourname/Training/short_reads
$ cd ../..
$ pwd
/home/yourname
```

`../..` means "up two levels". You can chain them.

### Shortcuts that save a lot of time

| Command | What it does |
|---|---|
| `cd ~` or just `cd` | Go to your home folder, from anywhere |
| `cd ..` | Go up one level |
| `cd -` | Go back to the folder you were in **before** (like a back button) |
| `cd /` | Go to the very top of the file system (called the **root**) |

Try them:

```
$ cd Training/short_reads
$ cd -
/home/yourname
$ cd ~
$ pwd
/home/yourname
$ cd /
$ pwd
/
$ cd ~
```

> **If you ever feel lost, type `cd ~` and then `pwd`.** You are back home.

## Step 6: Absolute and relative paths

There are two ways to write a path.

- An **absolute path** starts with `/` and works from anywhere. It is the full address: `/home/yourname/Training/long_reads`.
- A **relative path** does *not* start with `/`. It is worked out **from where you are now**: `Training/long_reads` only works if you are in your home folder.

```
$ cd /home/yourname/Training/long_reads
$ pwd
/home/yourname/Training/long_reads
$ cd ~
$ cd Training/long_reads
$ pwd
/home/yourname/Training/long_reads
$ cd ~
```

Both took you to the same place. Relative paths are shorter; absolute paths are safer when you are not sure where you are. And `~` can be used at the start of a path: `cd ~/Training` works from anywhere.

## Step 7: Typing less, and fixing mistakes

### Tab completion

Start typing a name and press **Tab**. The terminal completes it for you.

1. Type `cd Tra` and press **Tab**. It becomes `cd Training/`.
2. Type `ls Training/sh` and press **Tab**. It completes to `short_reads/`.

If more than one name matches, press **Tab** twice to see the choices. **Use Tab all the time.** It is faster, and it protects you from typing errors.

### The history

Press the **Up arrow (↑)** to bring back the previous command, and press it again to go further back. Press **Down arrow (↓)** to go forward. You can edit a recalled command before pressing Enter.

You can see your earlier commands with:

```
$ history
```

### Stopping a command: `Ctrl+C`

If a command is taking too long, or printed far more than you wanted, press **Ctrl** and **C** together. This interrupts it. It is the emergency brake, and you will use it more than once. (On a Mac the same key combination, Control+C, is used.)

### Clearing the screen

```
$ clear
```

This only tidies the screen. Nothing is deleted. `Ctrl+L` does the same.

### Copy and paste

Ctrl+C and Ctrl+V do **not** copy and paste in the terminal, because Ctrl+C means "stop". Instead:

| Action | Windows (Ubuntu) | Mac Terminal |
|---|---|---|
| Copy | Ctrl+Shift+C | Command+C |
| Paste | Ctrl+Shift+V, or right-click | Command+V |

## Step 8: Getting help

Most commands explain themselves.

```
$ ls --help | head -6
Usage: ls [OPTION]... [FILE]...
List information about the FILEs (the current directory by default).
Sort entries alphabetically if none of -cftuvSUX nor --sort is specified.

Mandatory arguments to long options are mandatory for short options too.
  -a, --all                  do not ignore entries starting with .
```

(We used `| head -6` to show only the first six lines; you will learn about `|` in Lesson 2.4. Without it, the help text is long.)

For a fuller manual, use `man`:

```
$ man ls
```

This opens a manual page you scroll with the arrow keys and **space**. **Press `q` to quit.** If you ever find yourself stuck in a screen that will not accept commands, press `q`.

Many bioinformatics tools use `--help` too, as you will see: `seqkit --help`, `samtools --help`, and so on.

## If something looks different

| What you see | What it means | What to do |
|---|---|---|
| `bash: cd: Training/shortreads: No such file or directory` | The name is wrong (a typo, or a missing `_`) | Check spelling and capital letters. Use `ls` to see the real names, or use Tab |
| `ls: cannot access 'Training/nothing': No such file or directory` | The path does not exist from where you are | Check with `pwd` where you are. Try an absolute path |
| `Command 'lss' not found` | A typo in the command name | Retype it |
| The terminal seems frozen | A command is still running, or you are in a viewer such as `man` | Press `Ctrl+C`, or press `q` |
| `pwd` shows a path starting with `/mnt/c/` | You are inside your Windows files | Type `cd ~` to return to your Linux home folder |

## Practice check

Using only what you have learned, answer these from the terminal.

1. How big is `barcode58.fastq`, in human-readable form?
2. How many items are in the `unpaired` folder? (Count everything you see.)
3. Which folder are you in after running `cd Training/short_reads/paired` and then `cd ../..`?
4. You are somewhere deep in the file system and have no idea where. What two commands get you home and confirm it?

### Answers

1. 28M. (Use `ls -lh Training/long_reads`.)
2. Nine: seven `.fastq.gz` read files, plus `download.sh` and `test.rotimi`.
3. `/home/yourname/Training`. The `..` goes up one level from `paired` to `short_reads`, and the second `..` goes up from `short_reads` to `Training`.
4. `cd ~` and then `pwd`.

## Summary of commands

| Command | What it does |
|---|---|
| `pwd` | Shows where you are |
| `ls` | Lists files and folders |
| `ls -l` | Long listing (details) |
| `ls -h` | Human-readable sizes (use with `-l`) |
| `ls -a` | Includes hidden files |
| `cd folder` | Moves into a folder |
| `cd ..` | Moves up one level |
| `cd ~` or `cd` | Moves to your home folder |
| `cd -` | Moves back to the previous folder |
| `clear` | Clears the screen |
| `history` | Shows earlier commands |
| `command --help` | Shows brief help |
| `man command` | Shows the full manual (press `q` to quit) |
| Tab | Completes names |
| ↑ | Recalls the previous command |
| Ctrl+C | Stops the running command |

## What comes next

[Lesson 2.2: Preparing your environment with Conda](Lesson_2_Preparing_Your_Environment_with_Conda.md)
