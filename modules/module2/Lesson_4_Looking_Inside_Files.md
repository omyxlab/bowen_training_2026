# Lesson 2.4: Looking Inside Files

**Module 2: Working in the Linux Terminal**

## What you will need

- A terminal open in your home folder
- `~/ebola_project/raw/` containing `SRR1553607_1.fastq` and `SRR1553607_2.fastq` (from Lesson 2.3)
- The `Training` folder

## What you will learn

By the end of this lesson you will be able to:

- Look at the beginning, end and contents of very large files safely
- Count lines and reads
- Search files for patterns with `grep`
- Join commands together with a pipe `|`
- Save the output of a command to a file with `>`
- Read compressed files without unpacking them

## Why this matters

You cannot, and should not, open a 54 MB sequencing file in a text editor. It would be slow, might freeze the computer, and you could not read millions of lines anyway. Instead, you **peek**: look at the first few lines, count things, and search for patterns. This is how bioinformaticians check that their data are what they expect before doing anything else.

> **Do not use `cat` on sequencing files.** `cat` prints the whole file to the screen. On a large file that means a flood of text. If it ever happens, press **Ctrl+C**.

## Step 1: Start with the first lines, `head`

```
$ cd ~/ebola_project/raw
$ head -n 4 SRR1553607_1.fastq
@SRR1553607.1 1 length=101
GTTAGCGTTGTTGATCGCGACGCAACAACTGGTAAAGAATCTGGAAGAAGGATATCAGTTCAAACGCTCAAGCGAGATGATGGATATTTTTGAACGACTCA
+SRR1553607.1 1 length=101
BB@FFFFFHHHHHJJJJJJJJJJJJJJJJJJJGHIJJJJJJJJJHHHHHFFFFFEEEEEEEEEDDDDDDDDDDDDDDDDDEDDDDEDEEEDDDDDDDDDDD
```

`head -n 4` shows the first **4** lines. That is exactly one read, in the FASTQ format from Lesson 1.4:

1. `@SRR1553607.1 1 length=101`: the read's name
2. The sequence of 101 bases
3. `+SRR1553607.1 1 length=101`: a separator
4. The quality string, one character per base

Ask for eight lines to see two reads:

```
$ head -n 8 SRR1553607_1.fastq
@SRR1553607.1 1 length=101
GTTAGCGTTGTTGATCGCGACGCAACAACTGGTAAAGAATCTGGAAGAAGGATATCAGTTCAAACGCTCAAGCGAGATGATGGATATTTTTGAACGACTCA
+SRR1553607.1 1 length=101
BB@FFFFFHHHHHJJJJJJJJJJJJJJJJJJJGHIJJJJJJJJJHHHHHFFFFFEEEEEEEEEDDDDDDDDDDDDDDDDDEDDDDEDEEEDDDDDDDDDDD
@SRR1553607.2 2 length=101
GGTGTAAGCACAGTACTCGGCCCACATCGCCTTTGTGTTAATGAAGTTTGGGTATCAACTTTCATCCCCAATCTTCCGTGGAAGGAGTATGTTCCGTCAAT
+
?@;BDDDDFHFHFFFGIIGHIIJJGJIGIJIIIIGDGGGHEIGJIIIGIIHJ5@FGHJJIEGGEEHHFFFFFFEEDEDCBB?CCDDD?CDDDDECDCB@>C
```

Without `-n`, `head` shows 10 lines by default.

## Step 2: The last lines, `tail`

```
$ tail -n 4 SRR1553607_1.fastq
@SRR1553607.203445 203445 length=101
CTCGGGTTCCGCACGGTCACCACAACCTCGTTGCCCTCGCCCGCCGCGCGGGCCCACCTGAGCTGGCTGGCCGATTTCTCCGGCCCACCTGGCAGCATGGC
+
CCCFFFDDHHDFHIJIHHGIJJJJJIJJJIIJJJIJJJGJJJGIHFCD@B@BDDDDDBDDDDDDDDDDDDDDDBDDDCEDDDBDBDDDDDDBDDDDBDCDD
```

The last read is numbered **203445**. That already tells us how many reads the file has.

Checking both ends is a good habit. If a file was cut short while downloading, the end will look wrong (an incomplete last read).

## Step 3: Scroll through a file with `less`

`less` opens a file in a viewer you scroll through, without loading it all at once.

```
$ less SRR1553607_1.fastq
```

| Key | What it does |
|---|---|
| **Space** | Next page |
| **b** | Previous page |
| **↓** / **↑** | One line down / up |
| **/** then a pattern, then **Enter** | Search for the pattern; **n** jumps to the next match |
| **G** / **g** | Jump to the end / the start |
| **q** | **Quit** |

**Always remember `q`.** It is the way out. Try it now: open the file, scroll a little, search for `/GATCGATC`, and press `q` to leave.

## Step 4: Count with `wc -l`

`wc` is the **word count** program, and `-l` makes it count **lines**.

```
$ wc -l SRR1553607_1.fastq
813780 SRR1553607_1.fastq
```

The file has 813,780 lines. Compare the two files of the pair:

```
$ wc -l SRR1553607_1.fastq SRR1553607_2.fastq
   813780 SRR1553607_1.fastq
   813780 SRR1553607_2.fastq
  1627560 total
```

Wildcards save typing:

```
$ wc -l *.fastq
   813780 SRR1553607_1.fastq
   813780 SRR1553607_2.fastq
  1627560 total
```

### The reads-from-lines trick

Each read takes **four lines**, so:

> **number of reads = number of lines ÷ 4**

813,780 ÷ 4 = **203,445 reads** in each file. Both files have the same count, as they should in a paired-end run, and it matches the number on the last read.

## Step 5: Search with `grep`

`grep` prints the lines that contain a pattern. Put the pattern in quotes.

```
$ grep "SRR1553607.1 " SRR1553607_1.fastq
@SRR1553607.1 1 length=101
+SRR1553607.1 1 length=101
```

(There are two matching lines, because the name appears in line 1 and line 3 of that read.)

Search for a DNA motif and see where it occurs. `-n` adds the line number and `-m 3` stops after the first three matches:

```
$ grep -m 3 -n "GATCGATC" SRR1553607_1.fastq
1362:ATCCGGCCTGGCTTCGCCGACAGATCGGCGTCGTGTTGCAGGAGAACGTGCTGTTCAACCGCTCGGTGCGCGAGAACATCGCGCTGATCGATCCGTCGATG
3622:GAATAGAGGGGCCTTCACTGGTGCGGCTGATCGATCTGCTGCAGTCTGAAGGCCTGGTCGAGCGTCGTGAGGACCCGACCGACCGGCGCGCCAAGACGCTG
4466:GATCAGACCGGCGTCGAGGATCGAGGAATGCGGATCGATCACGGTGCCGGCGACCGCGATCTCATCGTCGGCGCAGCGCAGGATCCCCTTGAGCGGGCCCT
```

`-c` **counts** matching lines instead of printing them.

### Counting reads with grep: choose the pattern carefully

Every read-name line starts with `@SRR`. The `^` in a pattern means "at the **start** of the line". Count them:

```
$ grep -c "^@SRR" SRR1553607_1.fastq
203445
$ grep -c "^@SRR" SRR1553607_2.fastq
203445
```

203,445 reads in each file, the same answer as lines ÷ 4.

Now see what happens if you pick a looser pattern:

```
$ grep -c "^@" SRR1553607_1.fastq
268597
```

**268,597?** That is wrong. The `@` character is also a valid **quality** symbol, so some quality lines start with `@` too. The pattern was not specific enough.

And a pattern that appears twice per read gives a doubled count:

```
$ grep -c "length=101" SRR1553607_1.fastq
406890
```

That is 2 × 203,445: once in the name line and once in the `+` line.

> **Lesson:** always check a count against another method. Here, lines ÷ 4 and `grep -c "^@SRR"` agree, so we can trust 203,445.

## Step 6: Chain commands with the pipe `|`

The **pipe** `|` takes the output of one command and feeds it into the next. It lets you build small tools from simple parts.

Show the first three read names only:

```
$ grep "^@SRR" SRR1553607_1.fastq | head -n 3
@SRR1553607.1 1 length=101
@SRR1553607.2 2 length=101
@SRR1553607.3 3 length=101
```

Read it left to right: *find every line starting with `@SRR`, then show only the first three of them*.

Count only the reads in the first 400 lines (which should be 100 reads):

```
$ head -n 400 SRR1553607_1.fastq | grep -c "^@SRR"
100
```

Pipes pass data directly from one program to the next without saving anything in between. You will use them constantly, and in the mapping step they save a lot of time and disk space.

## Step 7: Save output to a file with `>`

So far results scroll past on the screen. The `>` symbol sends them into a file instead.

```
$ grep -c "^@SRR" SRR1553607_1.fastq > read_count.txt
$ ls
SRR1553607_1.fastq  SRR1553607_2.fastq  read_count.txt
$ head read_count.txt
203445
```

The screen showed nothing, because the output went into `read_count.txt`.

> **Warning: `>` overwrites.** If the file already exists, its old contents are replaced without asking.

To **add** to a file instead of replacing it, use `>>`:

```
$ grep -c "^@SRR" SRR1553607_1.fastq >> read_count.txt
$ head read_count.txt
203445
203445
```

Tidy up:

```
$ rm read_count.txt
```

You will use `>` throughout the course to keep reports and results.

## Step 8: Compressed files with `zcat`

Large sequencing files are usually **compressed** to save space; their names end in `.gz`. The 16S microbiome files are like this. You do **not** need to unpack them:

- `zcat file.gz` prints the uncompressed contents (like `cat`, but for `.gz` files).
- Combine it with `head`, `grep` and `wc` using a pipe.

```
$ cd ~/Training/short_reads/unpaired
$ zcat SRR11282407_Case.fastq.gz | head -n 1
@SRR11282407.1 1/1
$ zcat SRR11282407_Case.fastq.gz | wc -l
28216
$ zcat SRR11282407_Case.fastq.gz | grep -c "^@SRR"
7054
```

28,216 lines ÷ 4 = **7,054 reads**, matching the `grep` count.

What happens if you forget `zcat` and run `wc -l` directly on the compressed file?

```
$ wc -l SRR11282407_Case.fastq.gz
3761 SRR11282407_Case.fastq.gz
```

3,761 is the number of "lines" in the **compressed** bytes, which is meaningless. Always use `zcat` first for `.gz` files.

## Step 9: Nanopore reads

The long reads are not compressed, so you can use the same commands.

```
$ cd ~/Training/long_reads
$ head -n 1 barcode58.fastq
@e6485134-01fd-4a74-90a4-c696b4927296	st:Z:2024-06-27T02:52:41.739+00:00	RG:Z:fba2136ff67b57066e5c7e23383eba9e075ff2b2_dna_r10.4.1_e8.2_400bps_sup@v4.3.0
$ wc -l *.fastq
   34268 barcode57.fastq
   36784 barcode58.fastq
   11880 sample3.fastq
   82932 total
```

The header is a long random code with extra tags, very different from the Illumina name, as you saw in Lesson 1.4. Dividing by four:

| File | Lines | Reads |
|---|---|---|
| `barcode57.fastq` | 34,268 | 8,567 |
| `barcode58.fastq` | 36,784 | 9,196 |
| `sample3.fastq` | 11,880 | 2,970 |

Notice that these reads are not easy to count with `grep -c "^@"`, because the names do not share a common word the way `SRR` does. That is a good reason to remember lines ÷ 4. Return home when you are done:

```
$ cd ~
```

## If something looks different

| What you see | What it means | What to do |
|---|---|---|
| A rush of text that will not stop | You ran something like `cat` on a large file | Press **Ctrl+C** |
| The screen is stuck in `less` | `less` is still open | Press **q** |
| `grep: SRR1553607_1.fastq: No such file or directory` | You are in the wrong folder | Check with `pwd`, then `cd ~/ebola_project/raw` |
| `grep -c` gives a number that is too big | The pattern matches other lines too | Make the pattern more specific, for example `"^@SRR"` |
| `wc -l` on a `.gz` file gives a small number | You counted the compressed file | Use `zcat file.gz \| wc -l` |
| `zcat: command not found` | Unusual; some systems use `gzcat` | Tell a trainer |

## Practice check

1. A FASTQ file has 2,000,000 lines. How many reads does it contain?
2. You want the first six lines of `barcode57.fastq`. What do you type?
3. What does the `^` mean in `grep "^@SRR"`?
4. What is the difference between `>` and `>>`?
5. How would you count the reads in `SRR11282410_Case.fastq.gz`?

### Answers

1. 500,000 (2,000,000 ÷ 4).
2. `head -n 6 barcode57.fastq`
3. "At the start of the line".
4. `>` replaces the file contents; `>>` adds to the end.
5. `zcat SRR11282410_Case.fastq.gz | wc -l` and divide by 4 (or `zcat SRR11282410_Case.fastq.gz | grep -c "^@SRR"`). Both give 9,115 reads.

## Summary of commands

| Command | What it does |
|---|---|
| `head file` | First 10 lines |
| `head -n 4 file` | First 4 lines |
| `tail -n 4 file` | Last 4 lines |
| `less file` | Scroll through a file (**q** to quit) |
| `wc -l file` | Counts lines |
| `grep "pattern" file` | Prints lines containing the pattern |
| `grep -c "pattern" file` | Counts matching lines |
| `grep -n "pattern" file` | Shows line numbers |
| `grep -m N "pattern" file` | Stops after N matches |
| `^` in a pattern | Start of the line |
| `a \| b` | Sends the output of `a` into `b` |
| `command > file` | Saves output to a file (replacing it) |
| `command >> file` | Adds output to the end of a file |
| `zcat file.gz` | Prints a compressed file's contents |

**Key idea:** reads = lines ÷ 4.

## What comes next

You can now find your way around, manage files and inspect data. In **Module 3** you check the quality of the Ebola reads and map them to a reference genome. Before that, make sure Lesson 2.2 (Conda) is complete so your tools are ready.
