# scan-to-paperless

A bash script to scan documents from a **Fujitsu ScanSnap iX500** directly into a [paperless-ngx](https://docs.paperless-ngx.com/) consume directory. Scanned pages are automatically combined into a single PDF.

## Features

- Duplex and simplex scanning via ADF (Automatic Document Feeder)
- Color, grayscale, and black & white modes
- Configurable resolution (50–600 dpi)
- Custom filename prefixes for easy document organization
- Automatic multi-page PDF creation
- Scanned files are placed in `~/paperless-consume` for automatic paperless-ngx ingestion

## Prerequisites (Arch-based Linux)

### 1. SANE (Scanner Access Now Easy)

SANE provides the backend for scanner communication.

```bash
sudo pacman -S sane
```

### 2. No firmware needed

The iX500 is driven by SANE's built-in `fujitsu` backend over USB. It does **not** need any firmware files or AUR packages (the firmware-based `epjitsu` backend is only for older models such as the S300/S1300/S1100).

### 3. img2pdf

Used to combine the scanned pages (JPEG for Color/Gray, TIFF for Lineart) into a single PDF without re-encoding.

```bash
sudo pacman -S img2pdf
```

### 4. User permissions

Your user must be in the `scanner` group to access USB scanner devices:

```bash
sudo usermod -aG scanner $USER
```

Log out and back in (or `newgrp scanner`) for the group change to take effect.

### 5. Verify scanner detection

Connect the iX500 via USB and verify SANE detects it:

```bash
# List all detected scanners
scanimage -L

# Expected output (similar to):
# device `fujitsu:ScanSnap iX500:5085' is a FUJITSU ScanSnap iX500 scanner
```

If the scanner is not found, check:
- The paper chute (top lid) is open — the iX500 only powers on and shows up on USB while it is open (blue button lit)
- USB cable connection
- `lsusb` should show `04c5:132b` (Fujitsu iX500)
- User is in the `scanner` group

If `scanimage -L` reports a different device name (e.g. a different number after `iX500:`), adjust `DEVICE=` at the top of the script.

### Summary of required packages

| Package | Source | Purpose |
|---------|--------|---------|
| `sane` | pacman | Scanner backend (scanimage) |
| `img2pdf` | pacman | Pages → PDF without re-encoding |

## Installation

```bash
# Copy the script to a directory in your PATH
cp scan-to-paperless ~/bin/
chmod +x ~/bin/scan-to-paperless

# Make sure ~/bin is in your PATH
# bash (add to ~/.bashrc):
export PATH="$HOME/bin:$PATH"
# fish (run once, persists):
fish_add_path ~/bin
```

## Usage

```
scan-to-paperless [OPTIONS]

Options:
  -s simplex|duplex      Scan mode (default: duplex)
  -m Color|Gray|Lineart  Color mode (default: Color)
  -r RESOLUTION          DPI resolution 50-600 (default: 300)
  -n PREFIX              Filename prefix (default: scan)
  -h                     Show help
```

## Examples

```bash
# Default: Duplex, Color, 300 dpi
scan-to-paperless

# Text-only documents — smallest file size
scan-to-paperless -m Lineart -r 150

# Letters and forms — good balance between quality and size
scan-to-paperless -m Gray -r 200

# Single-sided grayscale scan
scan-to-paperless -m Gray -r 200 -s simplex

# High-resolution color photo scan
scan-to-paperless -r 600 -n "photo"

# Colored receipts with default quality
scan-to-paperless -m Color -r 300 -n "receipt"
```

### Choosing the right settings

| Use case | Mode | Resolution | Relative file size |
|----------|------|------------|--------------------|
| Text documents, contracts | Lineart | 150 dpi | ~smallest |
| Letters, forms, invoices | Gray | 200 dpi | ~small |
| General documents (default) | Color | 300 dpi | ~medium |
| Photos, detailed graphics | Color | 600 dpi | ~large |

> **Rule of thumb:** File size scales quadratically with resolution (600 dpi = 4× the pixels of 300 dpi) and linearly with color depth (Color ≈ 3× Gray ≈ 24× Lineart).

## How it works

1. The script calls `scanimage` with the configured options to scan all pages from the ADF
2. Each page is saved in a temporary directory — as JPEG for Color/Gray, as 1-bit TIFF for Lineart
3. `img2pdf` combines all pages into a single PDF, embedding the pages as-is (no re-encoding)

> **Why JPEG for Color/Gray?** Uncompressed TIFF pages get Flate-compressed by `img2pdf`, which barely shrinks scanner noise. A duplex Gray 200 dpi scan dropped from ~5 MB to ~1.1 MB after switching to JPEG.
4. The PDF is placed in `~/paperless-consume/`
5. Paperless-ngx automatically detects and ingests the new file

## Paperless-ngx integration

The script outputs PDFs to `~/paperless-consume/`, which should be configured as the consume directory in your paperless-ngx instance. See the [paperless-ngx consumption docs](https://docs.paperless-ngx.com/configuration/#paths-and-filenames) for setup details.

## License

Copyright (c) 2026 Kalle Hagen

Licensed under [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/) — free to use, share, and adapt for non-commercial purposes.
