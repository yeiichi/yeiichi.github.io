---
layout: default
title: "Running Architecture-Specific Shell Commands on macOS"
permalink: /run-by-architecture-macos/
---

# Running Architecture-Specific Shell Commands on macOS
_Last Updated: 2026-07-24_

## 1. Introduction

A shell configuration may accumulate commands and scripts that work only on Intel Macs, while newer Macs use Apple Silicon. Repeating an architecture check around every command makes `.zshrc` noisy and difficult to maintain.

A small dispatcher function provides a simpler pattern: define one handler for Apple Silicon, another for Intel, and let a generic helper select the appropriate implementation.

The important design choice is to pass function names rather than complete commands stored as strings. This avoids `eval`, preserves normal shell quoting, and allows each handler to contain pipelines, redirections, or multiple commands.

---

## 2. Generic Architecture Dispatcher

### A. Architecture Detection

Place the following helper in a file such as `~/.ztools`:

```zsh
is_apple_silicon() {
    [[ $(sysctl -n hw.optional.arm64 2>/dev/null) == 1 ]]
}
```

`sysctl` returns `1` when the Mac supports the ARM64 architecture used by Apple Silicon. The function returns a normal shell success or failure status, so it can be used directly in an `if` statement.

---

### B. Dispatcher Function

Add the generic dispatcher to the same file:

```zsh
run_by_architecture() {
    local arm64_handler=$1
    local intel_handler=$2
    shift 2

    if is_apple_silicon; then
        "$arm64_handler" "$@"
    else
        "$intel_handler" "$@"
    fi
}
```

The two arguments are function names:

* **`arm64_handler`**: the function to run on Apple Silicon.
* **`intel_handler`**: the function to run on an Intel Mac.

Quoting the selected handler variable executes the named function without interpreting it as arbitrary shell code. Any additional arguments are forwarded to the selected handler.

---

## 3. Example: Reading CPU Temperature

Define one function for each architecture:

```zsh
dietemp_arm64() {
    sudo powermetrics --samplers thermal,cpu_power
}

dietemp_intel() {
    sudo powermetrics --samplers smc |
        grep -i 'die temp'
}
```

Then define the public command:

```zsh
dietemp() {
    run_by_architecture dietemp_arm64 dietemp_intel "$@"
}
```

Running the following command now selects the correct implementation automatically:

```console
% dietemp
```

The same structure can be reused for other utilities:

```zsh
example_arm64() {
    command_for_apple_silicon
}

example_intel() {
    command_for_intel
}

example() {
    run_by_architecture example_arm64 example_intel "$@"
}
```

Finally, load the helper file from `.zshrc`:

```zsh
source "$HOME/.ztools"
```

---

## 4. Why Use Functions Instead of Command Strings?

A tempting design is to pass complete commands as quoted strings and execute them with `eval`. That approach becomes fragile when commands contain spaces, quotes, pipelines, redirections, or user-supplied values.

Command substitution, written as `$(...)`, is not a replacement for `eval`. It runs a command and captures its output. It is intended for expressions such as:

```zsh
machine_name=$(hostname)
```

It is not intended to dispatch a command stored as text.

Functions avoid both problems. Each architecture-specific implementation remains ordinary shell code, while the dispatcher handles only the choice of which function to invoke.

The resulting pattern has three useful properties:

* **Readable:** each implementation is visible as normal shell code.
* **Safe:** no shell source code is reconstructed and evaluated from a string.
* **Extensible:** handlers may contain pipelines, redirections, multiple commands, or additional checks.

---

## 5. Maintenance Notes

- **Naming:** Use a consistent suffix such as `_arm64` and `_intel` for architecture-specific handlers.
- **Placement:** Keep reusable helpers and command functions in `~/.ztools`, and keep `.zshrc` focused on loading configuration.
- **Scope:** Use this dispatcher when both architectures need different implementations. A command that already works identically on both architectures does not need a wrapper.
- **Detection target:** This checks the Mac's hardware capability, not the architecture of the current shell process. Use a process-level check such as `uname -m` if you need Rosetta-aware behavior.
- **Portability:** This helper is macOS-specific because it relies on `sysctl -n hw.optional.arm64`.
- **Testing:** Test each handler directly before testing the public wrapper, for example `dietemp_arm64` or `dietemp_intel`.
