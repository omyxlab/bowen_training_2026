# Lesson 5.3: Classification with Kraken2

**Module 5: Long-Read Analysis, the Cassava Virus Example**

## What you will need

- The `bioinfo` environment active
- `~/cassava_project/clean/barcode58_nonhost.fastq` and `barcode57_nonhost.fastq` (Lesson 5.2)
- `~/Training/long_reads/sample3.fastq`
- The **Kraken2 viral database**, from your trainers (see Step 1)

## What you will learn

By the end of this lesson you will be able to:

- Explain what taxonomic classification is and how Kraken2 does it
- Set up a Kraken2 database and run a classification
- Read a Kraken2 summary and its report
- Pick out the species rows with `grep -w`
- Explain why most reads are unclassified
- Be appropriately cautious about taxa supported by only a few reads

## Why this matters

After host removal, we are left with reads that are not cassava. But what are they? **Classification** answers this by comparing every read with a database of known genomes and assigning it to the organism it most resembles.

**Kraken2** does this very fast. It breaks each read into short pieces (*k-mers*, 35 bases long here) and looks each piece up in a database that records which organisms contain it. The read is given to the organism its pieces point to most strongly. There is no alignment, so it takes seconds, even for millions of reads.

A classifier can only name what is in its database. We use a **viral** database (the NCBI RefSeq viral genomes), so Kraken2 can recognise viruses and nothing else. Any read from another source, such as a bacterium, will come out as "unclassified". Keep that in mind when reading the results.

## Step 1: Get the Kraken2 database

The database is large: about **590 MB** to download and about **1.2 GB** once unpacked. As with the cassava genome, **do not download it on the course network.** Your trainers will give it to you.

Put it in a folder called `databases` in your home folder. If you receive it as a ready-made folder, copy it so you have `~/databases/kraken2_viral`. If you receive the compressed file `k2_viral_20261022.tar.gz`, unpack it:

```
$ mkdir -p ~/databases/kraken2_viral
$ tar -xzf k2_viral_20261022.tar.gz -C ~/databases/kraken2_viral
```

(`tar -xzf` unpacks a compressed archive. `-C` says which folder to unpack into.) On the JupyterHub the database is already in place.

If you are at home on a good connection, you can download it yourself. The file should be **589,941,425 bytes**:

```
$ mkdir -p ~/databases && cd ~/databases
$ wget https://genome-idx.s3.amazonaws.com/kraken/k2_viral_20261022.tar.gz
$ ls -l
$ mkdir -p kraken2_viral
$ tar -xzf k2_viral_20261022.tar.gz -C kraken2_viral
$ cd ~/cassava_project
```

That is the release available when this course was written. Newer ones are listed at <https://benlangmead.github.io/aws-indexes/k2>. A different release can give slightly different numbers.

Check that the database is in place. Kraken2 needs three files, `hash.k2d`, `opts.k2d` and `taxo.k2d`:

```
$ ls -lh ~/databases/kraken2_viral | head -n 20
```

You should see these three among the files, with `hash.k2d` about 634 MB.

## Step 2: Classify

```
$ cd ~/cassava_project
$ kraken2 --db ~/databases/kraken2_viral --threads 4 --report kraken/barcode58.report --output kraken/barcode58.kraken clean/barcode58_nonhost.fastq
Loading database information... done.
8472 sequences (12.63 Mbp) processed in 1.077s (472.2 Kseq/m, 703.81 Mbp/m).
  100 sequences classified (1.18%)
  8372 sequences unclassified (98.82%)
```

| Part | Meaning |
|---|---|
| `--db ~/databases/kraken2_viral` | The database to use |
| `--threads 4` | Use 4 processor cores |
| `--report kraken/barcode58.report` | Write a **summary report** here (what we will read) |
| `--output kraken/barcode58.kraken` | Write the **per-read results** here |
| `clean/barcode58_nonhost.fastq` | The reads to classify |

It takes **about a second**. The summary on screen already tells the story:

- **100 reads (1.18%) were classified**
- **8,372 reads (98.82%) were not**

## Step 3: The per-read results

```
$ head -n 3 kraken/barcode58.kraken
U	e6485134-01fd-4a74-90a4-c696b4927296	0	641	0:607
U	8fec4baf-5f64-4984-9f4e-6319db0a61c5	0	9925	0:9891
U	554f4739-092b-4b72-89db-8a89bcf1f22e	0	1598	0:1564
```

One line per read: `U` (unclassified) or `C` (classified), the read name, the organism's taxonomy number (0 if none), the read length, and details of how its k-mers matched. All three of these reads are unclassified (`U`). You rarely need this file; the report below is far more useful.

## Step 4: The report

```
$ head -n 14 kraken/barcode58.report
 98.82	8372	8372	U	0	unclassified
  1.18	100	0	R	1	root
  1.18	100	0	R1	10239	  Viruses
  0.97	82	0	R2	2731342	    Floreoviria
  0.97	82	0	K	2732092	      Shotokuvirae
  0.97	82	0	P	2732416	        Cressdnaviricota
  0.97	82	0	C	2732424	          Repensiviricetes
  0.97	82	0	O	2732539	            Geplafuvirales
  0.96	81	0	F	10811	              Geminiviridae
  0.96	81	0	G	10814	                Begomovirus
  0.65	55	55	S	2560285	                  Begomovirus manihotisburkinafasoense
  0.31	26	26	S	10817	                  Begomovirus manihotis
  0.01	1	0	F	1910928	              Genomoviridae
  0.01	1	0	G	1542744	                Gemycircularvirus
```

Each line has six columns:

| Column | Meaning |
|---|---|
| 1 | The **percentage** of all reads in this group |
| 2 | The number of reads in this group **and everything below it** |
| 3 | The number of reads assigned **directly** to this exact group |
| 4 | The **rank**: `U` unclassified, `R` root, `K` kingdom, `P` phylum, `C` class, `O` order, `F` family, `G` genus, `S` species |
| 5 | The taxonomy number |
| 6 | The name, **indented** to show where it sits in the family tree |

The indentation is the tree. Read it as a family tree, from the root down: *Viruses* contain *Floreoviria*, which contain *Shotokuvirae*, and so on down through the family *Geminiviridae*, the genus *Begomovirus*, and finally two species.

### What the report says

- **98.82% of the reads are unclassified.**
- **Everything classified is a virus** (100 reads, all under *Viruses*), because that is all the database contains.
- The largest group is the family **Geminiviridae**, genus ***Begomovirus***, with **81 reads**. Begomoviruses are the group of plant viruses that includes the **cassava mosaic viruses**, which cause one of the most serious diseases of cassava in Africa. That is exactly what we hoped to find.
- Two species are named:
  - ***Begomovirus manihotisburkinafasoense***: 55 reads. This is *African cassava mosaic Burkina Faso virus*.
  - ***Begomovirus manihotis***: 26 reads. This is *African cassava mosaic virus*.
- A single read goes to a different virus family, *Genomoviridae*.

## Step 5: Pick out the species

Reports can be long. Because the rank is in its own column, you can pull out only the **species** rows with `grep -w` (`-w` means "match whole words only", so it finds the column that is exactly `S`):

```
$ grep -w "S" kraken/barcode58.report
  0.65	55	55	S	2560285	                  Begomovirus manihotisburkinafasoense
  0.31	26	26	S	10817	                  Begomovirus manihotis
  0.01	1	1	S	1983771	                    Gemycircularvirus sp.
  0.21	18	18	S	67082	                      BeAn 58058 virus
```

Four species, with 55, 26, 1 and 18 reads. And the genera with `-w "G"`:

```
$ grep -w "G" kraken/barcode58.report
  0.96	81	0	G	10814	                Begomovirus
  0.01	1	0	G	1542744	                Gemycircularvirus
  0.21	18	0	G	2733297	                  Oryzopoxvirus
```

Notice something odd: **BeAn 58058 virus**, with 18 reads, belongs to the genus *Oryzopoxvirus*, which is a group of **poxviruses**. These infect animals, not plants. A cassava sample is not where we would expect one. It could be a misassignment, where reads are given to a similar sequence in the database by mistake, or contamination. We cannot tell from this alone. The cassava mosaic viruses are the plausible finding here; this one is not. This is a good reminder that **classification is a lead, not a proof.** Unexpected results deserve a second look, and are best confirmed by mapping the reads to a reference of that virus.

## Step 6: The other samples

Do the same for barcode57 and for `sample3`. For `sample3` we use the original reads, because it has not been through host removal.

```
$ kraken2 --db ~/databases/kraken2_viral --threads 4 --report kraken/barcode57.report --output kraken/barcode57.kraken clean/barcode57_nonhost.fastq
Loading database information... done.
7997 sequences (11.42 Mbp) processed in 0.903s (531.5 Kseq/m, 758.96 Mbp/m).
  93 sequences classified (1.16%)
  7904 sequences unclassified (98.84%)
$ kraken2 --db ~/databases/kraken2_viral --threads 4 --report kraken/sample3.report --output kraken/sample3.kraken ~/Training/long_reads/sample3.fastq
Loading database information... done.
2970 sequences (2.45 Mbp) processed in 0.457s (390.1 Kseq/m, 321.14 Mbp/m).
  2586 sequences classified (87.07%)
  384 sequences unclassified (12.93%)
```

**barcode57** looks like barcode58: about 1% of reads classified. Look at its species:

```
$ grep -w "S" kraken/barcode57.report
  0.33	26	26	S	2560285	                  Begomovirus manihotisburkinafasoense
  0.25	20	20	S	10817	                  Begomovirus manihotis
  0.16	13	0	S	2955762	                    Diorhovirus intestinalis
  0.03	2	0	S	3430982	                Myrnavirus phabba
  0.01	1	0	S	2846070	                  Plymouthvirus RPP1
  ...
```

(The list continues for another dozen rows.) The same two begomovirus species lead, with 26 and 20 reads. After them, you see a **long tail of species with only one or two reads**, from many unrelated groups of viruses. **Reads supported by one or two matches are very likely noise.** A sensible rule is to give weight only to taxa with a real number of reads, and to treat the tail as background.

**sample3** is completely different:

```
$ grep -w "S" kraken/sample3.report
 87.07	2586	2586	S	1983771	                    Gemycircularvirus sp.
```

**87% of the reads, 2,586 of 2,970, go to a single species**: *Gemycircularvirus sp.*, from the family *Genomoviridae*. Those are small circular DNA viruses, a different group from the begomoviruses. This explains something: **`sample3` contains no cassava mosaic virus**, which is why its reads did not map to the cassava mosaic references when we tested. It is a different virus, from a different sample.

## What we have learned

| Sample | Reads classified | What they are |
|---|---|---|
| barcode58 (after host removal) | 1.18% (100 of 8,472) | Mostly begomoviruses (the cassava mosaic type): 55 + 26 reads |
| barcode57 (after host removal) | 1.16% (93 of 7,997) | The same two begomovirus species (26 + 20 reads), plus a noisy tail |
| sample3 (host not removed) | 87.07% (2,586 of 2,970) | One gemycircularvirus |

### Why is most of the data unclassified?

98 to 99% of the reads in barcode57 and barcode58 are unclassified. Is that a problem? No, for three reasons:

- **The database only knows viruses.** A read from a bacterium, a fungus, or any other organism can only come out "unclassified". We removed the plant; we did not remove everything else.
- **Nanopore reads have errors.** An error in a read breaks the k-mers that overlap it. Long reads with lower accuracy give fewer exact k-mer matches than accurate short reads do.
- **Viruses are rare in the sample.** Most of the DNA in a plant extract is not viral.

What the result does tell us is that the viral part of the sample is dominated by **begomoviruses of the cassava mosaic type**. Classification finds what is in the database; it does not find everything that is there.

## If something looks different

| What you see | What it means | What to do |
|---|---|---|
| `kraken2: command not found` | The environment is not active | `conda activate bioinfo` |
| `kraken2: database ("...") does not contain necessary file taxo.k2d` | The database path is wrong, or it is incomplete | Check `ls ~/databases/kraken2_viral` for `hash.k2d`, `opts.k2d`, `taxo.k2d` |
| `Loading database information...` then nothing | It is loading about 640 MB; wait a few seconds | Wait |
| `Killed` or out of memory | Too little memory | The viral database needs about 1 GB; close other programs |
| Everything unclassified | The wrong database, or the wrong reads | Check the database name and that the FASTQ is not empty |
| Different numbers of reads | A different database release or Kraken2 version | Small differences are expected |
| `No such file or directory` for the reads | You are in the wrong folder | `cd ~/cassava_project`, and use Tab |

## Practice check

1. What can a Kraken2 database not do?
2. In the report, what do the second and third columns mean?
3. How many reads in barcode58 were classified as *Begomovirus*, and why do we care?
4. Why is BeAn 58058 virus a result to treat with caution?
5. What did Kraken2 tell us about `sample3`?
6. Why should you distrust species that have only one or two reads?

### Answers

1. Recognise anything that is not in it. Reads from organisms outside the database can only be "unclassified".
2. The second is the number of reads in the group and everything below it. The third is the number assigned directly to that exact group.
3. 81 reads. Begomoviruses include the cassava mosaic viruses, the plant viruses we were looking for.
4. It is a poxvirus, a group not expected in a plant sample, and only 18 reads support it. It could be a misassignment or contamination, so it needs confirming.
5. 87% of its reads are a single species of gemycircularvirus, so it is a different virus from the cassava mosaic viruses in barcode57 and barcode58.
6. Matches by chance, sequencing errors and similar sequences in the database can produce a few stray assignments. Real viruses tend to attract many reads.

## Summary of commands

| Command | What it does |
|---|---|
| `kraken2 --db DB --report R --output O reads.fastq` | Classifies reads and writes a report |
| `head -n 14 report` | Looks at the top of the report |
| `grep -w "S" report` | Lists the species rows |
| `grep -w "G" report` | Lists the genus rows |
| `tar -xzf file.tar.gz -C folder` | Unpacks a compressed archive into a folder |

## What comes next

[Lesson 5.4: Krona plots](Lesson_4_Krona_Plots.md)
