---
layout: default
title: "Recipe: The Clean EDA Rig"
permalink: /eda-rig
---


# Recipe

A Declarative, XDG-Compliant, "Headless-Ready" Analytics Station

## 1. Introduction: The Post-PEP 668 Reality

If you\'ve recently tried to `pip install` something globally on a
modern Linux distro, you've hit the **PEP 668** wall: *\"externally
managed environment.\"* The days of polluting system Python are over.

### The Pain Points

-   **The Bloat:** Every time you start a new analysis, you create a new
    `.venv`. Suddenly, you have 15 copies of **Pandas** and **NumPy**
    eating gigabytes of disk space.
-   **The Mess:** Your home directory (`~`) looks like a junk drawer of
    `.venv`, `.ipynb_checkpoints`, and `.local` folders.
-   **The Loss of Context:** Moving between machines means manually
    rebuilding environments from memory.

## 2. Managing Environments with UV and XDG

We solve this by combining the **XDG Base Directory Spec** (keep it
clean) with **uv** (keep it fast).

-   **XDG Compliance:** We hide our environment in
    `~/.local/share/venvs`, keeping your home directory pristine.
-   **uv:** Replacing `pip` and `venv` with a single binary that is
    10x-100x faster and uses a global cache to save disk space.

## 3. Create the Hidden EDA Project

Instead of a \"dumb\" virtual environment, we create a **declarative
project** inside your XDG data folder.

``` bash
# 1. Create the XDG-compliant parent directory
mkdir -p ~/.local/share/venvs
cd ~/.local/share/venvs

# 2. Initialize the 'eda' project
uv init eda
cd eda

# 3. Add your EDA heavy-hitters
# This creates a pyproject.toml and a lockfile automatically
uv add jupyterlab pandas numpy matplotlib seaborn
```

Now, your environment isn\'t just a folder of binaries \-- it\'s a
defined **spec** that can be recreated anywhere.

## 4. Alias Me!

To avoid navigating to this hidden directory every time, we create a
\"sticky slip\" in your shell configuration (`~/.zshrc` or `~/.bashrc`).

Add this line to the bottom:

``` bash
# EDA Environment: Portable & Headless-ready
alias eda-lab='uv --project ~/.local/share/venvs/eda run jupyter-lab --no-browser'
```

### Why this works

-   **uv \--project:** Tells the system exactly where the environment
    lives, regardless of your current working directory.
-   **\--no-browser:** Makes the command work perfectly on headless
    servers via SSH (just copy the tokenized link).
-   **Self-documenting:** One look at your aliases tells you (or
    \"Future You\") exactly where the project is and how it\'s managed.
