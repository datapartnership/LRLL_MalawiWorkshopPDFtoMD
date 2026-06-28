The full pipeline: **record speech → ASR transcribes → NER finds entities** — a complete spoken-language understanding system in Chichewa.

---

## Quickstart — GitHub Codespaces

Everything is pre-installed. No local setup needed.

### Step 1 — Open a Codespace

1. Go to this repository on GitHub
2. Click the green **Code** button → **Codespaces** tab → **New with options**
3. Select **4-core · 16GB RAM · 32GB** under Machine type — required for NER and ASR training
4. You will be prompted for secrets (see Step 2)
5. Wait 60–90 seconds for the environment to build

### Step 2 — Add your secrets

When prompted during Codespace creation, enter:

| Secret | Required for | Where to get it |
|--------|-------------|----------------|
| `ANTHROPIC_API_KEY` | Session 1 (Claude notebook) | [console.anthropic.com](https://console.anthropic.com) |
| `HF_TOKEN` | Session 3 (ASR notebook) | [huggingface.co/settings/tokens](https://huggingface.co/settings/tokens) — Read access |

To add or update secrets later: [github.com/settings/codespaces](https://github.com/settings/codespaces)

### Step 3 — Verify the environment

Open the terminal (`Ctrl + `` `) and run:

```bash
python -c "import fitz, anthropic, pytesseract; print('Session 1 packages OK')"
python -c "import transformers, datasets; print('Session 2 packages OK')"
python -c "import torchaudio, librosa; print('Session 3 packages OK')"
```

### Step 4 — Run the notebooks in order

1. Open `pdf_text_extraction.ipynb` → Run All
2. Open `article_extraction_jupyter.ipynb` → Run All
3. Open `chichewa_ner_finetuning.ipynb` → Run Cell 1 first (downloads data), then Run All
4. Open `chichewa_asr_finetuning.ipynb` → Paste HF_TOKEN in Cell 1 → Run Cell 1 first, then Run All

---

## Memory management

NER and ASR training need at least 8GB free RAM. Before running Cell 4 in either notebook:

```bash
# Check available memory
nproc && free -h   # should show 4 cores and ~15GB total

# Kill Pylance if memory is low (it uses 3-4GB)
kill $(pgrep -f "pylance")
```

If training crashes, reduce `BATCH_SIZE` to 4 (NER) or 2 (ASR) in Cell 4.

---

## Local setup (alternative to Codespaces)

```bash
# System dependencies
sudo apt-get update && sudo apt-get install -y tesseract-ocr ffmpeg

# Python packages
pip install anthropic pymupdf pytesseract pillow jupyter \
    transformers "datasets==2.16.1" torch torchaudio \
    seqeval "accelerate>=1.1.0" librosa soundfile \
    jiwer evaluate ipykernel huggingface_hub

# Launch Jupyter
jupyter notebook
```

---

## Requirements summary

| Package | Session | Purpose |
|---------|---------|---------|
| `pymupdf` | 1 | PDF reading and rendering |
| `anthropic` | 1b | Claude API for article extraction |
| `pytesseract`, `pillow` | 1 | OCR fallback for image-only PDFs |
| `transformers` | 2, 3 | DistilmBERT and Whisper models |
| `datasets==2.16.1` | 2, 3 | Masakhane and Common Voice datasets |
| `torch`, `torchaudio` | 2, 3 | Deep learning framework |
| `seqeval` | 2 | NER evaluation (F1, precision, recall) |
| `librosa`, `soundfile` | 3 | Audio loading and processing |
| `jiwer`, `evaluate` | 3 | Word Error Rate calculation |
| `accelerate>=1.1.0` | 2, 3 | Required by HuggingFace Trainer |

---

## Repository structure

```
.
├── pdf_text_extraction.ipynb          # Session 1a — raw text extraction
├── article_extraction_jupyter.ipynb   # Session 1b — Claude article extraction
├── chichewa_ner_finetuning.ipynb      # Session 2 — NER fine-tuning
├── chichewa_asr_finetuning.ipynb      # Session 3 — ASR fine-tuning
├── TK1E2013_01_03_Page05.pdf          # Sample newspaper PDF
├── .devcontainer/
│   └── devcontainer.json              # Codespaces environment config
├── .vscode/
│   └── settings.json                  # Disables Pylance to save memory
├── .gitignore                         # Excludes model files and large data
└── README.md                          # This file
```
---

## Background

This workshop was developed for the **LRLL Malawi Workshop** to demonstrate practical NLP tools for Chichewa, one of Malawi's most widely spoken languages. The three sessions progress from text processing to NLP modelling to speech, and together form a complete low-resource language pipeline that can be adapted for other African languages by changing the dataset configuration.

- **Session 1** was originally built for Databricks/Azure and adapted to standard Jupyter
- **Session 2** uses the [Masakhane](https://masakhane.io) community NER dataset — a landmark effort to create NLP resources for African languages
- **Session 3** uses [Mozilla Common Voice](https://commonvoice.mozilla.org) — a community-contributed speech dataset
