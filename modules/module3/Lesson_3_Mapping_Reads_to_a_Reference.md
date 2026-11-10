# Lesson 3.3: Mapping Reads to a Reference

**Module 3: Short-Read Analysis I**

## What you will need

- The `bioinfo` environment active
- `~/ebola_project/trimmed/` holding `SRR1553607_1.trim.fastq` and `SRR1553607_2.trim.fastq` (Lesson 3.2)
- `~/ebola_project/reference/NC_002549.1.fasta` (Lesson 3.1)

## What you will learn

By the end of this lesson you will be able to:

- Explain what read mapping is and why it needs an index
- Map paired reads to a reference with `bwa mem`
- Read the first lines of a SAM file
- Convert, sort and index an alignment with `samtools`
- Read a `samtools flagstat` summary correctly
- Explain why most reads in this run do not map to Ebola virus
- Use a pipe to do the whole job in one command

## Why this matters

You now have 337,778 clean reads (168,889 pairs), each 50 to 101 bases long, and a reference genome of 18,959 bases. **Mapping** (also called **alignment**) finds, for each read, the place on the reference where it fits best.

Think of the reference as a picture on a puzzle box and the reads as small puzzle pieces. Mapping places each piece on the picture. Where many pieces land, we know the genome well. Where none land, we know nothing. Pieces that do not fit anywhere belong to a different puzzle.

## Step 1: Index the reference

Searching the whole genome for every read would be far too slow. The mapping program first builds an **index** of the reference, a kind of lookup table, so it can find positions quickly. You do this once per reference.

```
$ cd ~/ebola_project
$ bwa index reference/NC_002549.1.fasta
[bwa_index] Pack FASTA... 0.00 sec
[bwa_index] Construct BWT for the packed sequence...
[bwa_index] 0.00 seconds elapse.
[bwa_index] Update BWT... 0.00 sec
[bwa_index] Pack forward-only FASTA... 0.00 sec
[bwa_index] Construct SA from BWT and Occ... 0.00 sec
[main] Version: 0.7.19-r1273
[main] CMD: bwa index reference/NC_002549.1.fasta
[main] Real time: 0.111 sec; CPU: 0.008 sec
```

It takes a fraction of a second for a genome this small (a human genome takes far longer). Look at what appeared next to the FASTA:

```
$ ls reference
NC_002549.1.fasta      NC_002549.1.fasta.bwt  NC_002549.1.gff3
NC_002549.1.fasta.amb  NC_002549.1.fasta.pac
NC_002549.1.fasta.ann  NC_002549.1.fasta.sa
```

The five new files ending `.amb`, `.ann`, `.bwt`, `.pac` and `.sa` are the index. **Leave them alone**; `bwa` finds them by name.

## Step 2: Map the reads

```
$ bwa mem -t 4 reference/NC_002549.1.fasta trimmed/SRR1553607_1.trim.fastq trimmed/SRR1553607_2.trim.fastq > mapping/SRR1553607.sam
```

| Part | Meaning |
|---|---|
| `bwa mem` | The mapping program, using its standard algorithm for reads of this length |
| `-t 4` | Use 4 processor cores (use a smaller number if your computer has fewer) |
| `reference/NC_002549.1.fasta` | The reference (the index is found automatically) |
| the next two files | The paired reads: read 1 and then read 2 |
| `> mapping/SRR1553607.sam` | Save the results (the `>` from Lesson 2.4) |

`bwa` prints a stream of progress messages. The important ones are:

```
[M::process] read 337778 sequences (33327796 bp)...
[M::mem_pestat] mean and std.dev: (194.47, 76.58)
[M::mem_process_seqs] Processed 337778 reads in 18.642 CPU sec, 4.688 real sec
```

It read all 337,778 reads and finished in about 5 seconds. The `mem_pestat` lines are `bwa` working out how far apart the two reads of a pair usually are: the fragment size, here averaging about 194 bases.

Check the output:

```
$ ls -lh mapping
total 83M
-rw-r--r-- 1 yourname yourname 83M Nov 10 23:24 SRR1553607.sam
```

## Step 3: Look inside the SAM file

The output is in **SAM** format (Lesson 1.4), which is text, so you can look at it with the tools you know. First the header lines, which start with `@`:

```
$ grep "^@" mapping/SRR1553607.sam
@HD	VN:1.5	SO:unsorted	GO:query
@SQ	SN:NC_002549.1	LN:18959
@PG	ID:bwa	PN:bwa	VN:0.7.19-r1273	CL:bwa mem -t 4 reference/NC_002549.1.fasta trimmed/SRR1553607_1.trim.fastq trimmed/SRR1553607_2.trim.fastq
```

The header records the reference (`@SQ`, 18,959 bases long) and the exact command you ran (`@PG`). **The SAM file remembers how it was made.** Notice `SO:unsorted`: the reads are in the order they came in, not in genome order.

Now two alignment lines. `grep -v` shows lines that do **not** match, so `grep -v "^@"` skips the header:

```
$ grep -v "^@" mapping/SRR1553607.sam | head -n 2
SRR1553607.1	77	*	0	0	*	*	0	0	GTTAGCGTTGTTGATC...	BB@FFFFFHHHHHJJJJ...
SRR1553607.1	141	*	0	0	*	*	0	0	TCATTCGTCACATCCC...	CCCFFFFFHHHHHJJJJ...
```

(The sequence and quality columns are shortened here.) The first read pair did **not map**: the reference name is `*`, the position is `0`, and the flag is **77**, which means "paired, this read unmapped, its mate unmapped, first in pair". The second line, flag **141**, is the other read of the pair. Most reads are like this, as you will see in a moment.

## Step 4: Convert to a sorted, indexed BAM

A SAM file is large (83 MB) and in no useful order. Tools prefer a **sorted BAM** file: compressed, and arranged by position on the reference. Sorting is also what makes it possible to ask "show me the reads at position 5,900".

```
$ samtools sort -o mapping/SRR1553607.sorted.bam mapping/SRR1553607.sam
```

`samtools sort` reads the SAM, puts the alignments in order along the genome, and writes a compressed BAM file (`-o` names the output). It prints nothing when it works. Now **index** the BAM, so tools can jump straight to any position:

```
$ samtools index mapping/SRR1553607.sorted.bam
$ ls -lh mapping
total 106M
-rw-r--r-- 1 yourname yourname 83M Nov 10 23:24 SRR1553607.sam
-rw-r--r-- 1 yourname yourname 24M Nov 10 23:24 SRR1553607.sorted.bam
-rw-r--r-- 1 yourname yourname 305K Nov 10 23:24 SRR1553607.sorted.bam.bai
```

The BAM is **24 MB, under a third of the SAM's size**, and the `.bai` file is its index. You no longer need the SAM, so delete it:

```
$ rm mapping/SRR1553607.sam
```

## Step 5: How well did the reads map? `samtools flagstat`

```
$ samtools flagstat mapping/SRR1553607.sorted.bam
337879 + 0 in total (QC-passed reads + QC-failed reads)
337778 + 0 primary
0 + 0 secondary
101 + 0 supplementary
0 + 0 duplicates
0 + 0 primary duplicates
4135 + 0 mapped (1.22% : N/A)
4034 + 0 primary mapped (1.19% : N/A)
337778 + 0 paired in sequencing
168889 + 0 read1
168889 + 0 read2
3798 + 0 properly paired (1.12% : N/A)
3988 + 0 with itself and mate mapped
46 + 0 singletons (0.01% : N/A)
0 + 0 with mate mapped to a different chr
0 + 0 with mate mapped to a different chr (mapQ>=5)
```

| Line | Meaning |
|---|---|
| `337879 in total` | All alignment records. A little more than the reads, because a few reads have more than one alignment |
| `337778 primary` | The reads: one main record each (168,889 × 2) |
| `101 supplementary` | Extra records for reads that match in two pieces |
| `4135 mapped (1.22%)` | Records that matched the reference |
| **`4034 primary mapped (1.19%)`** | **The number of reads that mapped.** This is the one to use |
| `3798 properly paired` | Both reads of a pair mapped in the expected arrangement |
| `46 singletons` | One read of a pair mapped, but its mate did not |

> **Read the right line.** `mapped` includes the extra supplementary records; `primary mapped` counts each read once. When they differ, the primary line tells you how many reads mapped.

### Only 1.19% of the reads mapped

This is the big result. Of 337,778 reads, only **4,034** are Ebola virus. The other **333,744** (98.8%) did not map to the Ebola genome. You can confirm the counts with `-c` (count) and the flags `-F 4` (records that are *not* unmapped) and `-f 4` (records that *are* unmapped):

```
$ samtools view -c -F 4 mapping/SRR1553607.sorted.bam
4135
$ samtools view -c -f 4 mapping/SRR1553607.sorted.bam
333744
```

(4,135 + 333,744 = 337,879, the total.)

Is something wrong? No. **This is normal for a clinical sample.** The sample came from a patient, so the DNA and RNA in it comes from many sources: the patient's own cells, other microbes, and traces from the laboratory process. The virus is a small fraction.

### What are the other reads?

You can look at some of them yourself. `samtools fasta` writes reads as a FASTA file (the format BLAST wants), and `-f 4` picks the unmapped ones. Take the last three, which are fully unmapped pairs:

```
$ samtools fasta -f 4 mapping/SRR1553607.sorted.bam | tail -n 6
>SRR1553607.203444/2
ATCTCAGGGGGAGATGGTTCAGGCCATCTCCTCGCATCGGCCGGATGATGGCCGGACGGCGGCGAGCGGACGACGCCGCGCGGCTGCAGCCCGGCCTGGTC
>SRR1553607.203445/1
CTCGGGTTCCGCACGGTCACCACAACCTCGTTGCCCTCGCCCGCCGCGCGGGCCCACCTGAGCTGGCTGGCCGATTTCTCCGGCCCACCTGGCAGCATGGC
>SRR1553607.203445/2
GATCAAGATCTTCTATCGGCCTGCCATGCGGCCAGGTGGGCCGGAGAAATCGGCCAGCCAGCTCAGGTGGGCCCGCGCGGCGGGCGAGGGCAACGAGGTTG
```

Copy these into NCBI BLAST (<https://blast.ncbi.nlm.nih.gov/>, choose **Nucleotide BLAST**) and see what they match. In our test, one of the three had a clear match to a *Bradyrhizobium* bacterial genome (not every read finds a match, and your results will vary with the database). In a separate check of eight randomly chosen unmapped reads, six matched bacteria such as *Pseudomonas*, *Bradyrhizobium* and *Dietzia*. This is only a handful of reads, so treat it as an illustration, not a result. It fits what FastQC showed: the reads have a GC content of about 58%, which is typical of such bacteria and well above what the Ebola virus genome would give.

## Step 6: Do it all in one command

Writing a big SAM file only to convert it and delete it is wasteful. A **pipe** (Lesson 2.4) can send `bwa`'s output straight into `samtools sort`, so no SAM file is ever written:

```
$ bwa mem -t 4 reference/NC_002549.1.fasta trimmed/SRR1553607_1.trim.fastq trimmed/SRR1553607_2.trim.fastq 2>/dev/null | samtools sort -o mapping/check.sorted.bam -
```

- `2>/dev/null` hides `bwa`'s progress messages (they go to a different channel from the data).
- `|` passes the alignments directly to `samtools sort`.
- The final `-` tells `samtools sort` to read from the pipe instead of a file.

Confirm that it gives the same answer, then tidy up:

```
$ samtools flagstat mapping/check.sorted.bam | head -n 8
337879 + 0 in total (QC-passed reads + QC-failed reads)
337778 + 0 primary
0 + 0 secondary
101 + 0 supplementary
0 + 0 duplicates
0 + 0 primary duplicates
4135 + 0 mapped (1.22% : N/A)
4034 + 0 primary mapped (1.19% : N/A)
$ rm mapping/check.sorted.bam
```

Identical. From now on you can use this one-line form. (You still need `samtools index` afterwards.)

## Optional: what did trimming cost us?

Try mapping the **raw** reads in the same way and compare. Expect these approximate results:

| | Raw reads | Trimmed reads |
|---|---|---|
| Read pairs | 203,445 | 168,889 |
| Reads mapped to Ebola (primary) | about 5,700 | 4,034 |
| Genome covered | 98.97% | 98.97% |
| Mean depth | about 23x | about 17x |
| Mean base quality of mapped reads | about 26.7 | about 32.9 |

Trimming removed about 30% of the Ebola reads, so the depth dropped from 23x to 17x. But the **same 98.97% of the genome** is still covered, and the bases we kept are much more reliable (quality 32.9 against 26.7). That is the trade-off: **less data, better data.**

## If something looks different

| What you see | What it means | What to do |
|---|---|---|
| `[E::bwa_idx_load_from_disk] fail to locate the index files` | You have not run `bwa index` on this FASTA | Run Step 1 |
| `[E::main_mem] fail to open file` | A wrong file name | Check the names with `ls trimmed` and use Tab |
| `samtools: command not found` | The environment is not active | `conda activate bioinfo` |
| `[E::hts_open_format] Failed to open file` | The file does not exist | Check the path and name |
| `[bam_sort_core] merging from 0 files` | A normal message | Ignore it |
| The mapping percentage is very different from 1.2% | A different input or reference | Check you used the trimmed reads and `NC_002549.1.fasta` |
| `samtools: unrecognized option` for flagstat | A typo | Use `samtools flagstat file.bam` |

## Practice check

1. Why do we index the reference before mapping?
2. What does `|` do in the one-line mapping command?
3. How many reads mapped to Ebola virus, and what percentage of all reads is that?
4. Why does `samtools flagstat` show `mapped` and `primary mapped`, and which do you quote?
5. A colleague says "98.8% of reads unmapped means your sequencing failed". How do you respond?

### Answers

1. So the program can find the best position for each read quickly, without scanning the whole genome every time.
2. It sends `bwa`'s output straight into `samtools sort`, so no large intermediate SAM file is created.
3. 4,034 reads, which is 1.19% of the 337,778 reads.
4. `mapped` counts all alignment records, including supplementary ones. `primary mapped` counts each read once, so quote that.
5. The sequencing did not fail. A clinical sample contains reads from the patient and other organisms as well as the virus, so most reads normally do not map to the virus. What matters is that the virus reads we do have cover the genome well, which you check in the next lesson.

## Summary of commands

| Command | What it does |
|---|---|
| `bwa index ref.fasta` | Builds the index of a reference (once) |
| `bwa mem -t N ref R1 R2 > out.sam` | Maps paired reads |
| `samtools sort -o out.bam in.sam` | Sorts alignments and writes a BAM |
| `samtools index file.bam` | Indexes a sorted BAM |
| `samtools flagstat file.bam` | Summarises how the reads mapped |
| `samtools view -c -F 4 file.bam` | Counts mapped records |
| `samtools view -c -f 4 file.bam` | Counts unmapped records |
| `samtools fasta -f 4 file.bam` | Writes unmapped reads as FASTA |
| `bwa mem ... \| samtools sort -o out.bam -` | Maps and sorts in one step |

## What comes next

[Lesson 3.4: Alignment and coverage](Lesson_4_Alignment_and_Coverage.md)
