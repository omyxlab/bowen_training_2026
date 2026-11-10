# Lesson 1.4: Biological File Formats

**Module 1: Welcome and Introduction to Bioinformatics**

## What you will learn

By the end of this lesson you will be able to:

- Recognise FASTA, FASTQ, GFF3, BED, SAM/BAM and VCF files by how they look
- Read the main fields of each format
- Decode a FASTQ record and explain what a Phred quality score means
- Say which format goes into and comes out of each step of an analysis

## Why formats matter

Bioinformatics tools pass data to each other, so they need agreed ways of writing it down. Each **file format** is a set of rules for what goes on each line. Once you know the common formats, you can open almost any bioinformatics file and understand what you are looking at.

Nearly all of them are **plain text**. That means you can look inside with simple tools, which you will do from Module 2 onwards. (A file written in Word or Excel format is *not* plain text, and bioinformatics tools cannot read it.)

All the examples below come from the course data.

## FASTA: sequences

A **FASTA** file holds one or more sequences. Each sequence has a **header line** starting with `>`, followed by the sequence itself, usually split over several lines.

This is the start of the Ebola virus reference genome:

```
>NC_002549.1 Zaire ebolavirus isolate Ebola virus/H.sapiens-tc/COD/1976/Yambuku-Mayinga, complete genome
CGGACACACAAAAAGAAAGAAGAATTTTTAGGATCTTTTGTGTGCGAATAACTATGAGGAAGATTAATAA
TTTTCCTCTCATTGAAATTTATATCGGAATTTAAATTGAAATTGTTACTGTAATCACACCTGGTTTGTTT
...
```

- The header begins with the **identifier** (`NC_002549.1`), then a free-text description.
- Everything after the header, until the next `>`, is the sequence.
- FASTA can also store protein sequences.

**Used for:** reference genomes, assembled sequences, consensus sequences, and database search queries.

## FASTQ: reads with quality

A **FASTQ** file stores sequencing reads together with a quality score for every base. **Each read takes exactly four lines:**

| Line | Starts with | Content |
|---|---|---|
| 1 | `@` | The read's identifier |
| 2 | | The sequence |
| 3 | `+` | A separator (sometimes repeats the identifier) |
| 4 | | The quality string, one character per base |

The first read from the Ebola data (`SRR1553607_1.fastq`):

```
@SRR1553607.1 1 length=101
GTTAGCGTTGTTGATCGCGACGCAACAACTGGTAAAGAATCTGGAAGAAGGATATCAGTTCAAACGCTCAAGCGAGATGATGGATATTTTTGAACGACTCA
+SRR1553607.1 1 length=101
BB@FFFFFHHHHHJJJJJJJJJJJJJJJJJJJGHIJJJJJJJJJHHHHHFFFFFEEEEEEEEEDDDDDDDDDDDDDDDDDEDDDDEDEEEDDDDDDDDDDD
```

The sequence and the quality string are the same length (101 here): the first quality character belongs to the first base, and so on.

Because every read is four lines, **the number of reads is the number of lines divided by four**. You will use this in Module 2.

### Different machines write different headers

An Illumina read, as above, has a short identifier. An Oxford Nanopore read (from `barcode58.fastq`) looks different:

```
@e6485134-01fd-4a74-90a4-c696b4927296   st:Z:2024-06-27T02:52:41.739+00:00   RG:Z:fba2136f...dna_r10.4.1_e8.2_400bps_sup@v4.3.0
GAGCGCCTTATGCATAGTTTTTGAGAAAACTAAGTTGTTTTTGTGTATTATCGTATGGAAAAGTC...
+
)++,,,,,,85***-.689;@>6566DLLIQGEFJIHKOKSSSMQKIGHGE>====IIMROINS***)...
```

The identifier is a long random code, followed by tags that record the start time and the sequencing set-up. (The line above is shortened with `...` for display, and the parts are separated by tab characters.)

### Phred quality scores

Each quality character encodes a **Phred score**, which tells you how likely that base is to be wrong:

> **Q = -10 × log10(probability that the base is wrong)**

| Phred score | Chance the base is wrong | Accuracy |
|---|---|---|
| Q10 | 1 in 10 | 90% |
| Q20 | 1 in 100 | 99% |
| Q30 | 1 in 1,000 | 99.9% |
| Q40 | 1 in 10,000 | 99.99% |

Higher is better. A common rule of thumb is that Q30 or above is good quality.

Characters are used because a number like `37` would take two characters and break the "one character per base" rule. The score is found by taking the character's ASCII code and subtracting **33**. A few useful values:

| Character | `!` | `+` | `5` | `?` | `I` |
|---|---|---|---|---|---|
| Phred score | 0 | 10 | 20 | 30 | 40 |

So in the Ebola read above, `B` is Q33, `@` is Q31, `F` is Q37, `H` is Q39 and `J` is Q41: high quality throughout.

One warning: the `@` and `+` symbols can also appear in the quality line, because they are valid quality characters. A line that starts with `@` is therefore *not* always the start of a read. This is why we count reads by dividing the line count by four, rather than counting `@` lines.

## GFF3: gene annotation

A **GFF3** file describes *features* on a sequence, such as genes, along with their coordinates. Each line has **nine tab-separated columns**:

| Column | Name | Example |
|---|---|---|
| 1 | seqid | `NC_002549.1` (which sequence) |
| 2 | source | `RefSeq` |
| 3 | type | `gene`, `CDS`, `exon` |
| 4 | start | `56` |
| 5 | end | `3026` |
| 6 | score | `.` (none) |
| 7 | strand | `+` or `-` |
| 8 | phase | `.` or 0, 1, 2 (for coding regions) |
| 9 | attributes | `ID=gene-ZEBOVgp1;...;Name=NP;...` |

Real lines from the Ebola reference annotation (attributes shortened):

```
NC_002549.1   RefSeq   gene    56   3026   .   +   .   ID=gene-ZEBOVgp1;Dbxref=GeneID:911830;Name=NP;...
NC_002549.1   RefSeq   gene   3032  4407   .   +   .   ID=gene-ZEBOVgp2;Dbxref=GeneID:911827;Name=VP35;...
NC_002549.1   RefSeq   gene   4390  5894   .   +   .   ID=gene-ZEBOVgp3;Dbxref=GeneID:911825;Name=VP40;...
```

The Ebola genome has seven genes:

| Gene | Start | End |
|---|---|---|
| NP | 56 | 3026 |
| VP35 | 3032 | 4407 |
| VP40 | 4390 | 5894 |
| GP | 5900 | 8305 |
| VP30 | 8288 | 9740 |
| VP24 | 9885 | 11518 |
| L | 11501 | 18282 |

Coordinates in GFF3 are **1-based**: the first base of the sequence is position 1. A closely related format called **GTF** is used for many genome annotations; it has the same nine columns with a different attribute style.

**Used for:** finding out which gene a position (for example a variant) falls in.

## BED: regions

A **BED** file is a simpler way of listing regions. It has at least three tab-separated columns: the sequence name, the **start**, and the **end**. A fourth column is usually a name.

The NP gene from the table above, written as BED:

```
NC_002549.1   55   3026   NP
```

The start is **55**, not 56. BED uses **0-based** starts (the first base is position 0) where GFF3 uses 1-based, so the same region has a start one lower. This is a common source of off-by-one mistakes, so watch for it when moving between formats.

**Used for:** regions of interest, for example to pull out coverage over a gene.

## SAM and BAM: alignments

Once reads are mapped to a reference, the results are stored in **SAM** (a text format) or **BAM** (its compressed, binary version, which is much smaller). Tools such as `samtools` can show you what is inside a BAM file.

A SAM file has **header lines** starting with `@`, followed by one line per read:

```
@HD   VN:1.5   SO:coordinate
@SQ   SN:NC_002549.1   LN:18959
@PG   ID:bwa   PN:bwa ...
```

Each alignment line has at least 11 columns. A real example from the Ebola data (sequence and quality strings shortened):

```
SRR1553607.131793   99   NC_002549.1   104   60   101M   =   142   139   AATTTCAATTG...   (quality)
```

| Column | Name | Value here | Meaning |
|---|---|---|---|
| 1 | QNAME | `SRR1553607.131793` | The read's name |
| 2 | FLAG | `99` | A number encoding facts about the read (for 99: paired, properly paired, and the first read of its pair) |
| 3 | RNAME | `NC_002549.1` | The reference sequence it mapped to |
| 4 | POS | `104` | Where on the reference it starts (1-based) |
| 5 | MAPQ | `60` | How confident the mapping is (higher is better) |
| 6 | CIGAR | `101M` | How the read lines up: here, all 101 bases align |
| 7 | RNEXT | `=` | The mate maps to the same sequence |
| 8 | PNEXT | `142` | Where the mate starts |
| 9 | TLEN | `139` | The length of the whole fragment |
| 10 | SEQ | | The read's sequence |
| 11 | QUAL | | The read's quality string |

A BAM file is usually **sorted** and accompanied by an **index** (a `.bai` file) that lets tools jump straight to any region.

**Used for:** storing where every read maps. This is the main output of the mapping step.

## VCF: variants

A **VCF** file lists the places where a sample **differs from the reference**. It has two kinds of header lines (`##` for descriptions, and a single `#CHROM` line naming the columns), followed by one line per variant.

```
##fileformat=VCFv4.2
...
#CHROM        POS  ID  REF  ALT  QUAL     FILTER  INFO
NC_002549.1   295  .   A    C    225.417  .       DP=21;...;MQ=60
NC_002549.1   409  .   T    C    225.417  .       DP=22;...;MQ=60
```

| Column | Meaning |
|---|---|
| CHROM | The reference sequence |
| POS | Position of the variant (1-based) |
| REF | The base in the reference |
| ALT | The base found in the sample |
| QUAL | How confident the caller is that this is a real variant (higher is better) |
| FILTER | Whether the variant passed filters (`.` means none were applied) |
| INFO | Extra details, such as `DP`, the number of reads covering the position |

Read the first variant like this: *at position 295 of the reference, the reference has an A, the sample has a C, and 21 reads covered that position.*

These example lines are only illustrative. When you run variant calling yourself in Module 4, your numbers may differ slightly depending on settings and software versions.

**Used for:** recording and comparing the genetic differences found in a sample.

## Compressed and index files

| Ending | What it is |
|---|---|
| `.gz` | Compressed with gzip (for example `reads.fastq.gz`). Tools can read these directly, and you can look inside with `zcat` |
| `.bam` | Binary, compressed alignments |
| `.bai` | The index for a BAM file |
| `.fai` | The index for a FASTA file |

Several different endings mean the same format: `.fa`, `.fasta` and `.fna` are all FASTA; `.fq` and `.fastq` are both FASTQ.

## How the formats fit together

```
 FASTQ (raw reads)
     |  quality control and trimming
     v
 FASTQ (clean reads)  +  FASTA (reference genome)
     |  mapping
     v
 BAM (alignments)
     |                                     GFF3 (gene annotation)
     |  variant calling                          |
     v                                           |
 VCF (variants) ---------- which gene? ----------+
     |
     |  consensus
     v
 FASTA (the sample's own genome)
```

## Where each format appears in this course

| Format | You will meet it in |
|---|---|
| FASTA | Fetching the reference (Module 3); the consensus genome (Module 4) |
| FASTQ | Looking inside files (Module 2); quality control (Module 3); long reads (Module 5) |
| GFF3 | Finding which gene a variant is in (Module 4) |
| BAM and SAM | Mapping and coverage (Module 3); host removal (Module 5) |
| VCF | Variant calling (Module 4) |
| BED | Mentioned for reference only |

## Key terms

| Term | Meaning |
|---|---|
| **Plain text** | A file you can read with simple tools; no special program is needed |
| **Header** | A line at the start of a file or record that describes what follows |
| **Phred score (Q)** | A number expressing the probability that a base is wrong |
| **1-based / 0-based** | Whether the first position of a sequence is numbered 1 or 0 |
| **Alignment** | The location where a read lines up on the reference |
| **Variant** | A position where the sample differs from the reference |
| **Index** | A companion file that lets tools find data quickly |

## Check your understanding

1. How many lines does one read take in a FASTQ file? If a file has 800,000 lines, how many reads does it hold?
2. A base has a quality score of Q30. What is the chance that it is wrong?
3. Which format would you use to store a reference genome? Which would you use to store the differences found in a sample?
4. A gene starts at position 56 in a GFF3 file. What start would a BED file give for the same gene?
5. A VCF line reads `NC_002549.1  1000  .  G  T  ...`. In words, what does it say?

### Answers

1. Four lines per read. 800,000 ÷ 4 = 200,000 reads.
2. 1 in 1,000 (accuracy 99.9%).
3. FASTA for the reference genome; VCF for the differences.
4. 55, because BED counts from 0.
5. At position 1000 of the sequence `NC_002549.1`, the reference has a G but the sample has a T.

## What comes next

[Lesson 1.5: Biological databases](Lesson_5_Biological_Databases.md)
