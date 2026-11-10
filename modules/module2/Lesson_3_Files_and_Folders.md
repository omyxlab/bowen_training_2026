# Lesson 2.3: Files and Folders

**Module 2: Working in the Linux Terminal**

## What you will need

- A terminal open in your home folder (`cd ~`)
- The `Training` folder (see [Module 0](../module0/Getting_Started.md))

## What you will learn

By the end of this lesson you will be able to:

- Create folders and organise a project
- Copy, move and rename files and folders
- Delete files and folders, and explain why deleting is dangerous
- Use wildcards to work with many files at once
- Write a short note in the `nano` text editor
- Check how much space files use

## Why this matters

A sequencing project quickly produces dozens of files: raw reads, cleaned reads, alignments, reports, results. If they are not organised, you lose track of what is what. Good habits start now:

1. Keep **one folder per project**.
2. Keep your **raw data separate and untouched**. If you never edit the originals, you can always start again.
3. Give files **clear names** with no spaces.

You will build the folder for the Ebola project in this lesson and use it for the rest of the course.

## Step 1: Create the project folder

`mkdir` stands for **make directory**.

```
$ cd ~
$ mkdir ebola_project
$ ls
ebola_project  Training
```

(Your `ls` may list names in a slightly different order from this lesson. How capital letters are sorted depends on your computer's settings. The contents are what matters.)

Now make a folder **inside** it to hold the raw data:

```
$ mkdir ebola_project/raw
$ ls ebola_project
raw
```

> **No spaces in names.** A name like `my project` is read by the terminal as two separate things. Use underscores (`my_project`) or hyphens instead. Capital letters are allowed, but lowercase is easier to type.

To make several levels at once, add `-p`:

```
$ mkdir -p practice/notes/week1
```

`-p` creates any missing folders along the way and does not complain if one already exists. We will use this folder for practice and delete it later. First, check it:

```
$ ls -R practice
practice:
notes

practice/notes:
week1

practice/notes/week1:
```

`ls -R` lists a folder and **everything inside it** (R for recursive).

## Step 2: Copy files with `cp`

`cp` copies a file. The pattern is `cp  what  where`.

Copy the two Ebola read files into the project's `raw` folder. The originals stay where they are.

```
$ cp Training/short_reads/paired/SRR1553607_1.fastq ebola_project/raw/
$ cp Training/short_reads/paired/SRR1553607_2.fastq ebola_project/raw/
$ ls -lh ebola_project/raw
total 108M
-rwxr-xr-x 1 yourname yourname 54M Nov 10 22:59 SRR1553607_1.fastq
-rwxr-xr-x 1 yourname yourname 54M Nov 10 22:59 SRR1553607_2.fastq
```

Both files are there at 54 MB each. Remember to use **Tab** to complete the long names.

The trailing `/` after `raw` tells `cp` "this is a folder, put the file inside it".

To copy a whole **folder**, add `-r` (recursive), as you will see shortly.

## Step 3: Write a note with `nano`

`nano` is a simple text editor that runs inside the terminal. Use it to keep a record of what you do.

```
$ nano practice/notes.txt
```

A screen opens. Type these three lines:

```
Ebola project notes
Raw reads copied on arrival
Never edit the raw folder
```

Then save and leave:

1. Press **Ctrl+O** (write **O**ut) and then **Enter** to confirm the file name.
2. Press **Ctrl+X** to exit.

The shortcuts are listed along the bottom of the screen, where `^` means Ctrl. You can use your mouse only to focus the window; move around with the arrow keys.

Check that the file exists and see its contents:

```
$ ls practice
notes  notes.txt
$ head practice/notes.txt
Ebola project notes
Raw reads copied on arrival
Never edit the raw folder
```

(`head` prints the beginning of a file; Lesson 2.4 explains it in detail.)

## Step 4: Copy, move and rename

Make a copy of the note:

```
$ cp practice/notes.txt practice/notes_backup.txt
$ ls practice
notes  notes.txt  notes_backup.txt
```

`mv` **moves** a file. It is also how you **rename** one: moving a file to a new name in the same place just changes its name.

```
$ mv practice/notes_backup.txt practice/old_notes.txt
$ ls practice
notes  notes.txt  old_notes.txt
```

Now move that file into a folder. Create the folder first:

```
$ mkdir -p practice/archive
$ mv practice/old_notes.txt practice/archive/
$ ls -R practice
practice:
archive  notes  notes.txt

practice/archive:
old_notes.txt

practice/notes:
week1

practice/notes/week1:
```

The file has left `practice` and is now in `practice/archive`. Unlike `cp`, `mv` leaves **no copy behind**.

Copy a whole folder with `cp -r`:

```
$ cp -r practice practice_copy
$ ls -R practice_copy
practice_copy:
archive  notes  notes.txt

practice_copy/archive:
old_notes.txt

practice_copy/notes:
week1

practice_copy/notes/week1:
```

## Step 5: Delete with `rm`, carefully

`rm` removes files.

```
$ rm practice/archive/old_notes.txt
$ ls -R practice
practice:
archive  notes  notes.txt

practice/archive:

practice/notes:
week1

practice/notes/week1:
```

> **There is no recycle bin.** A file removed with `rm` is **gone for good**. There is no undo.

Before you press Enter on any `rm`, read the command once more. Three habits help:

1. Use **Tab** to complete the name, so you do not mistype.
2. Run `ls` on the same name first to see exactly what you are about to delete.
3. Never run `rm` on a folder that holds your only copy of important data. This is why we keep the raw data separate and copied.

`rm` refuses to remove a folder unless you ask:

```
$ rm practice_copy
rm: cannot remove 'practice_copy': Is a directory
```

Add `-r` to remove a folder and everything inside it:

```
$ rm -r practice_copy
$ ls
ebola_project  practice  Training
```

> **Be extremely careful with `rm -r`.** It deletes everything inside the folder without asking. Never type `rm -r` followed by `/` or `~` or `*` on its own.

## Step 6: Wildcards

A **wildcard** stands in for characters you do not want to type. The `*` matches anything, any number of characters.

List every FASTQ file in the raw folder:

```
$ ls ebola_project/raw/*.fastq
ebola_project/raw/SRR1553607_1.fastq
ebola_project/raw/SRR1553607_2.fastq
```

Only the first of the pair:

```
$ ls ebola_project/raw/*_1.fastq
ebola_project/raw/SRR1553607_1.fastq
```

Select by part of the name. The 16S files are labelled by group:

```
$ ls Training/short_reads/unpaired/*Healthy*
Training/short_reads/unpaired/SRR11282408_Healthy.fastq.gz
Training/short_reads/unpaired/SRR11282409_Healthy.fastq.gz
Training/short_reads/unpaired/SRR11282411_Healthy.fastq.gz
Training/short_reads/unpaired/SRR11282413_Healthy.fastq.gz
```

The `?` matches **exactly one** character:

```
$ ls Training/long_reads/barcode5?.fastq
Training/long_reads/barcode57.fastq
Training/long_reads/barcode58.fastq
```

Wildcards work with any command that takes file names (`ls`, `cp`, `mv`, `rm` and the tools you will use later). That makes them powerful, and dangerous with `rm`. **Always try the pattern with `ls` first.**

## Step 7: How much space? `du`

`du` shows **disk usage**. Use `-s` (summary) and `-h` (human-readable):

```
$ du -sh ebola_project
108M	ebola_project
$ du -sh Training
196M	Training
$ du -sh Training/*
57M	Training/long_reads
139M	Training/short_reads
```

Sequencing data is large, and these are small datasets. Always check that you have space before copying big files.

## Step 8: Tidy up

Remove the practice folder, and confirm that the project folder remains:

```
$ rm -r practice
$ ls
ebola_project  Training
```

Your home folder is now ready for the project, with the raw data safely copied into `ebola_project/raw`.

## If something looks different

| What you see | What it means | What to do |
|---|---|---|
| `cp: cannot stat 'Training/...': No such file or directory` | A wrong path or a typo | Use Tab to complete, and check with `ls` |
| `mkdir: cannot create directory 'ebola_project': File exists` | The folder is already there | Nothing to fix; carry on |
| `cp: -r not specified; omitting directory 'practice'` | You tried to copy a folder without `-r` | Add `-r` |
| `rm: cannot remove 'x': Is a directory` | You used `rm` on a folder | Use `rm -r` (check the name first) |
| `nano` shows a strange screen and you cannot leave | You are still inside `nano` | Press Ctrl+X, and answer `N` if it asks to save |
| `Permission denied` | You are trying to change a file you do not own | Check you are in your own home folder |

## Practice check

1. Which command copies `barcode57.fastq` into a folder called `ebola_project`? (Do not run it; just write it.)
2. What is the difference between `cp` and `mv`?
3. Why do we copy the raw reads rather than work on the originals?
4. What does `ls *.fastq` show?

### Answers

1. `cp Training/long_reads/barcode57.fastq ebola_project/`
2. `cp` leaves the original in place and makes a copy. `mv` moves (or renames) it, leaving nothing behind.
3. So the originals stay untouched and you can start over if you make a mistake.
4. Every file in the current folder whose name ends with `.fastq`.

## Summary of commands

| Command | What it does |
|---|---|
| `mkdir name` | Makes a folder |
| `mkdir -p a/b/c` | Makes nested folders in one go |
| `cp file destination` | Copies a file |
| `cp -r folder destination` | Copies a folder and its contents |
| `mv old new` | Renames or moves a file or folder |
| `rm file` | Deletes a file (permanently) |
| `rm -r folder` | Deletes a folder and everything in it (permanently) |
| `ls -R` | Lists a folder and everything inside it |
| `nano file` | Opens the text editor (Ctrl+O saves, Ctrl+X exits) |
| `*` and `?` | Wildcards: any characters, and exactly one character |
| `du -sh name` | Shows how much space a file or folder uses |

## What comes next

[Lesson 2.4: Looking inside files](Lesson_4_Looking_Inside_Files.md)
