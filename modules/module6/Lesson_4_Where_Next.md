# Lesson 6.4: Where Next, Bioinformatic Fridays

**Module 6: Wrap-Up and Next Steps**

## What you will need

- Nothing. This lesson is a guide to what to do next.

## What you will learn

By the end of this lesson you will be able to:

- Say what you can now do, and what you have not yet covered
- Explain why some topics were left out
- Choose sensible next steps for your own work
- Find the Bioinformatic Fridays materials

## What you can do now

Look at how far you have come. At the start of the training, most of you had never used a terminal. You can now:

- **Move around a Linux system** and manage files and folders without a mouse
- **Install software** with Conda and keep it in a tidy environment
- **Recognise the main sequence file formats** and read them
- **Check the quality** of short and long reads, and clean them
- **Map reads** to a reference and judge the result
- **Find variants**, build a **consensus genome**, and identify what a sample is
- **Remove host reads and classify** what remains, and draw the result
- **Read your own results critically**: the right line of `flagstat`, a filter as a decision, a rare taxon as a possible artefact

That is a real set of skills, and every part of it is used in published work.

## What this course left out, and why

We left some important topics out **on purpose**. Learning the command line and one complete workflow properly was a better use of three days than touching everything lightly. These are the natural next steps.

| Topic | What it is | Why it comes next |
|---|---|---|
| **Shell scripting** | Writing a file of commands (with variables and loops) so the computer repeats them for you | You typed every command yourself. For 50 samples you would want a script |
| **Git and GitHub** | Tracking changes to your work and sharing it | Reproducible, shareable analysis |
| **Remote servers (SSH)** | Running analyses on a powerful remote computer | Big datasets do not fit on a laptop |
| **Genome assembly** | Building a genome from reads, without a reference | When no good reference exists |
| **Phylogenetics** | Building family trees from sequences | Comparing many genomes, for example to place the Ebola sample on a tree |
| **Annotating variants** | Working out whether a variant changes a protein | The step beyond counting variants per gene |
| **Amplicon (16S) analysis** | Identifying microbes from a marker gene | The 16S files in your training folder were never used |
| **Statistics and visualisation** | Testing whether differences are real and plotting them | Comparing groups, for example case against healthy |
| **Workflow managers** | Tools that run whole pipelines automatically | Larger projects |

## Bioinformatic Fridays

If you enjoyed this, **Bioinformatic Fridays** is the natural next step. It is a beginner-friendly learning series from the same team, and its materials are public:

**<https://github.com/omyxlab/Bioinformatic_Fridays>**

### Join the class

Bioinformatic Fridays is also a **live weekly class**. It meets **every Friday from 12:00 to 14:00, in person at Covenant University**, and a **new cohort has just started**.

The team is considering an **online option**, so that people who cannot come to Covenant University can take part too. **If you would like to join, or you would be interested in an online class, please let us know** by emailing **bioinformaticsfridays@gmail.com**.

### The materials

Its modules follow on directly from what you have done:

| Module | What it covers |
|---|---|
| Module 1: Linux | Fundamentals, text processing and pipes, scripting and permissions, SSH and remote work, paths |
| Module 2: Conda | Environments, channels, and building a bioinformatics environment |
| Module 3: Omics | The omics hierarchy, quality control, genome assembly, de novo and reference-based assembly |
| Module 4: Recap exercises | Practice problems that combine Linux, Conda and omics, with solutions |
| Module 5: Project | A begomovirus phylogenetics exercise |
| Module 6: Problem solving | Computational thinking, and debugging |
| Module 7: Bash scripting | Scripts, variables, loops and log files |
| Module 8: GitHub | Accounts, repositories, branches, collaboration |

A sensible order for most of you: **Module 7 (Bash scripting)** first, so that you can automate what you did by hand here, then **Module 3** (assembly), and **Module 8** (GitHub) to keep your work organised and shareable.

## Next steps for your own work

1. **Finish the capstone.** It is the best consolidation you can do.
2. **Redo the workflows from scratch** at home, with the commands in Lessons 6.1 and 6.2, until you could do them without the lessons open.
3. **Try your own data.** Pick a question from your own research and ask which tools from this course apply. Even if your organism or data type is different, the steps (QC, clean, map or classify, interpret) will be much the same.
4. **Read the manuals.** Every tool has a `--help` option and a website. You now know enough to read them. `samtools`, `bcftools`, `seqkit` and `fastp` are all well documented.
5. **Build your cheat sheet.** The [command cheat sheet](../reference/Command_Cheat_Sheet.md) is a start; add your own notes.
6. **Ask questions.** Communities such as Biostars (<https://www.biostars.org/>) are full of people who have met the same problems.
7. **Keep the habits.** Raw data untouched. Files named clearly. Every number checked a second way. Every filter written down.

## A final thought

Everything you ran in this course took seconds. The hard part was never typing the commands. It was knowing what each one means, which number to trust, and what the result does and does not show. That judgment is what makes a bioinformatician, and you have started to build it.

Thank you for taking part.

## Practice check

1. Name two topics the course left out, and say why each is a good next step.
2. Which Bioinformatic Fridays module would you do first to automate the commands you typed, and why?
3. What is one habit from this course that you will keep?

### Answers

1. For example: shell scripting, so you can repeat a workflow on many samples without retyping it; and phylogenetics, so you can compare the sample with many other genomes.
2. Module 7 (Bash scripting), because it teaches you to put commands in a script with variables and loops.
3. Any of the habits listed above, such as keeping raw data untouched, or checking a count a second way.
