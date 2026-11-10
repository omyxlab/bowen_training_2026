# Module 0: Getting Started

**Linux for Bioinformatics for Life Scientists · Bowen University · 16–18 November 2026**

Please complete this guide **before Day 1**. It takes about 45 to 90 minutes, mostly waiting for downloads. By the end you will have a working Linux terminal on your own computer, a tool called Conda, and a ready-made set of bioinformatics programs. We will use them from the first morning.

You do not need any previous experience. Every command is written out so you can copy and paste it. If something goes wrong, go to **Part 9: Troubleshooting** at the end, and if you are still stuck, do not worry: we have a backup plan (see Part 1).

---

## Before you start: your checklist

Tick each box as you finish it.

- [ ] Part 2: My computer meets the requirements
- [ ] Part 3: I checked whether WSL is already installed
- [ ] Part 4: I installed Ubuntu and can open it
- [ ] Part 5: I updated Ubuntu
- [ ] Part 6: I installed Miniforge (Conda)
- [ ] Part 7: I created the `bioinfo` environment
- [ ] Part 8: I downloaded the training data and ran the final check

---

## Part 1: Which route is yours?

| Your computer | What to do |
|---|---|
| **Windows 10 or 11 laptop** | Follow this whole guide (Route A: WSL). |
| **Mac** | Skip to **Part 10 (Mac users)**. |
| **Linux computer** | You already have a terminal. Start at Part 6. |
| **Windows computer that cannot run WSL**, or you get stuck | No problem. On Day 1, 08:00–09:00, we will give you a temporary login to an online system (JupyterHub) that already has everything installed. Tell a trainer when you arrive. The login works only for the duration of the training. |

**What is WSL?** WSL stands for *Windows Subsystem for Linux*. It lets a Windows computer run a real Linux system (called Ubuntu) inside a normal window, without wiping or changing Windows. You can remove it later if you want.

---

## Part 2: Check that your computer is ready

You need:

- Windows 10 (version 2004 or newer) or Windows 11
- At least **15 GB of free disk space**
- **8 GB of RAM or more** is recommended. One later step (Lesson 5.2) needs about 5 GB of free memory, and WSL normally gets only half of your computer's memory, so on an 8 GB laptop you may need the JupyterHub for that step. If you have less than 8 GB, tell us on Day 1.
- A reliable internet connection for the downloads
- Your laptop charger

### 2.1 Check your Windows version

1. Press the **Windows key + R** together.
2. Type `winver` and press **Enter**.
3. A small window opens. If it says **Windows 11**, you are fine. If it says **Windows 10**, the *Version* must be **2004 or higher** (for example 21H2, 22H2).

If your Windows is older, run **Windows Update** first, or use the JupyterHub backup (Part 1).

### 2.2 Check that virtualisation is switched on

WSL needs a computer feature called *virtualisation*. It is usually already on.

1. Press **Ctrl + Shift + Esc** to open Task Manager.
2. Click the **Performance** tab, then **CPU**.
3. Look at the bottom right for **Virtualization**. It should say **Enabled**.

If it says **Disabled**, it must be switched on in your computer's BIOS settings. The steps differ by manufacturer, so search the internet for *"enable virtualization BIOS"* followed by your laptop's brand and model (for example "HP ProBook"). If this looks too difficult, use the JupyterHub backup (Part 1) instead.

---

## Part 3: Check whether WSL is already installed

You only need to do this once. It tells you which step to do next.

1. Click the **Start** button, type `PowerShell`, and click **Windows PowerShell**. A window with a blue or black background opens. (You do not need to run it as administrator yet.)
2. Type this and press **Enter**:

```
wsl --list --verbose
```

Look at what appears:

| What you see | What it means | What to do |
|---|---|---|
| A table with the columns **NAME**, **STATE** and **VERSION**, and a row that says **Ubuntu** (or **Ubuntu-24.04**, **Ubuntu-22.04**) | WSL and Ubuntu are already installed. | Go to **Part 3.1** below, then continue to **Part 5**. |
| A message saying **"Windows Subsystem for Linux has no installed distributions"** | WSL is present but Ubuntu is not. | Go to **Part 4**. |
| A long list of options (help text), an error, or **"'wsl' is not recognized"** | WSL is not installed yet (or is very old). | Go to **Part 4**. The command there installs WSL as well. |
| A table that lists other systems (for example *docker-desktop*) but no Ubuntu | WSL is installed, but without Ubuntu. | Go to **Part 4**. |

### 3.1 If Ubuntu is already installed

Check the **VERSION** column. It should say **2**. If it says **1**, run this (use the name exactly as shown in the NAME column):

```
wsl --set-version Ubuntu 2
```

Then open Ubuntu: click **Start**, type `Ubuntu`, and click **Ubuntu**. A black window opens with a prompt that looks like this:

```
yourname@YOUR-PC:~$
```

If it asks you to create a username and password, follow Part 4, steps 5 and 6.

---

## Part 4: Install Ubuntu

1. Click **Start**, type `PowerShell`, then **right-click** *Windows PowerShell* and choose **Run as administrator**. Click **Yes** when Windows asks for permission.
2. Type this and press **Enter**:

```
wsl --install Ubuntu
```

3. Wait. It downloads and installs Ubuntu, which can take several minutes. You will see progress messages.

   - If you see a long list of help text instead, your version of Windows may need the older form of the command. Try this instead:

     ```
     wsl --install -d Ubuntu
     ```

   - If that still fails, run `wsl --update`, then try again.

4. **Restart your computer** if it asks you to (or if nothing seems to happen). After restarting, a window called **Ubuntu** usually opens by itself and says *"Installing, this may take a few minutes..."*. If it does not open, click **Start**, type `Ubuntu`, and open it. Wait until it asks for a username.

5. When it says **"Enter new UNIX username:"**, type a short username using **lowercase letters only, no spaces** (for example `ade` or `ngozi`). Press **Enter**.

6. When it says **"New password:"**, type a password and press **Enter**. **You will not see anything appear as you type. This is normal.** It is not broken. Type it carefully, press Enter, then type the same password again to confirm.

   **Write this password down.** You will need it again in Part 5.

7. You should now see a prompt like this:

```
yourname@YOUR-PC:~$
```

You are now inside Linux. This window is called the **terminal**.

### 4.1 Useful things to know about the terminal

- **To paste text**: right-click in the window, or press **Ctrl + Shift + V**. (Ctrl + V on its own does not work in many terminals.)
- **To copy text** you have selected: **Ctrl + Shift + C**.
- **To close Ubuntu**: type `exit` and press Enter, or just close the window.
- **To reopen Ubuntu later**: Start menu, type `Ubuntu`, open it.
- **Commands must be typed exactly**, including spaces and capital letters. Linux notices the difference between `NanoPlot` and `nanoplot`.
- **You press Enter after every command.**

### 4.2 Where you should work

When you open Ubuntu, type:

```
pwd
```

It should print something starting with **`/home/`**, for example `/home/ade`. That is your Linux home folder, and it is where you should do all your work.

> **Important:** if `pwd` prints something starting with **`/mnt/c/`**, you are inside your Windows files. Type `cd ~` and press Enter to go back to your Linux home folder. Everything in this course should happen in your home folder, because it is much faster than working in `/mnt/c/`.

---

## Part 5: Update Ubuntu and install two small helpers

In the Ubuntu terminal, type:

```
sudo apt update
```

It asks for your password (the one from Part 4). You will not see it as you type. Press Enter. A list of messages scrolls past, and it finishes when the prompt comes back.

Then type:

```
sudo apt install -y wget curl
```

`wget` and `curl` are programs that download files. If it says they are already installed, that is fine.

---

## Part 6: Install Miniforge (this gives you Conda)

**What is Conda?** Conda is a tool that installs bioinformatics software for you, along with everything each program needs to run, so you do not have to install each one by hand. **Miniforge** is a free installer for Conda. We use it because it is quick to set up and has no licence restrictions.

### 6.1 Check your computer type

In the Ubuntu terminal, type:

```
uname -m
```

- If it prints **`x86_64`**, continue with the next step. This is the normal result.
- If it prints **`aarch64`**, your computer uses an ARM processor. Some of the tools we use may not be available for it. Use the JupyterHub backup (Part 1) and tell a trainer on Day 1.

### 6.2 Download the installer

Type this (it is one long line, so copy it all):

```
wget https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh
```

Wait until the download finishes and you get your prompt back.

### 6.3 Run the installer

```
bash Miniforge3-Linux-x86_64.sh
```

The installer asks a few questions:

1. It shows a licence. **Press Enter**, then keep pressing **Space** to scroll to the end (or press **q** to skip to the question).
2. When it asks **"Do you accept the license terms? [yes|no]"**, type `yes` and press **Enter**.
3. When it asks where to install, it shows a location such as `/home/yourname/miniforge3`. **Press Enter** to accept it. Do not change it.
4. When it asks **"Do you wish to update your shell profile to automatically initialize conda? [yes|no]"**, type `yes` and press **Enter**.

It installs for a minute or two and then says it is finished.

### 6.4 Close and reopen Ubuntu

**This step matters.** Close the Ubuntu window (type `exit`, or click the X). Then open Ubuntu again from the Start menu.

Your prompt should now begin with **`(base)`**, like this:

```
(base) yourname@YOUR-PC:~$
```

### 6.5 Check that Conda works

```
conda --version
```

It should print something like `conda 24.x.x` (the numbers can differ). If it does, Conda is ready.

> **If you see `conda: command not found`**, type these two lines (one at a time), then close and reopen Ubuntu, and check again:
>
> ```
> source ~/miniforge3/bin/activate
> conda init bash
> ```

You can delete the installer file to save space:

```
rm Miniforge3-Linux-x86_64.sh
```

---

## Part 7: Create the `bioinfo` environment

**What is an environment?** An environment is a separate box that holds a set of programs. We put all of this course's tools in one box called `bioinfo`, so they never interfere with anything else on your computer.

You are about to install these tools: `seqkit`, `fastqc`, `fastp`, `bwa`, `minimap2`, `samtools`, `bcftools`, `nanoplot`, `entrez-direct`, `kraken2`, `krona` and `krakentools`. You do not need to know what they do yet. We will use each one during the course.

> **Do this on a good internet connection.** The plan has about 150 packages and **about 510 MB of downloads**, and the finished environment takes up about 2.7 GB on disk. It can take anything from a few minutes to half an hour. You can leave it running. Do not close the window until your prompt comes back.

### 7.1 Create the environment

Copy this whole command (it is one long line), paste it into Ubuntu, and press **Enter**:

```
conda create -n bioinfo -c conda-forge -c bioconda seqkit fastqc fastp bwa minimap2 samtools bcftools nanoplot entrez-direct kraken2 krona krakentools -y
```

Lots of text will scroll past. This is normal. When it finishes, you will see your prompt again.

### 7.2 Switch the environment on

```
conda activate bioinfo
```

Your prompt should now begin with **`(bioinfo)`** instead of `(base)`:

```
(bioinfo) yourname@YOUR-PC:~$
```

**You must type `conda activate bioinfo` every time you open a new Ubuntu window.** If the prompt says `(base)` and tools seem to be missing, this is almost always the reason.

### 7.3 Check that all the tools are installed

With `(bioinfo)` showing in your prompt, type:

```
which seqkit fastqc fastp bwa minimap2 samtools bcftools NanoPlot efetch kraken2 ktImportText
```

You should see **11 lines**, each a path ending in the tool's name, like this:

```
/home/yourname/miniforge3/envs/bioinfo/bin/seqkit
/home/yourname/miniforge3/envs/bioinfo/bin/fastqc
...
```

If a tool is **missing from the list**, its line will simply not appear. Count the lines. If you have fewer than 11, see Part 9.

Now check two tools actually run:

```
seqkit version
samtools --version | head -1
```

You should see version numbers (for example `seqkit v2.x.x` and `samtools 1.x`). The exact numbers do not matter.

---

## Part 8: Download the training data

The training files (about 90 MB) are in a single compressed file called `training.tar.gz`.

> **Download page:** <https://zenodo.org/records/24057548>

### Option 1: download directly inside Ubuntu (easiest)

Copy this command (it is one long line), paste it into Ubuntu, and press Enter:

```
cd ~
wget "https://zenodo.org/records/24057548/files/training.tar.gz?download=1" -O training.tar.gz
```

It downloads a file of about 90 MB. Wait until your prompt comes back.

### Option 2: download in Windows and copy it across

1. Open <https://zenodo.org/records/24057548> in your normal web browser and click the **Download** button next to `training.tar.gz`. The file goes to your Windows **Downloads** folder.
2. In Ubuntu, find your Windows username:

   ```
   ls /mnt/c/Users
   ```

   You will see a list of folder names. Yours is the one that matches your Windows account (ignore `Public`, `Default` and similar).
3. Copy the file into your Linux home folder. Replace `WINDOWSNAME` with your Windows username:

   ```
   cp /mnt/c/Users/WINDOWSNAME/Downloads/training.tar.gz ~/
   ```

### Unpack the data

Whichever option you used, run:

```
cd ~
tar -xzf training.tar.gz
```

Then check it:

```
ls Training
```

You should see two folders:

```
long_reads  short_reads
```

If you see these, you are done with the data.

> **Two larger files** (a virus database and a cassava genome) will be used on Day 2. **Do not download them.** We will give them to you separately.

---

## Final check

Open a **new** Ubuntu window and run these four commands, one at a time:

```
conda activate bioinfo
seqkit version
ls ~/Training
pwd
```

You should see:

- Your prompt change to `(bioinfo)`
- A `seqkit` version number
- The two folders `long_reads` and `short_reads`
- A path starting with `/home/`

**If all four worked, you are ready for Day 1.** Please also tick the checklist at the top of this guide.

---

## Part 9: Troubleshooting

| Problem | What to try |
|---|---|
| `wsl --install` shows a long list of options instead of installing | Run `wsl --update` and try again. If it still fails, try `wsl --install -d Ubuntu`. Make sure PowerShell was opened with **Run as administrator**. |
| The Ubuntu download fails or cannot connect (for example on a university network or if the Microsoft Store is blocked) | In the administrator PowerShell, run `wsl --install --web-download -d Ubuntu`. This downloads Ubuntu directly instead of through the Microsoft Store. |
| An error mentioning **virtualization** or error code `0x80370102` | Virtualisation is switched off. See Part 2.2. |
| The install seems to hang or does nothing | Restart the computer, then open **Ubuntu** from the Start menu. It often finishes the install on the first launch. |
| Ubuntu opens and closes immediately | Open PowerShell and type `wsl --list --verbose`. If Ubuntu is listed, type `wsl` to enter it and read any error message. Take a photo or screenshot of the message and email it to bioinformaticsfridays@gmail.com. |
| You forgot your Ubuntu password | Tell a trainer on Day 1. We can reset it in a few moments. |
| Nothing appears when you type your password | This is normal. Linux hides passwords completely. Type it and press Enter. |
| `command not found` for a tool | Look at your prompt. If it does not begin with `(bioinfo)`, type `conda activate bioinfo` and try again. |
| `conda: command not found` | See the box at the end of Part 6.5. |
| Pasting does not work | Use right-click or **Ctrl + Shift + V** in the terminal. |
| The download is very slow or keeps stopping | Try again later or on a different network. You can also bring your laptop to the Day 1 setup hour (08:00–09:00) and we will help. |
| The `conda create` command fails or shows an error | Run the command again: Conda keeps what it has already downloaded. If the error mentions `CondaHTTPError`, `CONNECTION FAILED` or a timeout, your connection is slow: type `conda config --set remote_read_timeout_secs 120` and run the command again. If it fails a second time, take a screenshot of the error and email it to bioinformaticsfridays@gmail.com, or come to the Day 1 setup hour. |
| `Disk quota` or `No space left on device` | Free up space on your Windows drive (you need at least 15 GB) and try again. |
| You made a mistake and want to start the environment again | Type `conda deactivate`, then `conda env remove -n bioinfo`, then repeat Part 7. |

### If you are completely stuck

**Do not worry.** Come to the **Day 1 setup hour, 08:00–09:00**, with your laptop and charger, and tell a trainer. If your computer cannot run WSL, we will give you a temporary JupyterHub login that works in your web browser, with everything already installed. You will not miss anything.

**Contact us before the day:** <bioinformaticsfridays@gmail.com>. Please include a short description of the problem, which step (Part number) you were on, and a photo or screenshot of any error message.

---

## Part 10: Mac users

Your Mac already has a terminal, so you do not need WSL.

1. Open **Terminal** (press **Command + Space**, type `Terminal`, press Enter).
2. Check your chip by typing:

   ```
   uname -m
   ```

   It prints `arm64` (Apple Silicon, M1 or newer) or `x86_64` (Intel).
3. Download and install Miniforge. For **Apple Silicon** (`arm64`):

   ```
   curl -L -O https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-MacOSX-arm64.sh
   bash Miniforge3-MacOSX-arm64.sh
   ```

   For **Intel** (`x86_64`):

   ```
   curl -L -O https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-MacOSX-x86_64.sh
   bash Miniforge3-MacOSX-x86_64.sh
   ```

   Answer the questions as described in Part 6.3 (accept the licence, accept the default location, answer `yes` to initialising).
4. Close Terminal and open it again. Your prompt should begin with `(base)`.
5. Continue with **Part 7** (create the environment) and **Part 8** (download the data) exactly as written above. Wherever it says "Ubuntu", read "Terminal". Where it says "Option 2", your Downloads folder is `~/Downloads`, so the copy command is `cp ~/Downloads/training.tar.gz ~/`.

Some of the tools may not be available for every Mac chip. If the environment fails to create, tell a trainer on Day 1 and we will give you a JupyterHub login.

---
