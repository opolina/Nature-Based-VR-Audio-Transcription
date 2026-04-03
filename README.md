# Audio Transcription Tool

Transcribes audio files using WhisperX with word-level timestamps. Produces multiple export formats for temporal correlation with screen recordings.

## Prerequisites

- Python 3.11-3.13 (WhisperX does not support 3.14+)
- NVIDIA GPU with CUDA support (CPU fallback available but slow)
- ffmpeg installed and on PATH ([download](https://ffmpeg.org/download.html))

## Setup

```bash
# Create virtual environment with Python 3.13
# Using uv (https://docs.astral.sh/uv/), or replace with: python -m venv .venv
uv venv --python 3.13 .venv

# Activate (on Windows):
.venv\Scripts\activate
# For macOS/Linux:  source .venv/bin/activate

# Ensure pip is available in the virtual environment
python -m ensurepip --upgrade

# Install all dependencies (PyTorch, WhisperX, export libraries)
python -m pip install -r requirements.txt

# Verify
python -c "import torch; print('CUDA:', torch.cuda.is_available()); import whisperx; print('OK')"
```

## Usage

### Interactive mode (no arguments)

```bash
python transcribe.py
```

Opens a graphical interface where you can select audio files, choose export formats, and configure options. Settings (output directory, model, formats) are remembered between sessions.

### Command Line Usage (Single File)

```bash
python transcribe.py recording.wav
```

### Batch (all audio files in a directory)

```bash
python transcribe.py --batch audio_files/ -o results/
```

### CLI Options

| Flag | Default | Description |
|------|---------|-------------|
| `-o`, `--output-dir` | script directory | Output directory for exported files |
| `-m`, `--model` | `large-v3` | Whisper model size |
| `-l`, `--language` | `en` | Language code |
| `--batch-size` | `16` | Reduce to 4–8 if GPU runs out of memory |
| `--device` | auto | Force `cuda` or `cpu` |
| `--compute-type` | auto | Force `float16` or `int8` |
| `--formats` | `csv_segments txt xlsx` | Export formats (see below) |

## Export Formats

For each input file (e.g., `participant_01.wav`), the following outputs can be generated:

**Segment-level:**

| Format | File | Description |
|--------|------|-------------|
| `txt` | `participant_01.txt` | Plain transcript text, one segment per line |
| `csv_segments` | `participant_01_segments.csv` | One row per segment with start, end, duration, text |
| `docx` | `participant_01.docx` | Word document with full transcript and timestamped segment table |

**Word-level (includes segments):**

| Format | File | Description |
|--------|------|-------------|
| `csv_words` | `participant_01_words.csv` | One row per word with start, end, duration, confidence |
| `xlsx` | `participant_01.xlsx` | Excel workbook with Segments, Words, and Full Transcript sheets |

In Interactive mode, formats are selectable via checkboxes. In CLI mode, use the `--formats` flag:

```bash
python transcribe.py recording.wav --formats txt xlsx
```

All timestamps are in **seconds** (float).

The `score` field (0-1) in word-level outputs indicates timestamp confidence. Low scores suggest the word boundary may be imprecise.

## First Run

The first run downloads ~3 GB of model weights (Whisper large-v3 + wav2vec2 alignment model), thus it will be slow. Subsequent runs use the cached models so they will be signifantly faster.
