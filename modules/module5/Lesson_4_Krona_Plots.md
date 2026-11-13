# Lesson 5.4: Krona Plots

**Module 5: Long-Read Analysis, the Cassava Virus Example**

## What you will need

- The `bioinfo` environment active
- `~/cassava_project/kraken/barcode58.report`, `barcode57.report` and `sample3.report` (Lesson 5.3)
- A web browser
- Krona's taxonomy database updated, **if you are on your own computer** (see "Before you start")

## What you will learn

By the end of this lesson you will be able to:

- Convert a Kraken2 report into the input that Krona needs
- Build an interactive Krona chart with `ktImportText`
- Open the chart and explore it
- Combine several samples into one chart for comparison
- Explain what the chart shows, including the large "unclassified" wedge

## Why this matters

A Kraken2 report is accurate but hard to take in. Looking at columns of numbers, it is difficult to see at a glance what a sample is made of. A picture helps.

**Krona** draws the classification as a set of **concentric rings**, like a multi-layered pie chart. The centre is the whole sample; each ring outwards is a finer level of classification (from kingdom to species). The size of each slice is the number of reads. The chart is **interactive**: you click a slice to zoom in, and hover to see exact counts.

It is also a good way to **share results** with someone who does not use a terminal: a single HTML file that opens in any browser.

## Before you start: update Krona's taxonomy (own computer only)

Krona keeps its own copy of the NCBI taxonomy, the list of organisms and how they are related. On a fresh installation this copy has not been filled in yet. That is the reason Krona printed a message about updating its taxonomy databases when it was installed in Lesson 2.2. It is a **one-off** step.

- **Windows with WSL, Mac or Linux (your own computer):** with the `bioinfo` environment active and an internet connection, run:

  ```
  $ ktUpdateTaxonomy.sh
  ```

  It downloads the NCBI taxonomy and builds Krona's copy. This takes a few minutes and prints progress as it goes. Wait for your prompt to return. If you already ran it as part of [Module 0](../module0/Getting_Started.md), you do not need to run it again.

- **JupyterHub (training.omyxlab.com):** the trainers have **already done this** for the shared `bioinfo` environment. **Do not run `ktUpdateTaxonomy.sh`.** Go straight to Step 1.

## Step 1: Convert the report

Krona does not read a Kraken2 report directly. A small helper from **KrakenTools**, `kreport2krona.py`, converts it into Krona's text format.

```
$ cd ~/cassava_project
$ kreport2krona.py -r kraken/barcode58.report -o kraken/barcode58.krona.txt
```

| Part | Meaning |
|---|---|
| `-r kraken/barcode58.report` | The Kraken2 report to read |
| `-o kraken/barcode58.krona.txt` | The Krona text file to write |

It prints nothing when it works. Look at the result:

```
$ head -n 8 kraken/barcode58.krona.txt
8372	Unclassified
0	p__Cressdnaviricota
0	p__Cressdnaviricota	c__Repensiviricetes
0	p__Cressdnaviricota	c__Repensiviricetes	o__Geplafuvirales
0	p__Cressdnaviricota	c__Repensiviricetes	o__Geplafuvirales	f__Geminiviridae
0	p__Cressdnaviricota	c__Repensiviricetes	o__Geplafuvirales	f__Geminiviridae	g__Begomovirus
55	p__Cressdnaviricota	c__Repensiviricetes	o__Geplafuvirales	f__Geminiviridae	g__Begomovirus	s__Begomovirus_manihotisburkinafasoense
26	p__Cressdnaviricota	c__Repensiviricetes	o__Geplafuvirales	f__Geminiviridae	g__Begomovirus	s__Begomovirus_manihotis
```

Each line is a **count**, then the **lineage**, one rank per column (`p__` phylum, `c__` class, `o__` order, `f__` family, `g__` genus, `s__` species). The same numbers as the report, laid out as paths through the family tree. Count the lines:

```
$ wc -l kraken/barcode58.krona.txt
17 kraken/barcode58.krona.txt
```

## Step 2: Build the chart

```
$ ktImportText kraken/barcode58.krona.txt -o kraken/barcode58.krona.html
Writing kraken/barcode58.krona.html...
```

`ktImportText` reads the text file and writes an **HTML file**. Check it:

```
$ ls -lh kraken/barcode58.krona.html
-rw-r--r-- 1 yourname yourname 224K Nov 10 23:30 kraken/barcode58.krona.html
```

A 224 KB file. It contains everything it needs, so it works **offline** and you can email it or put it on a USB drive.

## Step 3: Open the chart

A terminal cannot show a chart, so open the HTML file in your browser.

- **Windows with WSL:** run `explorer.exe kraken` to open the folder in Windows File Explorer, then double-click `barcode58.krona.html`. (Or type `\\wsl$\Ubuntu\home\yourname\cassava_project\kraken` in the File Explorer address bar; the `Ubuntu` part may be named differently on your computer.)
- **Mac:** run `open kraken`, then double-click the file.
- **JupyterHub:** in the file browser, find the `kraken` folder and open or download the HTML file.

## Step 4: Explore the chart

When it opens, you see a circle made of rings.

- **The centre** (labelled `all`) is every read in the sample.
- **The outer rings** are finer levels of classification, from root outwards.
- **The angle** of a slice is the number of reads.

You will immediately see that the circle is **almost entirely one colour: "Unclassified"**. This is the 8,372 reads (98.8%) that Kraken2 could not name, so it takes up nearly the whole circle. That is an accurate picture of the data, and it is a good illustration of what we discussed in Lesson 5.3.

To see the viruses, you have to zoom in:

1. **Click on the slice labelled `Viruses`** (it is a thin sliver). The chart zooms so that Viruses fills the whole circle.
2. You now see the rings for *Cressdnaviricota*, *Geminiviridae*, *Begomovirus*, and finally the two species.
3. **Hover** over a slice. A box on the right shows its name and the **number of reads** and the percentage.
4. **Click the centre** to zoom back out.

At the top of the page there is a **search** box, and options to change how deep the rings go. Try typing `Begomovirus` into the search to highlight it.

What you should find: within the viruses, **the begomovirus slice dominates**, split between *Begomovirus manihotisburkinafasoense* (55 reads) and *Begomovirus manihotis* (26 reads), with small slices for the other viruses.

## Step 5: Compare all three samples in one chart

Make Krona files for the other two samples:

```
$ kreport2krona.py -r kraken/barcode57.report -o kraken/barcode57.krona.txt
$ kreport2krona.py -r kraken/sample3.report -o kraken/sample3.krona.txt
$ wc -l kraken/*.krona.txt
  75 kraken/barcode57.krona.txt
  17 kraken/barcode58.krona.txt
   7 kraken/sample3.krona.txt
  99 total
```

barcode57 has 75 lines because of its long tail of species with one or two reads. `sample3` has only 7, because it is dominated by a single species. Now put all three into **one** chart. `ktImportText` takes several files at once:

```
$ ktImportText kraken/barcode57.krona.txt kraken/barcode58.krona.txt kraken/sample3.krona.txt -o kraken/all_samples.krona.html
Writing kraken/all_samples.krona.html...
```

Open `all_samples.krona.html`. This time there is a **list of samples** in the top left (barcode57, barcode58, sample3). Click each name to switch between them.

- **barcode57 and barcode58** look alike: almost all "Unclassified", with a begomovirus sliver.
- **sample3** looks nothing like them. Almost the whole circle is classified, and it is one species: *Gemycircularvirus sp.*

A picture makes the difference between the samples obvious, in a way the numbers alone do not.

## Summary: what the Krona chart shows, and what it does not

**It shows** the proportions of what Kraken2 recognised, laid out by taxonomy, with exact read counts one click away.

**It does not show** what the unclassified reads are, or whether a small slice is real. A chart makes every slice look equally trustworthy. The rule from Lesson 5.3 still applies: be cautious about slices with only a handful of reads, such as the long tail in barcode57, or the poxvirus in barcode58.

## If something looks different

| What you see | What it means | What to do |
|---|---|---|
| `kreport2krona.py: command not found` | The environment is not active, or KrakenTools is missing | `conda activate bioinfo`; check `which kreport2krona.py` |
| `ktImportText: command not found` | The environment is not active | `conda activate bioinfo` |
| The `.krona.txt` file is empty | The Kraken2 report was empty or the wrong file | Check `head kraken/barcode58.report` |
| The HTML opens as plain text | Your browser did not run it | Right-click the file, and choose to open it with a browser |
| The chart is blank | The browser blocked scripts, or the file did not finish | Try another browser; rerun `ktImportText` |
| You see only a grey circle | Everything is "Unclassified" | Click `Viruses`; if it does not exist, the report has no classified reads |
| The file will not open on the JupyterHub | The hub's viewer does not run scripts | Download the file and open it on your own computer |
| A message about updating Krona's taxonomy | The taxonomy update has not been run on this computer | On your own computer, run `ktUpdateTaxonomy.sh` once (see "Before you start"). On the JupyterHub it is already done: tell a trainer |
| `ktUpdateTaxonomy.sh` stops with a download error | The connection dropped | Check your internet connection and run it again |

## Practice check

1. What does Krona need as input, and which program prepares it from a Kraken2 report?
2. Why is the chart almost entirely one slice, and what is that slice?
3. How do you see the viruses in the chart?
4. What is the main difference between the sample3 chart and the barcode58 chart?
5. Why is it wise to be careful when reading a Krona chart?

### Answers

1. A Krona text file of counts and lineages. `kreport2krona.py` makes it from the Kraken2 report.
2. It is "Unclassified", the 98.8% of reads that Kraken2's viral database could not name.
3. Click the `Viruses` slice to zoom in.
4. barcode58 is almost all unclassified with a small begomovirus slice. sample3 is almost entirely classified, as one species of gemycircularvirus.
5. A chart gives every slice the same visual weight, even those based on only one or two reads. Always check the read counts.

## Summary of commands

| Command | What it does |
|---|---|
| `kreport2krona.py -r in.report -o out.krona.txt` | Converts a Kraken2 report into Krona text format |
| `ktImportText in.krona.txt -o out.krona.html` | Builds an interactive chart |
| `ktImportText a.txt b.txt c.txt -o all.html` | Builds one chart comparing several samples |
| `explorer.exe folder` | (Windows with WSL) Opens a folder in File Explorer |

## What comes next

You have now analysed two very different datasets from raw reads to a final result. In **Module 6** we put the two workflows side by side, introduce the take-home capstone, and look at where to go next.

[Module 6: Wrap-Up and Next Steps](../module6/README.md)
