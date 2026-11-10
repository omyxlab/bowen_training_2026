# Lesson 3.2: Quality Control and Trimming

**Module 3: Short-Read Analysis I**

## What you will need

- The `bioinfo` environment active
- `~/ebola_project/raw/` with the two read files, and the folders from Lesson 3.1
- A web browser, to view reports

## What you will learn

By the end of this lesson you will be able to:

- Summarise a sequencing run with `seqkit stats`
- Run FastQC and read the main parts of its report
- Explain why some FastQC "FAIL" results are not real problems
- Trim adapters and low-quality reads with `fastp`
- Compare the data before and after, and judge what trimming cost

## Why this matters

Raw sequencing data are never perfect. Reads can carry leftover **adapter** sequence (the short pieces of DNA added in the laboratory so the machine can read the fragment), bases the machine was unsure about, or be too short to be useful. If we map such reads without checking, the errors flow through to every later result.

So we always do two things first: **look** at the data (quality control, QC) and then **clean** it (trimming). QC tells you whether the data can be trusted. Trimming removes what cannot.

## Step 1: A quick summary with `seqkit stats`

`seqkit stats` summarises sequence files in one table. You met this idea when you counted reads by hand in Lesson 2.4; this command does it for you.

```
$ cd ~/ebola_project
$ seqkit stats raw/*.fastq
file                    format  type  num_seqs     sum_len  min_len  avg_len  max_len
raw/SRR1553607_1.fastq  FASTQ   DNA    203,445  20,547,945      101      101      101
raw/SRR1553607_2.fastq  FASTQ   DNA    203,445  20,547,945      101      101      101
```

| Column | Meaning |
|---|---|
| `num_seqs` | Number of reads |
| `sum_len` | Total bases |
| `min_len`, `avg_len`, `max_len` | Shortest, average and longest read |

Three things to check:

- **Both files have 203,445 reads.** In paired-end data they must match exactly. They do, so the pair is intact.
- **Every read is 101 bases.** As expected for a fixed-cycle Illumina run (Lesson 1.3).
- 203,445 reads matches your hand count in Lesson 2.4.

For more detail, add `-a` (all statistics). The output has more columns than fit here; the ones to note are:

```
$ seqkit stats -a raw/*.fastq
```

| File | Q20 (%) | Q30 (%) | GC (%) |
|---|---|---|---|
| `SRR1553607_1.fastq` | 92.98 | 85.90 | 58.84 |
| `SRR1553607_2.fastq` | 85.47 | 78.15 | 59.24 |

**Q20** and **Q30** are the percentages of bases with a Phred score of at least 20 or 30 (Lesson 1.4). Read 1 is of better quality than read 2, which is normal for Illumina paired-end data: the second read is made later in the run and tends to be a little worse.

## Step 2: Run FastQC

**FastQC** makes a detailed quality report for each file.

```
$ fastqc --outdir qc --threads 2 raw/SRR1553607_1.fastq raw/SRR1553607_2.fastq
```

| Part | Meaning |
|---|---|
| `--outdir qc` | Put the reports in the `qc` folder (it must already exist) |
| `--threads 2` | Use 2 processor cores |
| The last two items | The files to analyse |

You see progress messages (`Approx 5% complete ...`) and it finishes in under 10 seconds:

```
Started analysis of SRR1553607_1.fastq
Approx 5% complete for SRR1553607_1.fastq
...
Analysis complete for SRR1553607_1.fastq
Started analysis of SRR1553607_2.fastq
...
Analysis complete for SRR1553607_2.fastq
```

Check what it made:

```
$ ls -lh qc
total 1.4M
-rw-r--r-- 1 yourname yourname 168K Nov 10 23:24 SRR1553607_1_fastqc.html
-rw-r--r-- 1 yourname yourname 216K Nov 10 23:24 SRR1553607_1_fastqc.zip
-rw-r--r-- 1 yourname yourname 169K Nov 10 23:24 SRR1553607_2_fastqc.html
-rw-r--r-- 1 yourname yourname 215K Nov 10 23:24 SRR1553607_2_fastqc.zip
```

For each file you get an **HTML report** (to read in a browser) and a **zip** (the raw numbers). If your version of FastQC does not recognise `--outdir`, run `fastqc --help` and look for the output-folder option.

### Open the report

A terminal cannot show a web page, so open the HTML file with your normal browser.

- **Windows with WSL:** run `explorer.exe qc` to open the folder in Windows File Explorer, then double-click `SRR1553607_1_fastqc.html`. (You can also type `\\wsl$\Ubuntu\home\yourname\ebola_project\qc` in the File Explorer address bar; the `Ubuntu` part may be named differently on your computer.)
- **Mac:** run `open qc`, then double-click the file.
- **JupyterHub:** in the file browser on the left, find the `qc` folder, and open or download the HTML file.

## Step 3: Read the report

Each report has a list of modules down the left, with a green tick, an orange warning or a red cross. Here is what FastQC says about our raw reads:

| Module | Read 1 | Read 2 |
|---|---|---|
| Basic Statistics | pass | pass |
| Per base sequence quality | pass | pass |
| Per sequence quality scores | pass | pass |
| Per base sequence content | **fail** | **fail** |
| Per sequence GC content | **fail** | **fail** |
| Per base N content | pass | pass |
| Sequence length distribution | pass | pass |
| Sequence duplication levels | pass | pass |
| Overrepresented sequences | warn | pass |
| Adapter content | **warn** | **warn** |

(Your report may differ slightly. Open yours and compare.)

### The ones that matter

**Per base sequence quality** is the most important plot. It shows the quality along the read. The average quality starts around 32 for read 1 and declines gently to around 28.6 at the end; read 2 goes from about 30 to 26. All are comfortably above the "good" threshold of Q20, so this passes.

**Adapter content** warns that some reads contain adapter sequence. These are short fragments where the machine read right through the DNA into the adapter at the other end. We will remove them in a moment.

### The "failures" that are not failures

Two modules fail, and this is a good place to learn that **a red cross is not automatically a problem.**

- **Per base sequence content** fails because the mix of A, C, G and T at the very start of the reads is uneven. This is common in RNA sequencing libraries, where the way the fragments are primed gives the first bases a bias.
- **Per sequence GC content** fails because FastQC expects a single organism and compares your reads to one bell-shaped curve. Our sample is a **mixture** of virus and other reads (you will see why in Lesson 3.3), so the curve is not a single bell. This is a fact about the sample, not a fault in the sequencing.

FastQC applies the same rules to every kind of data, so you must always ask: *is this failure surprising for this kind of sample?*

## Step 4: Trim with fastp

**fastp** cleans paired reads in one go. It will:

- Detect and cut off **adapter** sequence
- Remove reads of **low quality**
- Remove reads that become **too short** after trimming
- Keep the two reads of each pair together

```
$ fastp -i raw/SRR1553607_1.fastq -I raw/SRR1553607_2.fastq \
    -o trimmed/SRR1553607_1.trim.fastq -O trimmed/SRR1553607_2.trim.fastq \
    -l 50 --detect_adapter_for_pe \
    -h qc/fastp_report.html -j qc/fastp_report.json -w 4
```

The `\` at the end of a line means "this command continues on the next line", which keeps a long command readable. You can instead type it all on a single line, leaving out the backslashes.

| Option | Meaning |
|---|---|
| `-i` and `-I` | The input files: read 1 and read 2 |
| `-o` and `-O` | The output files: read 1 and read 2 |
| `-l 50` | Throw away reads shorter than 50 bases after trimming |
| `--detect_adapter_for_pe` | Find the adapter sequences for paired-end data automatically |
| `-h` and `-j` | Write a report as HTML and as JSON |
| `-w 4` | Use 4 processor cores (use a smaller number if your computer has fewer) |

It finishes in a few seconds and prints a summary. The important parts:

```
Detecting adapter sequence for read1...
>I7_Nextera_Transposase_1 | >Trans2_rc | >I7_Nextera_Transposase_1 | >Trans2_rc
CTGTCTCTTATACACATCTCCGAGCCCACGAGAC

Detecting adapter sequence for read2...
>I5_Nextera_Transposase_1 | >Trans1_rc | >I5_Nextera_Transposase_1 | >Trans1_rc
CTGTCTCTTATACACATCTGACGCTGCCGACGA

Read1 before filtering:
total reads: 203445
total bases: 20547945
Q20 bases: 19104638(92.9759%)
Q30 bases: 17651118(85.9021%)
...
Read1 after filtering:
total reads: 168889
total bases: 16663806
Q20 bases: 16362984(98.1948%)
Q30 bases: 15482049(92.9082%)
...
Filtering result:
reads passed filter: 337778
reads failed due to low quality: 68826
reads failed due to too many N: 0
reads failed due to too short: 286
reads failed due to adapter dimer: 0
reads with adapter trimmed: 41022
bases trimmed due to adapters: 836974
```

Reading it:

- fastp found the adapter on its own: a **Nextera** adapter (the name `Nextera_Transposase` tells us which library kit was used).
- **Quality went up.** The share of bases at Q30 or better rose from **85.9% to 92.9%** in read 1, and from **78.1% to 90.1%** in read 2.
- **We lost reads to get it.** 203,445 pairs went in, **168,889 pairs** came out: about **83%** were kept. Most of the loss, 68,826 reads, was for low quality; 286 were too short.
- **41,022 reads had adapter trimmed.**

(The counts here are reads, not pairs: 337,778 reads passed, which is 168,889 pairs. fastp keeps pairs together, so both files always have the same number.)

Check the new files:

```
$ seqkit stats trimmed/*.fastq
file                             format  type  num_seqs     sum_len  min_len  avg_len  max_len
trimmed/SRR1553607_1.trim.fastq  FASTQ   DNA    168,889  16,663,806       50     98.7      101
trimmed/SRR1553607_2.trim.fastq  FASTQ   DNA    168,889  16,663,990       50     98.7      101
```

- Both files still match at **168,889** reads: the pairs are intact.
- The reads are no longer all 101 bases. The minimum is 50 (our `-l 50` limit), and the average is 98.7 because adapters were cut off some reads.

## Step 5: Check again

Run FastQC on the cleaned reads and compare:

```
$ fastqc --outdir qc --threads 2 --quiet trimmed/SRR1553607_1.trim.fastq trimmed/SRR1553607_2.trim.fastq
$ ls qc
SRR1553607_1.trim_fastqc.html  SRR1553607_2_fastqc.zip
SRR1553607_1.trim_fastqc.zip   fastp_report.html
SRR1553607_1_fastqc.html       fastp_report.json
SRR1553607_1_fastqc.zip        SRR1553607_2.trim_fastqc.html
SRR1553607_2_fastqc.html       SRR1553607_2.trim_fastqc.zip
```

(`--quiet` hides the progress messages.) Open the new reports and compare with the old ones:

| Module | Before | After |
|---|---|---|
| Adapter content | warn | **pass** |
| Per base sequence quality | pass (average Q about 26 to 32) | pass (average Q about 30 to 33) |
| Sequence length distribution | pass | **warn** |
| Per base sequence content | fail | fail (same reason as before) |
| Per sequence GC content | fail | fail (same reason as before) |

- The **adapter warning is gone**: it worked.
- **Sequence length distribution now warns**, simply because the reads are no longer all the same length. That is expected after trimming and is not a problem.
- The two composition "failures" remain, for the same reasons as before.

You can also open `qc/fastp_report.html` in your browser for fastp's own before-and-after plots.

## Trimming has a cost

Trimming removed about 17% of the read pairs. That is a deliberate trade: **fewer reads, but better ones.** Whether it is worth it depends on how much data you have to spare. In the next lesson you will see exactly what it cost us, by mapping both the raw and the trimmed reads.

## If something looks different

| What you see | What it means | What to do |
|---|---|---|
| `fastqc: command not found` | The environment is not active | `conda activate bioinfo` |
| `fastqc: error: unrecognized arguments: -o` | This version of FastQC uses `--outdir` | Use `--outdir qc` as shown |
| `Specified output directory 'qc' does not exist` | The folder is missing | `mkdir qc` |
| `fastp: command not found` | The environment is not active | `conda activate bioinfo` |
| fastp keeps a different number of reads | A different fastp version, or different options | Small differences are fine; the files must still have equal counts |
| `bash: -o: command not found` after a long command | A line ended without the backslash `\` | Retype the command on one line |
| The HTML report will not open | The browser needs the Windows path | Use `explorer.exe qc` and double-click the file |

## Practice check

1. Why must the two files of a pair have the same number of reads?
2. Two FastQC modules fail on our raw reads. Are they a sign of poor sequencing? Explain.
3. How many read pairs did `fastp` keep, and what percentage is that?
4. Why does "Sequence length distribution" warn after trimming?

### Answers

1. Because read 1 and read 2 of every pair come from the same fragment. If the counts differ, reads have been lost from one file and the pairing is broken.
2. No. Per base sequence content fails because of how RNA libraries are primed, and per sequence GC content fails because the sample is a mixture of organisms. Neither reflects poor sequencing.
3. 168,889 pairs, which is about 83% of the 203,445 pairs.
4. Because trimming left the reads at different lengths, and FastQC flags any run where lengths are not all equal.

## Summary of commands

| Command | What it does |
|---|---|
| `seqkit stats files` | Summarises read counts and lengths |
| `seqkit stats -a files` | Adds quality and GC statistics |
| `fastqc --outdir qc files` | Makes a quality report for each file |
| `fastp -i R1 -I R2 -o out1 -O out2 -l 50 --detect_adapter_for_pe` | Trims adapters and low-quality reads from a pair |
| `explorer.exe folder` | (Windows with WSL) Opens a folder in File Explorer |

## What comes next

[Lesson 3.3: Mapping reads to a reference](Lesson_3_Mapping_Reads_to_a_Reference.md)
