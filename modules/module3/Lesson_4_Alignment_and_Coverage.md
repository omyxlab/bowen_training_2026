# Lesson 3.4: Alignment and Coverage

**Module 3: Short-Read Analysis I**

## What you will need

- The `bioinfo` environment active
- `~/ebola_project/mapping/SRR1553607.sorted.bam` and its index `.bai` (Lesson 3.3)
- `~/ebola_project/reference/NC_002549.1.fasta`

## What you will learn

By the end of this lesson you will be able to:

- Explain **coverage** and **depth**, and why both matter
- Measure them with `samtools coverage` and `samtools depth`
- Say which parts of the genome are well covered and which are not
- Look at the reads stacked on the reference with `samtools tview`
- Recognise, in that view, the places where the sample differs from the reference

## Why this matters

Mapping tells us *that* reads match the virus. It does not tell us whether we have *enough* of them, in the right places, to trust what we read off the genome. Two ideas answer that.

- **Coverage** (also called *breadth*): what **fraction** of the genome has at least one read over it.
- **Depth**: how many reads cover each position, on average. "17x" means each base is covered by about 17 reads.

A position seen by one read is a guess: that read could contain an error. A position seen by 20 reads agreeing is solid. Before looking for differences between our sample and the reference, we check that the evidence is good enough.

## Step 1: Summarise with `samtools coverage`

```
$ cd ~/ebola_project
$ samtools coverage mapping/SRR1553607.sorted.bam
#rname	startpos	endpos	numreads	covbases	coverage	meandepth	meanbaseq	meanmapq
NC_002549.1	1	18959	4135	18764	98.9715	17.1959	32.9	59.8
```

One row per reference sequence. Reading across:

| Column | Value | Meaning |
|---|---|---|
| `#rname` | NC_002549.1 | The reference sequence |
| `startpos`, `endpos` | 1, 18959 | The whole genome |
| `numreads` | 4,135 | Alignment records on the reference |
| `covbases` | 18,764 | Bases covered by at least one read |
| `coverage` | 98.97 | The percentage of the genome covered |
| `meandepth` | 17.2 | The average depth: each base is covered, on average, by 17 reads |
| `meanbaseq` | 32.9 | The average quality of the mapped bases |
| `meanmapq` | 59.8 | The average mapping confidence (the maximum is 60) |

So from about 4,000 reads in a sample that was 98.8% something else, we recover **99% of the Ebola genome at 17x depth**, with high quality (Q33) and highly confident mappings (59.8 out of 60). That is a good result.

## Step 2: A picture in the terminal

Add `-m` to draw a histogram of coverage along the genome:

```
$ samtools coverage -m mapping/SRR1553607.sorted.bam
NC_002549.1 (19.0Kbp)
>  90.00% │ ██████████████████████████████████████ │ Number of reads: 4135
>  80.00% │▂██████████████████████████████████████▄│     (46 filtered)
>  70.00% │████████████████████████████████████████│ Covered bases:   18.8Kbp
>  60.00% │████████████████████████████████████████│ Percent covered: 98.97%
>  50.00% │████████████████████████████████████████│ Mean coverage:   17.2x
>  40.00% │████████████████████████████████████████│ Mean baseQ:      32.9
>  30.00% │████████████████████████████████████████│ Mean mapQ:       59.8
>  20.00% │████████████████████████████████████████│ 
>  10.00% │████████████████████████████████████████│ Histo bin width: 473bp
>   0.00% │████████████████████████████████████████│ Histo max bin:   100%
          1        4.7K      9.5K     14.2K      19.0K  
```

The genome runs left to right. The height of each bar shows how much of that stretch is covered. It is essentially full across the whole genome, with a small dip at each end.

## Step 3: Depth at every position

`samtools depth` prints the depth at each position: the reference name, the position, and the number of reads.

```
$ samtools depth mapping/SRR1553607.sorted.bam | head -n 3
NC_002549.1	86	1
NC_002549.1	87	1
NC_002549.1	88	1
```

By default it only lists positions that have reads, so the **first covered position is 86**. Look at the other end:

```
$ samtools depth mapping/SRR1553607.sorted.bam | tail -n 3
NC_002549.1	18847	2
NC_002549.1	18848	2
NC_002549.1	18849	2
```

The **last covered position is 18,849**. The depth at both ends is only 1 or 2.

```
$ samtools depth mapping/SRR1553607.sorted.bam | wc -l
18764
```

18,764 positions are covered, matching `covbases` above. The genome has 18,959, so **195 bases are not covered**: positions 1 to 85 at the start (85 bases) and 18,850 to 18,959 at the end (110 bases). Those are the extreme ends of the genome. They are hard to cover for technical reasons, and in a sequencing library the very ends of a molecule are often under-represented.

Add `-a` to list **all** positions, including those with no reads:

```
$ samtools depth -a mapping/SRR1553607.sorted.bam | head -n 3
NC_002549.1	1	0
NC_002549.1	2	0
NC_002549.1	3	0
```

Zero depth: nothing is known about those bases. Look at a region in the middle, using `-r` to choose it. This is the start of the **GP** gene:

```
$ samtools depth -a -r NC_002549.1:5900-5905 mapping/SRR1553607.sorted.bam
NC_002549.1	5900	20
NC_002549.1	5901	20
NC_002549.1	5902	21
NC_002549.1	5903	22
NC_002549.1	5904	22
NC_002549.1	5905	22
```

Here we have about 20 reads at every base. That is solid evidence.

## Step 4: Which reads are where? `samtools idxstats` and regions

`idxstats` gives a one-line summary per reference sequence:

```
$ samtools idxstats mapping/SRR1553607.sorted.bam
NC_002549.1	18959	4135	46
*	0	0	333698
```

The columns are: name, length, mapped reads, unmapped reads *placed on that sequence*. The `*` row holds the reads with no position at all: **333,698** of them.

Because the BAM is sorted and indexed, you can count the reads in any region instantly. Compare the reads in two genes:

```
$ samtools view -c mapping/SRR1553607.sorted.bam NC_002549.1:56-3026
716
$ samtools view -c mapping/SRR1553607.sorted.bam NC_002549.1:11501-18282
1526
```

NP (2,971 bases) has 716 reads and L (6,782 bases) has 1,526. L is more than twice as long and has about twice the reads, so the coverage is roughly even per base.

## Step 5: See the reads with `samtools tview`

Numbers are good, but nothing beats looking. `samtools tview` ("text viewer") shows the reads stacked on top of the reference.

**To explore interactively**, give it the BAM file and the reference:

```
$ samtools tview mapping/SRR1553607.sorted.bam reference/NC_002549.1.fasta
```

A text screen opens. Keys that help:

| Key | What it does |
|---|---|
| **g** | Go to a position: type `NC_002549.1:5900` and press Enter |
| **← / →** | Scroll left or right along the genome |
| **↑ / ↓** | Scroll the reads up or down |
| **?** | Help |
| **q** | **Quit** |

**To print a single screenful** (useful for pasting into notes), add `-d T` (display as text) and `-p` for the position. Here is the view at position 285:

```
$ samtools tview -d T -p NC_002549.1:285 mapping/SRR1553607.sorted.bam reference/NC_002549.1.fasta | head -n 9
      291       301       311       321       331       341       351           
ACCTTAATAGAAACATTATTGTTAAAGGACAGCATTAGTTCACAGTCAAACAAGCAAGATTGAGAATTAACCTTGGTTTT
..........C............................................................T...A....
.......,,,c,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,t,,,a,,,,
,,,,,,,,,,c   ...............................         ,,,,,,,,,,,,,,,,,t,,,a,,,,
..........C..  ,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,                    ....
..........C.....      ,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,t,,,a,,,,
..........C.......      ,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,t,,,a,,,,
,,,,,,,,,,c,,,,,,,,       .............................................T...A....
```

How to read this picture:

- **The top line** is a ruler: position numbers along the genome (291, 301, 311, ...).
- **The second line** is the **reference**: the bases of `NC_002549.1`.
- **Every line below** is a read (or several reads in a row), laid out at the position where it mapped.
- A **dot** `.` means the read matches the reference on the forward strand. A **comma** `,` means it matches on the reverse strand.
- A **letter** means the read **differs** from the reference at that position. Uppercase is a forward-strand read, lowercase a reverse-strand read.

Now look at the column under position **295**, the 11th base of this screen (the reference base there is an `A`). Every read that covers it shows a **`C`** or a `c`. **Every single read disagrees with the reference at that position, and they all agree with each other.** That is not a sequencing error: errors would scatter randomly. It is a real difference between this virus and the reference genome.

Two more columns further right (the `T`/`t` and `A`/`a` stacks) show the same thing. These are the **variants** you will find automatically in the next lesson.

Look at the start of the GP gene, too:

```
$ samtools tview -d T -p NC_002549.1:5900 mapping/SRR1553607.sorted.bam reference/NC_002549.1.fasta | head -n 6
 5901      5911      5921      5931      5941      5951      5961               
GATGAAGATTAAGCCGACAGTGAGCGTAATCTTCATCTCTCTTAGATTATTTGTTTTCCAGAGTAGGGGTCGTCAGGTCC
......................................................C................A........
.........................................  ,,,,,,,,,,,c,,,,,,,,,,,,,,,,a,,,,,,,,
.............   ................................       ................A........
.........................................    ,,,,,,,,,c,,,,,,,,,,,,,,,,a,,,,,,,,
```

Again, the reads agree with each other and disagree with the reference at two positions. A single stray letter in one read would be an error; a letter in **every** read is a variant.

## Where did the evidence end?

Putting it together, the alignment has:

- **99% of the genome covered**, from position 86 to 18,849
- **About 17 reads** at a typical position, with solid depth (about 20) across the genes
- **Thin coverage at both ends** (1 to 2 reads), and nothing at all in the first 85 and last 110 bases
- Many **positions where every read differs from the reference**

That is enough to look for variants and to build a consensus, as long as we keep in mind that the ends are less certain.

## If something looks different

| What you see | What it means | What to do |
|---|---|---|
| `[E::hts_idx_load3] Could not load index file` | The BAM has no index | Run `samtools index mapping/SRR1553607.sorted.bam` |
| `samtools coverage` prints nothing | The BAM is empty or the wrong file | Check you used the sorted BAM, and `ls -lh mapping` |
| `samtools tview` shows a blank screen | You are at a position with no reads | Press `g` and enter `NC_002549.1:5900` |
| You cannot get out of `tview` | You are still inside it | Press **q** |
| `samtools depth -r ... ` prints nothing | The region name is wrong | Use `NC_002549.1:start-end` exactly |
| Numbers differ slightly from this lesson | A different tool version or trimming settings | Small differences are fine |

## Practice check

1. What is the difference between coverage and depth?
2. What is the mean depth, and what percentage of the genome is covered?
3. Which positions of the genome have no reads, and why do they matter?
4. In `tview`, every read shows `C` where the reference has `A`. Is this a sequencing error? Explain.
5. How would you count the reads that fall in the GP gene (5900 to 8305)?

### Answers

1. Coverage is the fraction of the genome with at least one read. Depth is how many reads cover a position, on average or at a given point.
2. A mean depth of 17.2x, and 98.97% of the genome covered.
3. Positions 1 to 85 and 18,850 to 18,959 (195 bases). We have no information about them, so they cannot be used to find variants or write the consensus.
4. No. Sequencing errors are random and affect single reads. A change seen in every read that covers a position is a real difference between the sample and the reference.
5. `samtools view -c mapping/SRR1553607.sorted.bam NC_002549.1:5900-8305` (the answer is 606).

## Summary of commands

| Command | What it does |
|---|---|
| `samtools coverage file.bam` | Summary of coverage and depth per sequence |
| `samtools coverage -m file.bam` | The same, with a histogram |
| `samtools depth file.bam` | Depth at each covered position |
| `samtools depth -a file.bam` | Depth at every position, including zero |
| `samtools depth -a -r REGION file.bam` | Depth within a region |
| `samtools idxstats file.bam` | Reads per reference sequence |
| `samtools view -c file.bam REGION` | Counts reads in a region |
| `samtools tview file.bam ref.fasta` | Opens the interactive viewer (**q** to quit) |
| `samtools tview -d T -p POS file.bam ref.fasta` | Prints one screen as text |

## What comes next

You have taken the Ebola reads from raw files to a checked alignment. In **Module 4** you find the differences between this virus and the reference, build the sample's own genome, and work out what it is.

[Module 4: Short-Read Analysis II](../module4/README.md)
