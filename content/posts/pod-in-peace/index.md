---
title: "Pod in Peace — Podcast Archiver + Transcriber"
date: 2025-10-10
draft: false
slug: "podinpeace"
description: "A lightweight Python tool for downloading, normalizing, and transcribing podcasts — with GPU-accelerated Whisper support and a clean Tkinter interface."
tags: ["python", "audio", "ffmpeg", "whisper", "tkinter", "cli","archive"]
image: "/images/podinpeace-banner.png"
---

![Pod in Peace Screenshot](/images/podinpeace-ui.png)

---

**Pod in Peace** is a small, self-contained Python app for discovering, downloading, normalizing, and transcribing podcasts.  
It supports both a **Tkinter GUI** and a **command-line interface**, letting you archive entire feeds or transcribe individual episodes in a few clicks.

---

## Features

- Download full podcast feeds or only the latest episodes.  
- Skip episodes by keyword, duration, or duplication.  
- Optional FFmpeg-based loudness normalization.  
- GPU-accelerated transcription powered by [faster-whisper](https://github.com/SYSTRAN/faster-whisper).  
- Built-in search dialog for discovering feeds.  
- Background task queue keeps the interface responsive.

---

## Installation

```bash
git clone https://github.com/mgelsinger/podinpeace.git
cd podinpeace
python -m pip install -r requirements.txt
```

### Optional GPU Transcription

Enable CUDA acceleration (Windows or Linux):

```bash
python -m pip install faster-whisper nvidia-cudnn-cu12 nvidia-cublas-cu12 nvidia-cuda-runtime-cu12
```

If you store CUDA DLLs outside standard locations, define:

```bash
set PODINPEACE_CUDNN_PATH=C:\path\to\cudnn
```

---

## Quick Start

### GUI Mode

```bash
python download_podcast.py
```

Use the graphical interface to:
- Search or paste podcast RSS feeds.
- Choose a download folder.
- Enable normalization or transcription.
- Track progress via the integrated log pane.

### CLI Mode

```bash
python download_podcast.py <feed_url> <output_directory> [options]
```

Example:

```bash
python download_podcast.py https://feed.skeptoid.com ./Skeptoid --limit 10 --transcribe --transcribe-model large-v2
```

| Option | Description |
|--------|--------------|
| `--limit <N>` | Download only the latest N episodes |
| `--exclude <keywords>` | Skip episodes containing listed keywords |
| `--min-length <minutes>` / `--max-length <minutes>` | Restrict by duration |
| `--transcribe` | Generate transcripts |
| `--transcribe-model <name>` | Override the default Whisper model |
| `--transcribe-device {auto,cuda,cpu}` | Choose or auto-detect backend |

---

## Structure

```
podinpeace/
 ├─ audio/        # Normalization + transcription
 ├─ feed/         # Downloading and metadata
 ├─ gui/          # Tkinter interface
 ├─ tasks/        # Background queue and progress
 └─ utils/        # Shared helpers
```

---

## Philosophy

Most podcast managers are cloud-tied or bloated.  
**Pod in Peace** aims for **local-first, minimal, and transparent** design — everything happens on your machine.  
It’s an ideal starting point for anyone building offline podcast workflows, speech datasets, or searchable audio archives.

[→ View the source on GitHub](https://github.com/mgelsinger/podinpeace)

---

## Next Steps

Planned updates include:
- Multi-feed batch downloads  
- Automated transcript tagging and search  
- Optional web interface (FastAPI prototype)

---

_“Pod in Peace” — Take your listening offline, and your transcripts anywhere._
