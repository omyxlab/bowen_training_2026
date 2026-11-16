# Offline Ubuntu on Windows (WSL) — Instructor Guide

A step-by-step guide for instructors to set up **Ubuntu 24.04 on WSL 2** on learners' Windows PCs **without relying on the internet**, using USB drives or a local network share.

> **Why this guide exists:** when the venue's connection is slow or unreliable, 8+ people downloading hundreds of MB at once will fail. Download everything **once**, copy it to USB drives, and install from those.

---

## 1. Before the session (instructor, on a machine with good internet)

### 1.1 Download the files

| # | File | Where to get it |
|---|------|-----------------|
| 1 | **Ubuntu 24.04.5 LTS WSL image** (`ubuntu-24.04.5-wsl-amd64.wsl`, about 371 MB, for standard 64-bit Intel/AMD PCs) | Direct link: <https://releases.ubuntu.com/noble/ubuntu-24.04.5-wsl-amd64.wsl> (all files: <https://releases.ubuntu.com/noble/>) |
| 2 | **WSL installer** (`wsl.3.0.5.0.x64.msi`, for standard 64-bit Intel/AMD PCs) | Direct link: <https://github.com/microsoft/WSL/releases/download/3.0.5/wsl.3.0.5.0.x64.msi> (all releases: <https://github.com/microsoft/WSL/releases>) |

Notes:

- Most laptops are **x64**. If a learner has an ARM-based Windows laptop (e.g. Snapdragon), download the **arm64** versions of both files instead.
- Do **not** use the old `wsl_update_x64.msi`. Use the newest MSI from the WSL releases page.
- Alternative to the `.wsl` image: a root filesystem tarball from <https://cloud-images.ubuntu.com/wsl/noble/current/> (`ubuntu-noble-wsl-amd64-wsl.rootfs.tar.gz`). See [Fallback B](#fallback-b-import-a-tarball).

### 1.2 Verify the downloads

On the instructor's PC (PowerShell), compare the Ubuntu image against the `SHA256SUMS` file published next to it on the releases page:

```powershell
Get-FileHash .\ubuntu-24.04.5-wsl-amd64.wsl -Algorithm SHA256
```

Do this once, on the original download. Then every copy you make from it is a known-good copy.

### 1.3 Prepare the USB drives

Put these in one folder on each drive:

```
WSL-offline/
├── ubuntu-24.04.5-wsl-amd64.wsl
├── wsl.3.0.5.0.x64.msi
├── README.txt        (optional: the learner steps from section 3)
└── (optional) bioinfo.tar.gz, Kraken2 database, training data. See section 6.
```

- One 8 GB (or larger) USB drive per 2–3 learners is plenty. Copy the folder to each drive before the session.
- Alternatively, share the folder over the local network (a network share on the instructor's PC, or a shared folder on a Wi-Fi router in the room).

### 1.4 Check each learner's PC meets the requirements

- Windows 10 (version 2004 or newer) or Windows 11, 64-bit
- At least **10 GB** of free disk space
- Administrator rights on the PC (needed to turn on Windows features)
- **Virtualization enabled in the BIOS/UEFI** (see [Troubleshooting](#5-troubleshooting)). This is the most common cause of failure.

---

## 2. Instructor workflow during the session

1. Hand out the USB drives (or point learners to the network share).
2. Walk the room through **section 3** together, one step at a time, on the projector.
3. Do not move on until everyone has completed the step and, where noted, run the check.
4. Use the **Troubleshooting** table for anyone who gets stuck. Pair a stuck learner with a neighbour who is done, so the group keeps moving.

Suggested time: about 20–30 minutes for a group, plus a restart.

---

## 3. Learner steps (follow along with the instructor)

All commands below are run in **PowerShell as Administrator**:
*Start menu → type "PowerShell" → right-click → **Run as administrator**.*

### Step 1 — Copy the files to the PC

Copy the `WSL-offline` folder from the USB drive to the desktop (or any folder you can find easily).

In PowerShell, go into that folder. For example:

```powershell
cd $HOME\Desktop\WSL-offline
```

### Step 2 — Turn on the required Windows features

```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

Both commands should finish with *"The operation completed successfully."*

### Step 3 — Restart the PC

```powershell
shutdown /r /t 0
```

After the restart, open **PowerShell as Administrator** again and `cd` back into the `WSL-offline` folder.

### Step 4 — Install the WSL package

Double-click the `wsl.3.0.5.0.x64.msi` file and click **Next** through the installer. Or from PowerShell:

```powershell
msiexec /i .\wsl.3.0.5.0.x64.msi
```

(Use the full file name if the wildcard does not work.)

### Step 5 — Install Ubuntu from the image

Either double-click the `.wsl` file, or run:

```powershell
wsl --install --from-file .\ubuntu-24.04.5-wsl-amd64.wsl
```

(Replace the file name with the exact name of your file.)

### Step 6 — Create your Linux user

When Ubuntu opens for the first time it asks you to create a **username** and **password**.

- Pick a short lowercase username (no spaces).
- The password does not show as you type. This is normal.
- **Write the password down.** You need it for `sudo`.

### Step 7 — Check that it works

In the Ubuntu window:

```bash
whoami
lsb_release -a
```

Expected: your username, and `Ubuntu 24.04` in the release information.

Back in PowerShell, confirm it is running as **WSL 2**:

```powershell
wsl -l -v
```

The `VERSION` column for Ubuntu should show `2`.

### Done

Learners can open Ubuntu any time from the Start menu, or by typing `wsl` in PowerShell or Windows Terminal.

---

## 4. Fallbacks

### Fallback A: if the `.wsl` image will not install

An older WSL version may not support `.wsl` files. If you are not offline for a moment, `wsl --update` fixes this. Otherwise, use [Fallback B](#fallback-b-import-a-tarball).

### Fallback B: import a tarball

Use the `ubuntu-noble-wsl-amd64-wsl.rootfs.tar.gz` file from the cloud-images site:

```powershell
mkdir C:\WSL\Ubuntu-24.04
wsl --import Ubuntu-24.04 C:\WSL\Ubuntu-24.04 .\ubuntu-noble-wsl-amd64-wsl.rootfs.tar.gz
wsl -d Ubuntu-24.04
```

Imported this way, the distro starts as **root**, so create a normal user first:

```bash
adduser yourname
usermod -aG sudo yourname
exit
```

Then make that user the default (back in PowerShell):

```powershell
wsl -d Ubuntu-24.04 -u yourname
```

### Fallback C: use the browser-based Linux terminal instead

If a particular PC cannot run WSL (no virtualization support, no admin rights, very old Windows), the learner can use the shared training JupyterHub in their browser and open a **Terminal** there. They only need a working browser and the login details you give them:

```
https://<your-training-hub-address>
```

---

## 5. Troubleshooting

| Symptom | Likely cause | What to do |
|---------|--------------|------------|
| Error **`0x80370102`** when starting Ubuntu | Virtualization is turned off in the BIOS/UEFI | Restart, enter the BIOS/UEFI (usually F2, F10, F12, Del or Esc at startup), enable **Intel VT-x / AMD-V / SVM** (sometimes called "Virtualization Technology"), save and exit |
| Error **`0x800701BC`** | WSL package/kernel is outdated or missing | Install the newest WSL MSI from the USB drive (Step 4) and retry |
| `wsl` is not recognised as a command | The features are not enabled, or the PC has not restarted | Repeat Step 2, then restart (Step 3) |
| `dism.exe` says "Access denied" or "requires elevation" | PowerShell was not run as Administrator | Close it and reopen as Administrator |
| `wsl --install --from-file` says the option is unknown | WSL is too old | Install the newest WSL MSI (Step 4), or use Fallback B |
| Installer hangs at 0% with "Downloading…" | `wsl --install` without `--from-file` tries to download | Always use `--from-file` with the local image, or use Fallback B |
| Ubuntu opens as `root` and never asks for a username | Distro was imported from a tarball | Create a user as shown in Fallback B |
| "Not enough disk space" | Less than ~10 GB free | Free space or install to another drive via `wsl --import` (Fallback B) |
| Windows Home edition | Nothing special needed | WSL 2 works on Windows 11 Home; no Hyper-V setup required |

---

## 6. Bioinformatics tools offline (packed conda environment)

A fresh Ubuntu on WSL has none of the course software, and installing it over a poor connection will fail the same way the Ubuntu download did. Instead, build the environment **once**, pack it into a single file with [`conda-pack`](https://conda.github.io/conda-pack/), and distribute it by USB.

### 6.1 Compatibility: same OS and CPU architecture

A packed environment only works on the **same operating system and CPU architecture** it was built on.

| Built on | Works on | Does not work on |
|----------|----------|------------------|
| Linux x86_64 (`linux-64`), for example a standard cloud VM | **WSL on a normal (x64) Windows PC**, and any x86_64 Linux machine | Windows ARM laptops (Snapdragon), Apple Silicon Macs, native Windows or macOS (outside WSL) |

- Check the build machine's architecture: `uname -m` should print `x86_64`.
- For an ARM laptop you would need an environment built on an ARM (`aarch64`) Linux machine, and some bioinformatics packages are not available for ARM. For those learners, use [Fallback C](#fallback-c-use-the-browser-based-linux-terminal-instead).
- Build on a Linux machine, not on Windows. The packed environment is only used inside WSL.

### 6.2 Pack the environment (instructor, once, on the build machine)

This assumes the environment is called `bioinfo`, installed at `/opt/tljh/user/envs/bioinfo`. Change the path if yours differs.

Install `conda-pack` into a **separate throwaway environment**, never into `bioinfo` itself, or it would end up inside the pack:

```bash
uname -m                                   # expect: x86_64
sudo -E /opt/tljh/user/bin/conda create -y -p /tmp/packtool -c conda-forge conda-pack
```

Pack it. This can take several minutes and produces a large file, so run it in `tmux` or `screen`:

```bash
sudo /tmp/packtool/bin/conda-pack -p /opt/tljh/user/envs/bioinfo -o /tmp/bioinfo.tar.gz
ls -lh /tmp/bioinfo.tar.gz
sudo rm -rf /tmp/packtool                  # clean up the throwaway env
```

Notes:

- Anything stored **inside** the environment folder is packed with it. For example, Krona's taxonomy data (in `opt/krona/taxonomy`) travels with the environment.
- Anything stored **outside** it does not. For example, a Kraken2 database kept in `/opt/kraken_db` must be copied separately (see 6.3).
- If `conda-pack` reports a problem, read the message. It names the package. It most often means a package was changed after installation, and rebuilding the environment from scratch fixes it.

Copy the file off the build machine to your laptop (for example with `scp`), then onto the USB drives.

### 6.3 Put these on the USB drives (next to the Ubuntu image)

```
WSL-offline/
├── ubuntu-24.04.5-wsl-amd64.wsl
├── wsl.3.0.5.0.x64.msi
├── bioinfo.tar.gz                     (the packed environment)
├── k2_viral_YYYYMMDD.tar.gz           (Kraken2 viral index, the original archive)
└── training_data/                     (reference genome, sample reads, exercises)
```

Verify the size and checksum of each copy (`sha256sum` on Linux, `Get-FileHash` on Windows) so a corrupted copy doesn't surprise you in the room.

### 6.4 Learner steps (inside Ubuntu on WSL)

A Windows drive appears in WSL under `/mnt`. A USB drive that is `D:` in Windows is `/mnt/d` in Ubuntu. Adjust the letter to match the drive.

**1. Unpack the environment:**

```bash
mkdir -p ~/bioinfo
tar -xzf /mnt/d/WSL-offline/bioinfo.tar.gz -C ~/bioinfo
source ~/bioinfo/bin/activate
conda-unpack
```

`conda-unpack` fixes the file paths inside the environment so it works from its new location. Run it **once**, right after unpacking.

**2. Unpack the Kraken2 viral database:**

```bash
mkdir -p ~/kraken_db/viral
tar -xzf /mnt/d/WSL-offline/k2_viral_YYYYMMDD.tar.gz -C ~/kraken_db/viral
ls ~/kraken_db/viral                      # expect hash.k2d, opts.k2d, taxo.k2d
```

**3. Point Kraken2 at the new location.** The packed environment may still contain a setting for the old server path, so replace it:

```bash
mkdir -p ~/bioinfo/etc/conda/activate.d
echo 'export KRAKEN2_DEFAULT_DB="$HOME/kraken_db/viral"' > ~/bioinfo/etc/conda/activate.d/kraken_db.sh
```

**4. Copy the training data:**

```bash
cp -r /mnt/d/WSL-offline/training_data ~/training
```

**5. Activate it every time you open a terminal** (optional: do it automatically):

```bash
source ~/bioinfo/bin/activate
# optional, to activate automatically at every login:
echo 'source ~/bioinfo/bin/activate' >> ~/.bashrc
```

### 6.5 Check that everything works

```bash
source ~/bioinfo/bin/activate
samtools --version | head -n 1
bwa 2>&1 | head -n 3
fastqc --version
kraken2 --version | head -n 1
echo $KRAKEN2_DEFAULT_DB
```

Each command should print a version or usage line, and the last one should print `/home/<you>/kraken_db/viral`.

### 6.6 Memory note

WSL uses part of the PC's RAM. Large reference genomes can need several GB to index. If a learner's PC has only 8 GB of RAM, give them a small reference or a pre-built index instead of making them build one. Check this on a spare PC before the session.

---

## 7. Instructor checklist

**Before the session**

- [ ] Downloaded the Ubuntu `.wsl` image and the WSL `.msi` (x64; arm64 if needed)
- [ ] Verified the Ubuntu image checksum
- [ ] Copied the `WSL-offline` folder to every USB drive (or set up the network share)
- [ ] Confirmed which learners have admin rights and virtualization enabled
- [ ] Tested the full steps once on a spare PC
- [ ] Have the browser-based Linux terminal (Fallback C) ready as a backup
- [ ] Packed the conda environment on an x86_64 Linux machine and tested the unpack in WSL (section 6)
- [ ] Copied the packed environment, Kraken2 database and training data to every USB drive

**During the session**

- [ ] Everyone completed Step 2 (features) and restarted
- [ ] Everyone completed Step 4 (WSL package)
- [ ] Everyone completed Step 5 (Ubuntu installed)
- [ ] Everyone created a user and passed the Step 7 check (`wsl -l -v` shows version 2)

---

## References

- Ubuntu on WSL: install guide — <https://ubuntu.com/wsl/docs/stable/howto/install-ubuntu-wsl2/>
- Ubuntu on WSL: download and customise an image — <https://ubuntu.com/wsl/docs/latest/howto/custom-ubuntu-distro/>
- Microsoft: install WSL, including the offline install — <https://learn.microsoft.com/windows/wsl/install>
- WSL releases (MSI installers) — <https://github.com/microsoft/WSL/releases>
- Ubuntu releases (`.wsl` images) — <https://releases.ubuntu.com/noble/>
