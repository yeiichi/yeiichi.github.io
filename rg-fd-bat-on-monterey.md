---
layout: default
title: "Modern CLI Tools on a Half-Fossilized macOS Monterey"
permalink: /rg-fd-bat-on-monterey
------------------------------

# Modern CLI Tools on a Half-Fossilized macOS Monterey

*Last Updated: 2026-06-03*

## 1. Introduction

As macOS Monterey ages, modern package managers increasingly drop official support. While Homebrew may continue to function for some users, relying on it becomes progressively riskier.

Fortunately, several essential developer tools are written in Rust and can be installed directly via Cargo. This page documents a lightweight setup using:

* `ripgrep (rg)` — a fast replacement for `grep`
* `fd` — a user-friendly replacement for `find`
* `bat` — a syntax-highlighted replacement for `cat`

The goal is not to modernize Monterey itself, but to keep daily development productive on a platform that is slowly becoming a fossil.

---

## 2. Data Schema

### A. Tool Definitions

* **rg**: Fast recursive text search tool (replacement for `grep -R`)
* **fd**: Fast and intuitive file finder (replacement for `find`)
* **bat**: Syntax-highlighted file viewer (replacement for `cat`)

---

### B. Design Policy

* **Cargo First:** Install Rust-native tools using Cargo rather than Homebrew.
* **Minimal Dependencies:** Avoid introducing additional package managers.
* **Monterey Survival:** Favor approaches that continue working after official OS support ends.

---

## 3. Installation Summary

| Tool                                   | Purpose     | Install Command         |
| -------------------------------------- | ----------- | ----------------------- |
| rg                                     | Text search | `cargo install ripgrep` |
| fd                                     | File search | `cargo install fd-find` |
| bat                                    | File viewer | `cargo install bat`     |

---

## 4. Everyday Usage Cheat Sheet

| Traditional Command                     | Modern Equivalent  | Notes                 |
| --------------------------------------- | ------------------ | --------------------- |
| `grep -R TODO .`                        | `rg TODO`          | Faster and cleaner    |
| `grep -R SECRET_KEY .`                  | `rg SECRET_KEY`    | Common project search |
| `find . -name "*.py"`                   | `fd -e py`         | Simpler syntax        |
| `find . -name Dockerfile`               | `fd Dockerfile`    | Filename search       |
| `cat README.md`                         | `bat README.md`    | Syntax highlighting   |
| `find . -name "*.py" \| xargs grep foo` | `rg foo -g "*.py"` | One command           |

---

## 5. Why This Combination Works

The three tools complement each other naturally:

```bash
rg "TODO"
fd "*.py"
bat README.md
```

Typical workflow:

```bash
# Locate a file
fd settings

# Search inside files
rg "DJANGO_SETTINGS_MODULE"

# View the file
bat config/settings/local.py
```

This creates a simple mental model:

* `fd` finds files.
* `rg` finds text.
* `bat` displays content.

---

## 6. Monterey Notes

Verify that Cargo binaries are available:

```bash
echo $PATH | tr ':' '\n' | rg cargo
```

Expected output:

```text
/Users/<username>/.cargo/bin
```

Confirm installation:

```bash
which rg
which fd
which bat
```

Expected:

```text
~/.cargo/bin/rg
~/.cargo/bin/fd
~/.cargo/bin/bat
```

List installed Cargo packages:

```bash
cargo install --list
```

---

## 7. Maintenance Notes

* **Extensibility:** Additional Rust CLI tools such as `eza`, `zoxide`, and `dust` can be added later.
* **Generation:** This page is maintained manually as a personal operations memo.
* **Interpretation:** The objective is practical productivity on aging hardware and operating systems, not strict adherence to the latest ecosystem trends.

### Personal Observation

The `rg` + `fd` + `bat` combination has a tendency to iron out brain wrinkles.

Instead of remembering obscure combinations of `find`, `grep`, and `cat`, the workflow becomes:

```bash
fd something
rg something
bat something
```

Whether this is enlightenment or cognitive atrophy remains under investigation.
