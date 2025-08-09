# encryptic

**Bash utility for PGP encryption, SHA-256/SHA-512 checksums, and SINPE TSA timestamps — with subkey selection and a one-command full mode.**

> Version: **2.0.0** • Language: Bash • Platforms: Debian-based Linux • Shell: Bash 4+

---

## Table of contents

- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Quick start](#quick-start)
- [Usage](#usage)
  - [Synopsis](#synopsis)
  - [Commands](#commands)
  - [Global options](#global-options)
  - [Outputs](#outputs)
  - [Examples](#examples)
- [Configuration](#configuration)
- [Security notes](#security-notes)
- [Platform support](#platform-support)

---

## Features

- List **encryption subkeys** (`[E]`) and **select a specific subkey** to encrypt with.
- **Four modes** per file:  
  `-c` Encrypt (ASCII-armored `.asc`) • `-h` Hash (SHA-256/512) • `-t` Timestamp (SINPE TSA) • `-f` Full (all three; **default**).
- Hash algorithm selectable: **SHA-512** (default) or **SHA-256** via flag or environment variable.
- Sensible safety/UX defaults:
  - Uses `--` before file paths to avoid flag/filename confusion.
  - Refuses to overwrite outputs (no `--force` by default).
  - `curl --fail` and basic `.tsr` size checks for TSA requests.
  - Config stored under `~/.config/encryptic/`.

---

## Requirements

- **Debian-based Linux** distribution (e.g., Debian, Ubuntu, Mint)
- **Bash 4+**
- Tools: **gpg**, **openssl**, **curl**

Install dependencies on Debian/Ubuntu:

```bash
sudo apt update && sudo apt install -y gnupg openssl curl
````

> macOS is not officially supported, but the script contains a fallback to `shasum` which may help in non-Linux environments.

---

## Installation

```bash
# 1) Make the script executable
chmod +x encryptic

# 2) Move it into your PATH
sudo mv encryptic /usr/local/bin/

# 3) Verify
encryptic -v
```

---

## Quick start

```bash
# 1) List encryption subkeys [E] and select one
encryptic -l
encryptic -s 2

# 2) Run the full workflow (encrypt + hash + timestamp)
encryptic -e MyFile.pdf
```

---

## Usage

### Synopsis

```bash
encryptic [--sha256|--sha512] [-l | -s <index> | -e <file> [-c|-h|-t|-f] | -v | -h]
```

### Commands

* `-l, --list`
  List available **encryption subkeys** `[E]` (numbered). Output shows the UID, subkey fingerprint (FPR), parent public key, and caps.

* `-s, --select <index>`
  Select the **subkey** by number from `-l`. The selection is saved to:

  ```
  ~/.config/encryptic/config   # stores: SUBKEY_FPR=<fingerprint>
  ```

* `-e, --encrypt <file> [-c|-h|-t|-f]`
  Operate on `<file>` with one of the modes below. If you omit the mode, **`-f`** is used by default.

  * `-c` — **Encrypt only**: ASCII-armored PGP → `<file>.asc`
  * `-h` — **Hash only**: SHA-256/512 checksum → `<file>.sha256` or `<file>.sha512`
  * `-t` — **Timestamp only** (SINPE TSA): request/reply → `<file>.tsq`, `<file>.tsr`
  * `-f` — **Full**: runs `-c`, `-h`, and `-t` in sequence (**default**)

* `-v, --version` — Print version

* `-h, --help` — Show help

### Global options

* `--sha512` / `--sha256`
  Choose the hashing algorithm (affects `-h` and `-t`). Default: **SHA-512**.
  You can also set:

  ```bash
  export ENCRYPTIC_HASH_ALG=sha512   # or sha256
  ```

* `ENCRYPTIC_TSA_URL`
  Override the SINPE TSA endpoint (default baked into the script):

  ```bash
  export ENCRYPTIC_TSA_URL="http://tsa.sinpe.fi.cr/tsaHttp/"
  ```

### Outputs

* **Encrypt (`-c`)** → `<file>.asc`
  ASCII armored; first line must be:

  ```
  -----BEGIN PGP MESSAGE-----
  ```
* **Hash (`-h`)** → `<file>.sha512` (or `.sha256`) containing:
  `128-hex-digest  <two spaces>  filename`
* **Timestamp (`-t`)** → `<file>.tsq` (request) and `<file>.tsr` (reply)

> The timestamp is produced **over the original file** (not the `.asc`).

### Examples

```bash
# 1) List subkeys and select one
encryptic -l
encryptic -s 3

# 2) Full workflow (encrypt + hash + timestamp) — default mode
encryptic -e document.pdf

# 3) Encrypt only (ASCII-armored .asc)
encryptic -e document.pdf -c

# 4) Hash only with SHA-256
encryptic --sha256 -e document.pdf -h

# 5) Timestamp only (SINPE TSA)
encryptic -e document.pdf -t

# 6) Use environment variables
ENCRYPTIC_HASH_ALG=sha256 ENCRYPTIC_TSA_URL="http://tsa.sinpe.fi.cr/tsaHttp/" encryptic -e doc.pdf -f
```

---

## Configuration

`encryptic` stores your selected subkey fingerprint here:

```
~/.config/encryptic/config
# Example:
# SUBKEY_FPR=ABCD1234...F00D
```

To change the selected subkey, run `encryptic -l` again and `encryptic -s <index>`.

---

## Security notes

* This tool **does not** bypass GnuPG’s web-of-trust by default. That’s intentional.
  If you need non-interactive automation with untrusted keys, consider enabling GPG’s `--trust-model always` at your own risk.
* Outputs are **not overwritten**. If a target file exists, the operation aborts. This prevents accidental data loss.
* Config files are created with a restrictive `umask` and a random temp file prior to atomic move.

---

## Platform support

* Designed and tested for **Debian-based Linux** with **Bash 4+**.
* Uses `sha256sum`/`sha512sum` when available; falls back to `shasum` otherwise.
* Other platforms/shells are not officially supported.

