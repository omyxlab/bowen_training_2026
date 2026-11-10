# Lesson 4.2: Consensus Genome

**Module 4: Short-Read Analysis II**

## What you will need

- The `bioinfo` environment active
- `~/ebola_project/mapping/SRR1553607.sorted.bam` and its `.bai`
- The `consensus` folder (from Lesson 3.1)

## What you will learn

By the end of this lesson you will be able to:

- Explain what a consensus sequence is and how it differs from a VCF
- Build a consensus genome with `samtools consensus`
- Explain what the `N` characters mean and where they come from
- Change the minimum depth and predict the effect
- Count the `N` characters with `grep`
- Rename the sequence in a FASTA file with `nano`

## Why this matters

The VCF in Lesson 4.1 lists the *differences* from the reference. What scientists often want is the whole **sequence of the virus in the sample**: its own genome. That is the **consensus sequence**: at every position, the base that most of the reads agree on.

A consensus is what you would deposit in a public database, compare with other samples, or use to build a family tree. Our question "what is in this sample?" ends with this sequence.

## Step 1: Build the consensus

```
$ cd ~/ebola_project
$ samtools consensus -a -f fasta mapping/SRR1553607.sorted.bam -o consensus/SRR1553607.consensus.fasta
```

| Part | Meaning |
|---|---|
| `samtools consensus` | Builds a consensus sequence from an alignment |
| `-a` | Output **all** positions, including the ends where there are no reads |
| `-f fasta` | Write it in FASTA format |
| `mapping/SRR1553607.sorted.bam` | The alignment (note: it needs the BAM, not the VCF) |
| `-o consensus/SRR1553607.consensus.fasta` | The output file |

It finishes almost instantly and prints nothing. Check the result:

```
$ ls -l consensus
total 20
-rw-r--r-- 1 yourname yourname 19243 Nov 10 23:25 SRR1553607.consensus.fasta
$ seqkit stats consensus/SRR1553607.consensus.fasta
file                                  format  type  num_seqs  sum_len  min_len  avg_len  max_len
consensus/SRR1553607.consensus.fasta  FASTA   DNA          1   18,959   18,959   18,959   18,959
```

One sequence, **18,959 bases**: exactly the length of the reference. With `-a`, every position of the reference gets a base, even the ones the reads do not cover. What fills those?

## Step 2: Meet the `N`

Look at the start of the file:

```
$ head -n 3 consensus/SRR1553607.consensus.fasta
>NC_002549.1
NNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN
NNNNNNNNNNNNNNNAATNTATANCGGAATTTAAATTNNAATTGTTACTGTAATCATACCTGGTTTGTTT
```

An **`N`** means "**unknown base**". The program writes an `N` wherever the reads do not give enough evidence to decide. The first 85 positions are all `N`, which matches what you found in Lesson 3.4: **nothing covers positions 1 to 85**. There are a few `N`s inside the second line too, where the evidence was thin.

Honest data has gaps, and the `N` is how a consensus admits it. A sequence with `N`s is **not wrong**. It is telling you where to be careful. A sequence that filled the gaps with guesses would be wrong.

### Count the `N`s

You might try `grep -c "N"`, but that counts **lines** containing an `N`, not the `N`s:

```
$ grep -c "N" consensus/SRR1553607.consensus.fasta
6
```

That gives 6, which is not what we want. To count the letters, print each `N` on its own line with `-o` ("only the matching part") and count the lines. `grep -v ">"` first removes the header, which has no `N` here but in general could:

```
$ grep -v ">" consensus/SRR1553607.consensus.fasta | grep -o "N" | wc -l
219
```

**219 of 18,959 bases (about 1.2%) are `N`.** Of those, 195 are the uncovered ends (85 at the start, 110 at the end). The other 24 are positions inside the genome where the evidence was too thin to call.

You can check the end of the file too:

```
$ tail -n 2 consensus/SRR1553607.consensus.fasta
TGATAATTAAGGCATTGACNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN
NNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN
```

The genome ends in a long run of `N`s: the last 110 bases are uncovered.

## Step 3: Be stricter, with a minimum depth

By default, a single read is enough to call a base. That is risky: a single read could be wrong. You can insist on a **minimum depth** with `-d`:

```
$ samtools consensus -a -d 5 -f fasta mapping/SRR1553607.sorted.bam -o consensus/SRR1553607.d5.fasta
$ grep -v ">" consensus/SRR1553607.d5.fasta | grep -o "N" | wc -l
797
```

Now any position with fewer than 5 reads becomes an `N`. The count rises from **219 to 797** (about 4.2% of the genome).

| Setting | `N` characters | What it means |
|---|---|---|
| Default (any depth) | 219 | More sequence, but some bases rest on a single read |
| Minimum depth 5 (`-d 5`) | 797 | Fewer bases, but each backed by at least 5 reads |

This is the central trade-off of consensus building: **completeness against reliability**. There is no single right answer. A rule of thumb is that anything you would publish or use to compare strains should be backed by enough reads; many laboratories use a minimum depth of 10 or more for that. What matters is that you choose a threshold deliberately and write it down.

(In our run the two files have the same length, 18,959, because `-a` gives every position of the reference a base or an `N`. If you try other depths, the length can occasionally change by a base or two, because the consensus also records insertions that the reads support.)

For the rest of the course, we use the **default consensus**.

## Step 4: Give the sequence a proper name

Look at the header line:

```
$ grep ">" consensus/SRR1553607.consensus.fasta
>NC_002549.1
```

The sequence is called `NC_002549.1`, the name of the **reference**, not of your sample. If this file ever got mixed with the reference, no one could tell them apart. Rename it with `nano`.

```
$ nano consensus/SRR1553607.consensus.fasta
```

1. The cursor is at the start of the first line. Use the **arrow keys** to move to the end of the first line (`>NC_002549.1`).
2. Press **Backspace** until only `>` is left.
3. Type `SRR1553607_consensus` so the line reads `>SRR1553607_consensus`.
4. Press **Ctrl+O**, then **Enter** to save.
5. Press **Ctrl+X** to leave.

Check it:

```
$ grep ">" consensus/SRR1553607.consensus.fasta
>SRR1553607_consensus
$ head -n 2 consensus/SRR1553607.consensus.fasta
>SRR1553607_consensus
NNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNNN
```

Be careful in `nano`: only the **header line** should change. Do not touch the lines of sequence, or you change the genome. If you make a mistake, press **Ctrl+X** and answer `N` to leave without saving, then start again.

Now keep a copy with your final results:

```
$ cp consensus/SRR1553607.consensus.fasta results/
$ ls results
SRR1553607.consensus.fasta
```

## What do we have?

You now have the **Ebola virus genome of your sample**: 18,959 bases, with 219 unknown bases (1.2%), mostly at the two ends. In Lesson 4.3 you will find out which known viruses it is closest to.

## If something looks different

| What you see | What it means | What to do |
|---|---|---|
| `samtools: command not found` | The environment is not active | `conda activate bioinfo` |
| `[E::hts_open_format] Failed to open file` | The BAM path is wrong | Check `ls mapping` and use Tab |
| The file is shorter than 18,959 | You left out `-a` | Add `-a` |
| A slightly different number of `N`s | A different samtools version | Small differences are fine (we saw 217 with an older version) |
| `grep -c "N"` gives a small number | It counts lines, not letters | Use `grep -v ">" file \| grep -o "N" \| wc -l` |
| `nano` shows `Error writing` | You do not have permission, or the folder is missing | Check `ls consensus` and that you own the file |
| The sequence lines changed in `nano` | You edited the wrong line | Leave without saving (Ctrl+X, then `N`) and redo the consensus |

## Practice check

1. What is the difference between a VCF and a consensus sequence?
2. What does `N` mean, and is a sequence containing `N`s wrong?
3. Why does `grep -c "N"` not give the number of `N` characters?
4. Why did the number of `N`s go up when we set `-d 5`?
5. Why rename the header?

### Answers

1. A VCF lists only the positions where the sample differs from the reference. A consensus is the sample's complete sequence.
2. `N` means an unknown base, where the reads did not give enough evidence. A sequence with `N`s is not wrong; it honestly marks where information is missing.
3. `grep -c` counts the **lines** that contain an `N`, not how many `N` characters there are. `grep -o "N" | wc -l` counts the characters.
4. Because positions covered by fewer than 5 reads are now written as `N` instead of being called from a few reads.
5. The default name is the reference's, which would make the sample's genome look like the reference.

## Summary of commands

| Command | What it does |
|---|---|
| `samtools consensus -a -f fasta file.bam -o out.fasta` | Builds a consensus sequence |
| `samtools consensus -a -d 5 ...` | Writes `N` where fewer than 5 reads cover a position |
| `grep ">" file.fasta` | Shows the header line |
| `grep -v ">" file.fasta \| grep -o "N" \| wc -l` | Counts the `N` characters |
| `nano file` | Edits a file (Ctrl+O saves, Ctrl+X exits) |
| `cp file results/` | Copies a file into the results folder |

## What comes next

[Lesson 4.3: Interpretation: genes and identity](Lesson_3_Interpretation_Genes_and_Identity.md)
