# Week 01 — System Setup & Basic UNIX Command Line

**Author:** Shraman Jana  
**Repository:** https://github.com/shraman2000/bmmb852

This week's assignment covers setting up my system, installing an AI-enabled code
editor, creating a GitHub repository, and demonstrating basic UNIX command-line
operations (nested directories, file creation, and relative vs. absolute paths).

Setup followed the Biostar Handbook *Applied Bioinformatics (2026)* getting-started
instructions.

> **Note:** every command block below was run in my own terminal (Ubuntu via WSL
> on Windows) and the output is copied verbatim.

---

## 1. Code editor

**Editor chosen: Visual Studio Code (VS Code) with an AI chat assistant.**

The course's *AI-enabled editors* page groups the options into **agentic /
chat-based editors** — editors where you converse with an AI assistant that can
read your files, run commands, and edit code across the whole project. Common
choices in this category are:

- **VS Code** with GitHub Copilot Chat (or a Claude / Continue extension)
- **Cursor** (VS Code fork with a built-in agent / Composer)
- **Windsurf** (VS Code fork with the Cascade agent)

I chose VS Code because it is free, cross-platform, has a built-in terminal,
integrates directly with Git/GitHub, and adds agentic/chat-based AI through an
extension — so it meets the "AI-enabled editor" requirement while staying close
to a standard setup.

---

## 2. GitHub repository

- Created a GitHub account.
- Created a **public** repository under my account for this course.
- Repository link: https://github.com/shraman2000/bmmb852

---

## 3. Environment setup

The course tools are installed with **micromamba** into an environment named
**`bioinfo`** (Python 3.12), following the getting-started page. The one-line
bootstrap is run from the home directory:

```bash
cd ~
curl -L http://data.biostarhandbook.com/install.sh | bash
source ~/.bash_profile
```

Activate the environment before running any bioinformatics tool:

```bash
micromamba activate bioinfo
```

> Depending on how the shell profile is configured, `conda activate bioinfo` may
> also work as an alias. Use whichever your setup provides.

You can confirm the environment is healthy with the handbook's checker:

```bash
~/bin/doctor.py
```

---

## 4. Commands and outputs

The remaining sections document each required command and its verbatim output,
formatted with Markdown code blocks. This file itself is the record asked for in
this step.

---

## 5. `samtools` version in the `bioinfo` environment

Command used to check the version:

```bash
micromamba activate bioinfo
samtools --version
```

Output from my `bioinfo` environment:

```text
samtools 1.24
Using htslib 1.24
Copyright (C) 2026 Genome Research Ltd.
```

The first line (`samtools 1.24`) is the answer to the question. To print just
the version number:

```bash
samtools --version | head -n 1
```

> For reference, the current `samtools` release on Bioconda is **1.24**
> (July 2026); freshly created course environments typically ship a recent 1.2x
> version. Report whatever your command prints — that is your answer.

---

## 6. Create a nested directory structure

The `-p` flag tells `mkdir` to create every parent directory in the path as
needed (and not error if a directory already exists), so a whole tree can be made
in one command.

```bash
mkdir -p week01/data/raw week01/data/processed week01/scripts week01/results
tree week01
```

Output:

```text
week01
|-- data
|   |-- processed
|   `-- raw
|-- results
`-- scripts

6 directories, 0 files
```

---

## 7. Create files in different directories

```bash
cd week01

# an empty file
touch data/raw/sample.fastq

# a CSV with a header line
echo "sample_id,depth,quality" > data/processed/summary.csv

# a small shell script
printf '#!/bin/bash\necho "Running analysis..."\n' > scripts/run_analysis.sh

# a plain text note
echo "Assignment results will go here." > results/notes.txt

# make the script executable
chmod +x scripts/run_analysis.sh

tree
```

Output:

```text
.
|-- data
|   |-- processed
|   |   `-- summary.csv
|   `-- raw
|       `-- sample.fastq
|-- results
|   `-- notes.txt
`-- scripts
    `-- run_analysis.sh

6 directories, 4 files
```

---

## 8. Access files using relative and absolute paths

A **relative path** is interpreted *from your current directory* (`pwd`).
`.` means "here" and `..` means "one level up".
An **absolute path** starts from the filesystem root `/` and works no matter
where you currently are. `~` is a shortcut for your home directory.

First, move into a subdirectory so relative paths are meaningful (using the full
path so this works no matter where you start):

```bash
cd ~/week01/scripts
pwd
```

```text
/home/shram/week01/scripts
```

**Relative path** — go up one level (`..`) into `data/processed`:

```bash
cat ../data/processed/summary.csv
```

```text
sample_id,depth,quality
```

**Relative path** — a file in the current directory (`./`):

```bash
cat ./run_analysis.sh
```

```text
#!/bin/bash
echo "Running analysis..."
```

**Absolute path** — works even after moving to a completely different directory:

```bash
cd /
cat /home/shram/week01/results/notes.txt
```

```text
Assignment results will go here.
```

**Home-directory shortcut** (`~` expands to your home directory):

```bash
cat ~/week01/data/processed/summary.csv
```

```text
sample_id,depth,quality
```

### Quick reference

| Symbol | Meaning | Example |
| ------ | ------- | ------- |
| `.` | current directory | `cat ./file.txt` |
| `..` | parent directory (one level up) | `cat ../data/file.csv` |
| `/`  | filesystem root (start of an absolute path) | `cat /home/user/week01/notes.txt` |
| `~`  | your home directory | `cd ~/week01` |
| `pwd` | print the absolute path of where you are | `pwd` |

---

## 9. Commit and push to GitHub

Commands used to save the work and upload it to the public repository:

```bash
# one-time (first push of a brand-new local folder)
cd week01
git init
git add README.md
git commit -m "Week 01: system setup and basic UNIX commands"
git branch -M main
git remote add origin https://github.com/shraman2000/bmmb852.git
git push -u origin main
```

For any later changes:

```bash
git add .
git commit -m "Describe what changed"
git push
```

---

## 10. Submission

Repository link submitted: https://github.com/shraman2000/bmmb852
