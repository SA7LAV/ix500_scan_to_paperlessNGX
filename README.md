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

### 2. Fujitsu ScanSnap iX500 firmware

The iX500 requires proprietary firmware blobs that are not included in SANE. Install the `sane-airscan` package (for network scanning) or the firmware package from the AUR:

```bash
# AUR package with Fujitsu firmware files
yay -S sane-airscan

# Alternatively, for USB-only with the classic SANE Fujitsu backend:
yay -S pfufs-scansnap-ix500
```

> **Note:** The firmware file `1300_0C26.nal` must be present in `/usr/share/sane/epjitsu/` for USB mode. The AUR package handles this automatically.

### 3. img2pdf

Used to combine scanned TIFF pages into a single PDF without re-encoding (lossless).

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
- USB cable connection
- `lsusb` should show `04c5:132b` (Fujitsu iX500)
- Firmware is installed correctly
- User is in the `scanner` group

### Summary of required packages

| Package | Source | Purpose |
|---------|--------|---------|
| `sane` | pacman | Scanner backend (scanimage) |
| `img2pdf` | pacman | Lossless TIFF → PDF conversion |
| `pfufs-scansnap-ix500` or `sane-airscan` | AUR | Fujitsu firmware / driver |

## Installation

```bash
# Copy the script to a directory in your PATH
cp scan-to-paperless ~/bin/
chmod +x ~/bin/scan-to-paperless

# Make sure ~/bin is in your PATH (add to ~/.bashrc or ~/.config/fish/config.fish)
export PATH="$HOME/bin:$PATH"
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
2. Each page is saved as a TIFF file in a temporary directory
3. `img2pdf` combines all pages into a single PDF (lossless, no re-encoding)
4. The PDF is placed in `~/paperless-consume/`
5. Paperless-ngx automatically detects and ingests the new file

## Paperless-ngx integration

The script outputs PDFs to `~/paperless-consume/`, which should be configured as the consume directory in your paperless-ngx instance. See the [paperless-ngx consumption docs](https://docs.paperless-ngx.com/configuration/#paths-and-filenames) for setup details.

## License

Copyright (c) 2026 Kalle Hagen

Licensed under [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/) — free to use, share, and adapt for non-commercial purposes.
