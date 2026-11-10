# Command Cheat Sheet

**Linux for Bioinformatics for Life Scientists · Bowen University · 16–18 November 2026**

Every command used in the course, grouped by what it does, with the lesson where you met it. Replace words in CAPITALS with your own file or folder names.

**Before anything else:** `conda activate bioinfo`. The prompt must begin with `(bioinfo)`.

---

## Finding your way (Lesson 2.1)

| Command | What it does |
|---|---|
| `pwd` | Shows where you are |
| `ls` | Lists files and folders |
| `ls -lh` | Long listing with human-readable sizes |
| `ls -a` | Includes hidden files |
| `ls -R` | Lists a folder and everything inside it |
| `cd FOLDER` | Moves into a folder |
| `cd ..` | Moves up one level |
| `cd ~` or `cd` | Moves to your home folder |
| `cd -` | Moves back to the previous folder |
| `clear` | Clears the screen |
| `history` | Shows earlier commands |
| `COMMAND --help` | Shows brief help |
| `man COMMAND` | Shows the full manual (**q** to quit) |

**Keys:** Tab completes names · ↑ recalls the last command · **Ctrl+C** stops a command · **q** leaves `less`, `man` and `tview`.

## Files and folders (Lesson 2.3)

| Command | What it does |
|---|---|
| `mkdir NAME` | Makes a folder |
| `mkdir -p A/B/C` | Makes nested folders (and several at once: `mkdir -p a b c`) |
| `cp FILE DEST` | Copies a file |
| `cp -r FOLDER DEST` | Copies a folder and its contents |
| `mv OLD NEW` | Renames or moves |
| `rm FILE` | Deletes a file (**permanently**) |
| `rm -r FOLDER` | Deletes a folder and its contents (**permanently**) |
| `nano FILE` | Edits a text file (Ctrl+O saves, Ctrl+X exits) |
| `du -sh NAME` | Shows how much space something uses |
| `*` and `?` | Wildcards: any characters; exactly one character |

## Looking inside files (Lesson 2.4)

| Command | What it does |
|---|---|
| `head -n 4 FILE` | First 4 lines |
| `tail -n 4 FILE` | Last 4 lines |
| `less FILE` | Scroll through a file (**q** to quit) |
| `wc -l FILE` | Counts lines |
| `zcat FILE.gz` | Prints a compressed file |
| `grep "PATTERN" FILE` | Lines containing the pattern |
| `grep -c "PATTERN" FILE` | Counts matching lines |
| `grep -v "PATTERN" FILE` | Lines **not** matching |
| `grep -n`, `-m N`, `-w`, `-o` | Show line numbers; stop after N; whole word; only the matching part |
| `^` in a pattern | Start of the line (`grep "^@SRR"`) |
| `A \| B` | Sends the output of `A` into `B` |
| `COMMAND > FILE` | Saves output to a file (replaces it) |
| `COMMAND >> FILE` | Adds output to the end of a file |

**Key idea:** reads in a FASTQ file = lines ÷ 4.

## Conda (Lesson 2.2)

| Command | What it does |
|---|---|
| `conda env list` | Lists environments |
| `conda activate bioinfo` | Switches the environment on |
| `conda deactivate` | Switches it off |
| `conda create -n NAME -c conda-forge -c bioconda PACKAGES -y` | Makes a new environment |
| `conda env remove -n NAME` | Deletes an environment |
| `which TOOL` and `TOOL --version` | Where a program is, and its version |

## Getting references (Lesson 3.1)

| Command | What it does |
|---|---|
| `efetch -db nuccore -id ACC -format fasta > FILE.fasta` | Downloads a sequence |
| `efetch -db nuccore -id ACC -format gff3 > FILE.gff3` | Downloads the annotation |
| `grep "ID=gene-" FILE.gff3` | Lists the genes in a GFF3 file |

## Quality control and trimming (Lessons 3.2 and 5.1)

| Command | What it does |
|---|---|
| `seqkit stats FILES` | Reads, lengths, totals |
| `seqkit stats -a FILES` | Adds quality, N50 and GC |
| `fastqc --outdir qc --threads 2 FILES` | Quality report for each file |
| `fastp -i R1 -I R2 -o OUT1 -O OUT2 -l 50 --detect_adapter_for_pe -h qc/fastp_report.html -j qc/fastp_report.json -w 4` | Trims paired reads |
| `NanoPlot --fastq FILE --outdir DIR --threads 4` | Quality plots for Nanopore reads |

## Mapping and alignments (Lessons 3.3, 3.4 and 5.2)

| Command | What it does |
|---|---|
| `bwa index REF.fasta` | Indexes a reference (once) |
| `bwa mem -t 4 REF R1 R2 2>/dev/null \| samtools sort -o OUT.bam -` | Maps paired short reads and sorts |
| `minimap2 -t 4 -ax map-ont REF READS \| samtools sort -o OUT.bam -` | Maps Nanopore reads and sorts |
| `samtools index FILE.bam` | Indexes a sorted BAM |
| `samtools flagstat FILE.bam` | Mapping summary. **Read `primary mapped`** |
| `samtools view -c -F 4 FILE.bam` | Counts mapped records |
| `samtools view -c -f 4 FILE.bam` | Counts unmapped records |
| `samtools view -b -f 4 IN.bam > OUT.bam` | Keeps only unmapped reads |
| `samtools fastq IN.bam > OUT.fastq` | Converts a BAM to FASTQ |
| `samtools fasta -f 4 FILE.bam` | Writes unmapped reads as FASTA |
| `samtools coverage FILE.bam` | Coverage and mean depth |
| `samtools coverage -m FILE.bam` | The same, with a histogram |
| `samtools depth -a -r CHR:START-END FILE.bam` | Depth at each position in a region |
| `samtools idxstats FILE.bam` | Reads per reference sequence |
| `samtools view -c FILE.bam CHR:START-END` | Counts reads in a region |
| `samtools tview FILE.bam REF.fasta` | Interactive viewer (**g** goes to a position, **q** quits) |
| `samtools tview -d T -p CHR:POS FILE.bam REF.fasta` | Prints one screen as text |

## Variants and consensus (Lessons 4.1 and 4.2)

| Command | What it does |
|---|---|
| `bcftools mpileup -f REF.fasta FILE.bam \| bcftools call -mv --ploidy 1 -o OUT.vcf` | Finds the variants |
| `grep -vc "^#" FILE.vcf` | Counts the variants |
| `grep -v "^#" FILE.vcf \| grep -c "INDEL"` | Counts indels |
| `bcftools view -H FILE.vcf` | Shows variants without the header |
| `bcftools view -H -t CHR:START-END FILE.vcf` | Variants in a region |
| `bcftools query -f "%POS %REF %ALT %QUAL %DP\n" FILE.vcf` | Prints chosen columns |
| `bcftools stats FILE.vcf \| grep "^SN"` | Summary of the variants |
| `bcftools view -i "QUAL>=30 && INFO/DP>=10" IN.vcf -o OUT.vcf` | Keeps well-supported variants |
| `samtools consensus -a -f fasta FILE.bam -o OUT.fasta` | Builds the consensus sequence |
| `samtools consensus -a -d 5 ...` | `N` wherever fewer than 5 reads cover |
| `grep -v ">" FILE.fasta \| grep -o "N" \| wc -l` | Counts the `N` characters |

## Classification and charts (Lessons 5.3 and 5.4)

| Command | What it does |
|---|---|
| `kraken2 --db DB --threads 4 --report REPORT --output OUT READS.fastq` | Classifies reads |
| `grep -w "S" REPORT` | Lists the species in a Kraken2 report |
| `grep -w "G" REPORT` | Lists the genera |
| `kreport2krona.py -r REPORT -o OUT.krona.txt` | Converts a report for Krona |
| `ktImportText A.txt B.txt -o ALL.html` | Builds one interactive chart |
| `tar -xzf FILE.tar.gz -C FOLDER` | Unpacks a compressed archive |
| `free -h` | Shows available memory |

## Packing your work (capstone)

| Command | What it does |
|---|---|
| `history > FILE.txt` | Saves the commands you typed |
| `tar -czf NAME.tar.gz FOLDER` | Packs a folder into one compressed file |

---

## Numbers to remember

| Dataset | Result |
|---|---|
| Ebola reads | 203,445 pairs; 168,889 pairs after trimming |
| Mapped to the 1976 reference | 4,034 reads (1.19%); 98.97% covered; 17.2x |
| Variants against the 1976 reference | 559, or 394 after filtering |
| Consensus | 18,959 bases, 219 `N` |
| barcode58 | 9,196 reads; 724 cassava (7.87%); 8,472 kept |
| Kraken2 on barcode58 | 1.18% classified, mostly begomoviruses |
| sample3 | 87% a single gemycircularvirus |

## The rules to live by

1. Raw data is never edited. Work on copies.
2. Look at the data before you trust it.
3. Check every number a second way.
4. Name files clearly, and keep one folder per project.
5. A filter is a decision. Write down what you chose.
6. An `N` is honest. A guess is not.
