# Lesson 4.3: Interpretation, Genes and Identity

**Module 4: Short-Read Analysis II**

## What you will need

- The `bioinfo` environment active
- `~/ebola_project/variants/SRR1553607.filtered.vcf` (Lesson 4.1)
- `~/ebola_project/reference/NC_002549.1.gff3` (Lesson 3.1)
- `~/ebola_project/consensus/SRR1553607.consensus.fasta`, renamed (Lesson 4.2)
- A web browser

## What you will learn

By the end of this lesson you will be able to:

- Count variants in a chosen region with `bcftools view -t`
- Use the gene annotation to say which genes carry the variants
- Compare genes of different lengths fairly, by variants per kilobase
- Look up a gene's protein in UniProt
- Identify the sample by comparing the consensus genome with the public database using NCBI BLAST
- Say what these results show, and what they do not

## Why this matters

A list of positions and a string of letters are results, but not yet an *answer*. Interpretation connects them to biology. Two questions turn what you have into a conclusion:

1. **Where** in the genome are the differences? Which genes?
2. **What** is this virus? Which known viruses is it closest to?

## Part 1: Which genes carry the variants?

In Lesson 3.1 you found the seven genes in the annotation. Here they are again, with their coordinates:

| Gene | Start | End | Length (bases) |
|---|---|---|---|
| NP | 56 | 3026 | 2,971 |
| VP35 | 3032 | 4407 | 1,376 |
| VP40 | 4390 | 5894 | 1,505 |
| GP | 5900 | 8305 | 2,406 |
| VP30 | 8288 | 9740 | 1,453 |
| VP24 | 9885 | 11518 | 1,634 |
| L | 11501 | 18282 | 6,782 |

(The length is `end - start + 1`. For NP: 3026 − 56 + 1 = 2,971.)

### Count the variants in a region

You used `bcftools view -H -t` in Lesson 4.1 to see variants at a single position. The same option accepts a **region**, written `chromosome:start-end`. Count the variants in the **GP** gene:

```
$ cd ~/ebola_project
$ bcftools view -H -t NC_002549.1:5900-8305 variants/SRR1553607.filtered.vcf | wc -l
62
```

`-H` leaves out the header, so `wc -l` counts only variants. Look at the first few:

```
$ bcftools view -H -t NC_002549.1:5900-8305 variants/SRR1553607.filtered.vcf | head -n 3
NC_002549.1	5954	.	T	C	225.417	.	DP=17;VDB=0.852252;SGB=-0.688148;MQSBZ=0;MQ0F=0;AC=1;AN=1;DP4=0,0,5,10;MQ=60	GT:PL:AD	1:255,0:0,15
NC_002549.1	5971	.	G	A	225.417	.	DP=17;VDB=0.571893;SGB=-0.689466;MQSBZ=0.276026;MQ0F=0;AC=1;AN=1;DP4=0,0,7,9;MQ=58	GT:PL:AD	1:255,0:0,16
NC_002549.1	5988	.	C	T	211.421	.	DP=10;VDB=0.078521;SGB=-0.670168;MQSBZ=0.816497;MQ0F=0;AC=1;AN=1;DP4=0,0,6,4;MQ=60	GT:PL:AD	1:241,0:0,10
```

The first two variants, at 5954 and 5971, are the two columns you saw in `tview` at the start of GP in Lesson 3.4. **GP has 62 variants.**

Now repeat for every gene. Copy and run these seven commands one at a time:

```
bcftools view -H -t NC_002549.1:56-3026 variants/SRR1553607.filtered.vcf | wc -l
bcftools view -H -t NC_002549.1:3032-4407 variants/SRR1553607.filtered.vcf | wc -l
bcftools view -H -t NC_002549.1:4390-5894 variants/SRR1553607.filtered.vcf | wc -l
bcftools view -H -t NC_002549.1:5900-8305 variants/SRR1553607.filtered.vcf | wc -l
bcftools view -H -t NC_002549.1:8288-9740 variants/SRR1553607.filtered.vcf | wc -l
bcftools view -H -t NC_002549.1:9885-11518 variants/SRR1553607.filtered.vcf | wc -l
bcftools view -H -t NC_002549.1:11501-18282 variants/SRR1553607.filtered.vcf | wc -l
```

You should get **74, 17, 31, 62, 24, 40 and 133**, one per gene in the order of the table. Put them together:

| Gene | Length (bases) | Variants | Variants per kilobase |
|---|---|---|---|
| NP | 2,971 | 74 | 24.9 |
| VP35 | 1,376 | 17 | 12.4 |
| VP40 | 1,505 | 31 | 20.6 |
| GP | 2,406 | 62 | 25.8 |
| VP30 | 1,453 | 24 | 16.5 |
| VP24 | 1,634 | 40 | 24.5 |
| L | 6,782 | 133 | 19.6 |

(Variants per kilobase = variants ÷ length × 1,000. For GP: 62 ÷ 2,406 × 1,000 = 25.8.)

### Reading the table

- **L has the most variants (133)**, but that is mainly because it is by far the **longest** gene: 6,782 bases, more than twice the length of any other.
- To compare genes of different sizes fairly, use **variants per kilobase**. By that measure **GP (25.8), NP (24.9) and VP24 (24.5)** have the highest density and **VP35 (12.4)** the lowest.
- These seven genes hold 381 of the 394 variants. The remaining 13 are in the stretches **between** genes, such as the start of the genome, which no gene covers. (Neighbouring genes also overlap by a few bases in the annotation, so a variant in an overlap would be counted twice.)

### What we can and cannot say

The numbers show that the differences from the 1976 reference are **spread across the whole genome**, in every gene, not concentrated in one place. We should be careful not to say more. With 12 to 133 variants per gene, small differences in density can easily arise by chance, so we cannot claim from these counts that any gene is under special pressure. And we have not asked whether each variant **changes the protein**: a change in a gene may or may not alter the amino acid it codes for. Answering that needs a program that reads the annotation and the codons, which is beyond this course.

## Part 2: Look up the proteins in UniProt

Each gene codes for a protein, and UniProt (Lesson 1.5) describes it. Search for these accessions at <https://www.uniprot.org/>:

| Gene | UniProt accession | Protein | Length (amino acids) |
|---|---|---|---|
| NP | `P18272` | Nucleoprotein | 739 |
| VP40 | `Q05128` | Matrix protein VP40 | 326 |
| GP | `Q05320` | Envelope glycoprotein | 676 |
| L | `Q05318` | RNA-directed RNA polymerase L | 2,212 |

Open the **GP** entry (`Q05320`). On the page find:

- The **function** section: GP is the protein on the surface of the virus particle that it uses to attach to and enter cells.
- The **sequence** and its length (676 amino acids).
- The **organism** (strain Mayinga-76): the same 1976 isolate as our reference.

For VP35, VP30 and VP24, search for `Zaire ebolavirus VP35` and so on, and choose the reviewed (Swiss-Prot) entry. Why does this matter? Proteins such as GP are among the targets of vaccines, antibody treatments and diagnostic tests, so scientists pay close attention to changes in them. Knowing which genes a variant falls in is only the first step towards asking whether it could matter.

## Part 3: What virus is this?

We have a consensus genome. To find out what it is, we compare it with every sequence in the public database using **BLAST**, the tool you met in Lesson 1.5.

### Steps

1. Open <https://blast.ncbi.nlm.nih.gov/Blast.cgi> and click **Nucleotide BLAST**.
2. Under **Enter Query Sequence**, click **Choose File** and select your consensus FASTA, `SRR1553607.consensus.fasta`. (Alternatively, open the file and paste its contents into the box.)
   - **Windows with WSL:** in the file chooser, type `\\wsl$\Ubuntu\home\yourname\ebola_project\consensus` in the address bar (the `Ubuntu` part may be named differently on your computer). You can also run `explorer.exe consensus` to open the folder.
   - **Mac:** the file is in `~/ebola_project/consensus` in your home folder.
   - **JupyterHub:** download the file from the file browser first.
3. Leave the database as the nucleotide collection and the program as **Highly similar sequences (megablast)**.
4. Click **BLAST** and wait. For a 19 kb sequence this takes a minute or two.

### What we saw

When we ran this, the top hits were all *Zaire ebolavirus* genomes from the **2014 West African outbreak** (viruses called "Makona"):

| Hit | Isolate | Origin |
|---|---|---|
| `KR534529.1` | Makona-Conakry-1059 | Guinea, 2014 |
| `KR817244.1` | Makona-EM_080265 | Sierra Leone, 2014 |
| `KU143793.1` | Makona-S26 (complete genome) | Sierra Leone, 2014 |

with an identity of about **99%** over the covered region, and an E-value of 0.0. (The database grows over time, so your list of hits will differ in detail; the pattern should be the same.)

Compare that with the reference. The 1976 reference is far from the sample (559 differences). The best BLAST hits are **almost identical** to it. This confirms two things:

1. **What it is.** The sample contains *Zaire ebolavirus*, and it belongs with the 2014 West African (Makona) viruses.
2. **Why we saw so many variants.** The reference is a different, older lineage. Against a closer genome, such as `KU143793.1`, the difference would be tiny. You will test this for yourself in the capstone.

This also agrees with where the sample came from: collected in Sierra Leone in June 2014.

## What we have learned

Putting Modules 3 and 4 together, we can now answer the questions we started with:

| Question | Answer |
|---|---|
| Is Ebola virus in the sample? | Yes: 4,034 reads (1.19%) map to the genome |
| How much of the genome is recovered? | 98.97%, at a mean depth of 17x |
| How does it differ from the reference? | 559 single-base differences (394 well supported), in every gene |
| What is its genome? | A consensus of 18,959 bases with 219 unknown (`N`) bases |
| What is it closest to? | The 2014 West African (Makona) Zaire ebolavirus genomes |

The whole analysis took less than a minute of computing time. The effort was in understanding each step.

## If something looks different

| What you see | What it means | What to do |
|---|---|---|
| `bcftools view` prints the header, or `wc -l` is too big | You left out `-H` | Add `-H` |
| `[E::bcf_sr_...]` or `no such sequence` | The chromosome name is wrong | Use `NC_002549.1:start-end` exactly |
| A gene count differs slightly from the table | A different filter or tool version | Small differences are fine |
| BLAST says the sequence is too long or will not accept the file | The file was not uploaded | Paste the sequence into the box instead |
| BLAST takes a long time | The server is busy | Wait, or run it again later |
| Different BLAST hits | The database has changed | The top hits should still be Makona genomes |

## Practice check

1. Which gene has the most variants, and why is that not enough to say it varies most?
2. How do you compare genes of different lengths fairly?
3. Why can we not say from these results that a variant changes a protein?
4. What did BLAST tell us about the sample?
5. Why were there so many variants against the reference but the BLAST hits are near-identical to the sample?

### Answers

1. L, with 133, but L is also the longest gene (6,782 bases). Per kilobase it is in the middle (19.6); GP is highest (25.8).
2. Divide the number of variants by the gene's length in kilobases, to get variants per kilobase.
3. We counted variants by position but did not check how each one changes the codon. That needs an annotation-aware tool.
4. The sample is *Zaire ebolavirus*, and its closest relatives are the 2014 West African (Makona) genomes, matching where it was collected.
5. The reference is a 1976 isolate from a different outbreak and lineage, so it is distant from the sample. The BLAST hits are 2014 genomes from the same outbreak as the sample, so they are very close to it.

## Summary of commands

| Command | What it does |
|---|---|
| `bcftools view -H -t CHR:START-END file.vcf` | Shows the variants in a region |
| `bcftools view -H -t CHR:START-END file.vcf \| wc -l` | Counts the variants in a region |

Also: UniProt (<https://www.uniprot.org/>) for proteins, and NCBI BLAST (<https://blast.ncbi.nlm.nih.gov/>) for identifying a sequence.

## What comes next

You have taken the Ebola sample from raw reads to a genome, and identified it. In **Module 5** you apply the same ideas to a very different problem: a plant virus, sequenced with long reads.

[Module 5: Long-Read Analysis, the Cassava Virus Example](../module5/README.md)
