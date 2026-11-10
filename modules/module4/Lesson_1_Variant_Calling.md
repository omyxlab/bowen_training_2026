# Lesson 4.1: Variant Calling

**Module 4: Short-Read Analysis II**

## What you will need

- The `bioinfo` environment active
- `~/ebola_project/mapping/SRR1553607.sorted.bam` and its `.bai` (Module 3)
- `~/ebola_project/reference/NC_002549.1.fasta`
- The `variants` folder (from Lesson 3.1)

## What you will learn

By the end of this lesson you will be able to:

- Explain what a variant is, and the difference between a SNP and an indel
- Describe, in plain words, what variant calling does
- Run `bcftools mpileup` and `bcftools call` as one pipeline
- Read the columns of a VCF file
- Count variants with `grep` and `bcftools`
- Filter out variants supported by weak evidence

## Why this matters

Our virus is not identical to the reference genome. The places where they differ are **variants**, and they are what make one virus sample different from another. Variants let scientists tell outbreak strains apart, follow how a virus changes, and group related samples.

| Term | Meaning |
|---|---|
| **Variant** | A position where the sample differs from the reference |
| **SNP** | A single-nucleotide polymorphism: one base replaced by another (for example A becomes C) |
| **Indel** | An insertion or deletion of one or more bases |

## What is variant calling?

You already did the hard conceptual part. In Lesson 3.4 you looked at `tview` and saw that at position 295 **every** read said `C` while the reference said `A`. You recognised a variant by eye.

**Variant calling** is that same decision, made automatically for every position in the genome:

1. For each position, **gather what all the reads say** at that spot (this step is called a *pileup*: the stack of reads you saw in `tview`).
2. **Decide**, using how many reads agree, how good their base qualities are, and how confidently they were mapped, whether the sample really differs from the reference.

That is all. There are two programs, one for each step, and you join them with a pipe.

## Step 1: Call the variants

```
$ cd ~/ebola_project
$ bcftools mpileup -f reference/NC_002549.1.fasta mapping/SRR1553607.sorted.bam | bcftools call -mv --ploidy 1 -o variants/SRR1553607.vcf
[mpileup] 1 samples in 1 input files
[mpileup] maximum number of reads per input file set to -d 250
```

It takes less than a second. Read it as one command in two halves, joined by `|`:

| Part | Meaning |
|---|---|
| `bcftools mpileup` | Step 1: gather the reads at every position |
| `-f reference/NC_002549.1.fasta` | The reference to compare against |
| `mapping/SRR1553607.sorted.bam` | The alignment |
| `\|` | Pass the pileup straight to the next program |
| `bcftools call` | Step 2: decide which positions are variants |
| `-m` | Use the standard calling method |
| `-v` | Write only the variant positions (skip positions that match the reference) |
| `--ploidy 1` | Treat the genome as having **one** copy. Viruses like Ebola do, unlike human cells, which have two |
| `-o variants/SRR1553607.vcf` | Write the results to this file |

The two `[mpileup]` lines are just information. Check the file exists:

```
$ ls -lh variants
total 72K
-rw-r--r-- 1 yourname yourname 71K Nov 10 23:24 SRR1553607.vcf
```

## Step 2: Read the VCF

The output is a **VCF** file (Lesson 1.4). It is plain text, so you already know how to read it. First, the header lines, which start with `##`:

```
$ grep -c "^##" variants/SRR1553607.vcf
28
```

28 lines of description. (They explain every abbreviation used below, and record the commands that made the file.) Look at the first few:

```
$ grep "^##" variants/SRR1553607.vcf | head -n 6
##fileformat=VCFv4.2
##FILTER=<ID=PASS,Description="All filters passed">
##bcftoolsVersion=1.25+htslib-1.25
##bcftoolsCommand=mpileup -f reference/NC_002549.1.fasta mapping/SRR1553607.sorted.bam
##reference=file://reference/NC_002549.1.fasta
##contig=<ID=NC_002549.1,length=18959>
```

(Your version numbers may be newer.) Next, the single line that names the columns:

```
$ grep "^#CHROM" variants/SRR1553607.vcf
#CHROM	POS	ID	REF	ALT	QUAL	FILTER	INFO	FORMAT	mapping/SRR1553607.sorted.bam
```

And now the variants themselves: every line that does **not** start with `#`.

```
$ grep -v "^#" variants/SRR1553607.vcf | head -n 5
NC_002549.1	108	.	G	T	37.4152	.	DP=2;VDB=0.02;SGB=-0.453602;MQ0F=0;AC=1;AN=1;DP4=0,0,2,0;MQ=60	GT:PL:AD	1:67,0:0,2
NC_002549.1	109	.	A	C	37.4152	.	DP=2;VDB=0.02;SGB=-0.453602;MQ0F=0;AC=1;AN=1;DP4=0,0,2,0;MQ=60	GT:PL:AD	1:67,0:0,2
NC_002549.1	127	.	C	T	67.4148	.	DP=3;VDB=0.131948;SGB=-0.511536;MQ0F=0;AC=1;AN=1;DP4=0,0,3,0;MQ=60	GT:PL:AD	1:97,0:0,3
NC_002549.1	149	.	C	T	109.415	.	DP=5;VDB=0.176409;SGB=-0.556411;MQ0F=0;AC=1;AN=1;DP4=0,0,4,0;MQ=60	GT:PL:AD	1:139,0:0,4
NC_002549.1	155	.	A	C	113.415	.	DP=6;VDB=0.284843;SGB=-0.590765;MQ0F=0;AC=1;AN=1;DP4=0,0,5,0;MQ=60	GT:PL:AD	1:143,0:0,5
```

(`bcftools view -H variants/SRR1553607.vcf` does the same as `grep -v "^#"`: `-H` means "no header".)

That is a lot of text. For a friendlier view of the columns that matter, use `bcftools query` with a template. `%POS`, `%REF`, `%ALT`, `%QUAL` and `%DP` are the fields, and `\n` ends each line:

```
$ bcftools query -f "%POS %REF %ALT %QUAL %DP\n" variants/SRR1553607.vcf | head -n 6
108 G T 37.4152 2
109 A C 37.4152 2
127 C T 67.4148 3
149 C T 109.415 5
155 A C 113.415 6
182 A G 131.416 7
```

### What each column means

| Column | Example | Meaning |
|---|---|---|
| `CHROM` | `NC_002549.1` | The reference sequence |
| `POS` | `108` | The position of the variant |
| `ID` | `.` | An identifier (none here) |
| `REF` | `G` | The base in the **reference** |
| `ALT` | `T` | The base found in the **sample** |
| `QUAL` | `37.4` | How confident the program is that this variant is real (higher is better) |
| `FILTER` | `.` | Whether the variant failed a filter (none applied yet) |
| `INFO` | `DP=2;...` | Extra details, explained next |
| `FORMAT`, last column | `GT:PL:AD` and `1:67,0:0,2` | Details about this sample |

Inside the `INFO` column, three entries are worth knowing:

| Field | Meaning |
|---|---|
| `DP` | **Depth**: how many reads covered the position |
| `DP4` | Four numbers: reads supporting the **reference** (forward, reverse), then reads supporting the **variant** (forward, reverse) |
| `MQ` | Average mapping quality (60 is the maximum) |

And in the last column, `AD` (for example `0,2`) gives the reads for the reference base and then the variant base.

### Reading one line properly

Here is the variant you saw in `tview`:

```
$ bcftools view -H -t NC_002549.1:295 variants/SRR1553607.vcf
NC_002549.1	295	.	A	C	225.417	.	DP=19;VDB=0.239262;SGB=-0.691153;MQSBZ=0;MQ0F=0;AC=1;AN=1;DP4=0,0,13,5;MQ=60	GT:PL:AD	1:255,0:0,18
```

(`-t NC_002549.1:295` asks for the variants at that one position.) In words:

> *At position 295, the reference has an **A** and the sample has a **C**. 19 reads covered the position; **none** supports the reference (`DP4=0,0,...`), and **18** support the variant (13 on the forward strand and 5 on the reverse). The caller's confidence is high (QUAL 225).*

That agrees exactly with what you saw by eye in `tview`.

## Step 3: Count the variants

```
$ grep -vc "^#" variants/SRR1553607.vcf
559
```

`-v` inverts the match (lines that do **not** match) and `-c` counts them: **559 variants**. How many are SNPs and how many are indels? Indel lines contain the word `INDEL`. Search the variant lines only:

```
$ grep -v "^#" variants/SRR1553607.vcf | grep -c "INDEL"
0
```

> **A trap:** if you run `grep -c "INDEL" variants/SRR1553607.vcf` on the *whole file* you get `1`, which is wrong. The header contains a line that mentions `INDEL` while describing the format. Remove the header lines first, as above.

`bcftools stats` gives a ready-made summary:

```
$ bcftools stats variants/SRR1553607.vcf | grep "^SN"
SN	0	number of samples:	1
SN	0	number of records:	559
SN	0	number of no-ALTs:	0
SN	0	number of SNPs:	559
SN	0	number of MNPs:	0
SN	0	number of indels:	0
SN	0	number of others:	0
SN	0	number of multiallelic sites:	0
SN	0	number of multiallelic SNP sites:	0
```

All **559 are single-base changes (SNPs)**, with no indels. That is about 3% of the 18,959-base genome. It is a lot, and we will come back to why.

## Step 4: Filter out the weak variants

Look again at the first variants in the list: positions 108 and 109 have a depth of **2**, and their QUAL is only 37. Two reads is thin evidence, and these are near the start of the genome where coverage is poor (Lesson 3.4). Some of these may be errors.

We can keep only variants with good support. `bcftools view -i` keeps the lines that satisfy a condition:

```
$ bcftools view -i "QUAL>=30 && INFO/DP>=10" variants/SRR1553607.vcf -o variants/SRR1553607.filtered.vcf
```

| Part | Meaning |
|---|---|
| `-i` | **Include** only variants meeting the condition |
| `QUAL>=30` | The caller's confidence must be at least 30 |
| `&&` | ...and |
| `INFO/DP>=10` | ...at least 10 reads must cover the position |
| `-o` | Write the result to this file |

Count what remains:

```
$ grep -vc "^#" variants/SRR1553607.filtered.vcf
394
$ bcftools view -H variants/SRR1553607.filtered.vcf | head -n 3
NC_002549.1	230	.	C	T	225.417	.	DP=13;VDB=0.892818;SGB=-0.676189;MQSBZ=0;MQ0F=0;AC=1;AN=1;DP4=0,0,8,3;MQ=60	GT:PL:AD	1:255,0:0,11
NC_002549.1	236	.	T	C	225.417	.	DP=17;VDB=0.866636;SGB=-0.688148;MQSBZ=0.5;MQ0F=0;AC=1;AN=1;DP4=0,0,12,3;MQ=59.3333	GT:PL:AD	1:255,0:0,15
NC_002549.1	257	.	A	G	225.417	.	DP=16;VDB=0.474283;SGB=-0.683931;MQSBZ=0.426401;MQ0F=0;AC=1;AN=1;DP4=0,0,11,2;MQ=59.2308	GT:PL:AD	1:255,0:0,13
```

**394 variants remain**, out of 559. The 165 we dropped were mostly weak: for example, 160 of the original 559 had a depth between 2 and 9. Notice that the first one left is at position 230, past the poorly covered start of the genome. **We use this filtered file for the rest of the course.**

The filter thresholds are a judgement, not a rule. Stricter ones give fewer, more certain variants. Looser ones give more, with more false positives. What matters is that you state which you used.

## Why so many variants? A note on the reference

559 differences from the reference is a lot, and nearly every one has **every read agreeing**: they are real. The reason is the reference. Remember from Lesson 1.5: `NC_002549.1` is an isolate from **1976**, from the Democratic Republic of the Congo. Our sample is from **2014**, from Sierra Leone. About four decades of evolution, and a different part of Africa, separate them, so the two genomes differ at hundreds of positions.

This is a property of the **reference we chose**, not of the sample. A reference closer to the sample would show far fewer differences. You will test that for yourself in the capstone.

Among the 394 filtered variants, 333 are *transitions* (A to G or C to T, and the reverse) and 61 are *transversions* (every other kind of change). Transitions are the more common kind of change in real genomes, so a high proportion of them is another sign that these variants are real. You can see it with `bcftools stats variants/SRR1553607.filtered.vcf | grep "^TSTV"`.

## If something looks different

| What you see | What it means | What to do |
|---|---|---|
| `[E::faidx_...] Could not build fai index` or `reference not found` | The path to the FASTA is wrong | Check `ls reference` and use Tab |
| `Error: no BAM/CRAM index` | The BAM is not indexed | `samtools index mapping/SRR1553607.sorted.bam` |
| `bcftools: command not found` | The environment is not active | `conda activate bioinfo` |
| `variants/SRR1553607.vcf: No such file or directory` | The `variants` folder does not exist | `mkdir -p variants` and repeat |
| A different number of variants | A different tool version, or different trimming | Small differences are fine; it should be near 559 |
| `grep -c "INDEL"` gives 1 | The header line matches | Use `grep -v "^#" file \| grep -c "INDEL"` |
| `[E::filter_init] ...` | A typo in the filter expression | Use straight quotes and check `QUAL>=30 && INFO/DP>=10` |

## Practice check

1. What is the difference between a SNP and an indel?
2. In the pipeline, what does `bcftools mpileup` do, and what does `bcftools call` do?
3. Why do we use `--ploidy 1`?
4. What does `DP4=0,0,13,5` tell you?
5. Why do we filter, and how many variants are left after the filter we used?
6. A friend reports "559 variants, so this virus is very mutated". What is wrong with that statement?

### Answers

1. A SNP is a single base replaced by another. An indel is an insertion or deletion of one or more bases.
2. `mpileup` gathers what all the reads say at each position. `call` decides which positions are real variants.
3. The Ebola virus genome exists in one copy per virus particle, not two, so we tell the program to expect one allele.
4. No reads support the reference base, and 18 support the variant (13 forward, 5 reverse).
5. To remove variants supported by weak evidence, such as very low depth. 394 remain.
6. The number depends on the reference. Against a distant 1976 reference there are many differences; against a closer reference there are far fewer. It does not mean the sample is unusually mutated.

## Summary of commands

| Command | What it does |
|---|---|
| `bcftools mpileup -f ref.fa file.bam \| bcftools call -mv --ploidy 1 -o out.vcf` | Finds the variants |
| `grep -c "^##" file.vcf` | Counts header lines |
| `grep "^#CHROM" file.vcf` | Shows the column names |
| `grep -v "^#" file.vcf` | Shows only the variants |
| `grep -vc "^#" file.vcf` | Counts the variants |
| `bcftools view -H file.vcf` | Shows the variants (no header) |
| `bcftools view -H -t CHR:POS file.vcf` | Shows the variants at one position or region |
| `bcftools query -f "%POS %REF %ALT\n" file.vcf` | Prints chosen columns |
| `bcftools stats file.vcf \| grep "^SN"` | A summary of the variants |
| `bcftools view -i "QUAL>=30 && INFO/DP>=10" in.vcf -o out.vcf` | Keeps only well-supported variants |

## What comes next

[Lesson 4.2: Consensus genome](Lesson_2_Consensus_Genome.md)
