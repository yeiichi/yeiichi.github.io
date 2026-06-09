---
layout: default
title: "XDG Base Directories Cheat Sheet"
permalink: /xdg-base-directories-cheat-sheet
---

# XDG Base Directories Cheat Sheet

*Last Updated: 2026-06-09*

## 1. Introduction

The XDG Base Directory Specification defines conventional locations for user-specific configuration, cache, and application data on Linux and other Unix-like systems.

Understanding these directories helps keep the home directory tidy, simplifies backups, and improves portability between systems. Modern command-line tools and desktop applications increasingly follow these conventions.

For developers working across Linux, macOS, Docker containers, and remote servers, adopting XDG-style paths can reduce platform-specific complexity.

---

## 2. Data Schema

### A. Column Definitions

* **Directory**: Standard XDG directory.
* **Default Location**: Path used when the corresponding environment variable is unset.
* **Purpose**: Intended usage of the directory.

---

### B. Design Policy

* **User Scope:** XDG directories are user-specific and reside under the user's home directory.
* **Portable Configuration:** Applications should respect XDG environment variables when present.
* **Separation of Concerns:** Configuration, cache, and application data should be stored separately.

---

## 3. XDG Base Directory Reference

| Directory                              | Default Location | Purpose                                     |
| -------------------------------------- | ---------------- | ------------------------------------------- |
| XDG_CONFIG_HOME                        | ~/.config        | User configuration files                    |
| XDG_CACHE_HOME                         | ~/.cache         | Non-essential cached data                   |
| XDG_DATA_HOME                          | ~/.local/share   | User application data                       |
| XDG_STATE_HOME                         | ~/.local/state   | Persistent state data (logs, history, etc.) |
| HOME                                   | ~                | User home directory                         |

### Typical Examples

#### Configuration

```text
~/.config/git/
~/.config/pip/
~/.config/nvim/
~/.config/gh/
~/.config/whisper-smith/
```

#### Cache

```text
~/.cache/pip/
~/.cache/uv/
~/.cache/pytest/
~/.cache/matplotlib/
```

Cache files should be considered disposable and may be safely deleted when troubleshooting or reclaiming disk space.

#### Application Data

```text
~/.local/share/applications/
~/.local/share/fonts/
~/.local/share/whisper-smith/
```

This directory is intended for user data that should survive cache cleanup.

#### State Data

```text
~/.local/state/
```

Examples:

```text
~/.local/state/app.log
~/.local/state/history
```

State data is neither configuration nor cache.

---

## 4. Linux vs macOS Notes

Although macOS traditionally uses:

```text
~/Library/Application Support/
```

many modern cross-platform CLI tools use:

```text
~/.config/
```

on macOS as well.

For developer-oriented Python projects, the following convention works well:

| Environment             | Recommended Location  |
| ----------------------- | --------------------- |
| macOS Development       | ~/.config/<app-name>/ |
| Linux Development       | ~/.config/<app-name>/ |
| Linux Production Server | /etc/<app-name>/      |

Example:

```text
macOS:
~/.config/whisper-smith/secrets.env

Ubuntu Server:
/etc/whisper-smith/secrets.env
```

---

## 5. Secret Management Example

Recommended permissions:

```text
directory : 700
file      : 600
```

Example:

```bash
mkdir -p ~/.config/whisper-smith
chmod 700 ~/.config/whisper-smith
chmod 600 ~/.config/whisper-smith/secrets.env
```

Result:

```text
drwx------ ~/.config/whisper-smith
-rw------- ~/.config/whisper-smith/secrets.env
```

---

## 6. Maintenance Notes

* **Extensibility:** Additional XDG directories can be added as standards evolve.
* **Generation:** This page is maintained manually as a developer cheat sheet.
* **Interpretation:** The defaults shown assume the corresponding XDG environment variables are not explicitly set.
