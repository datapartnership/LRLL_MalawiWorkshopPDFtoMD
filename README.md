# 📰 TiKAMBE Newspaper Article Extraction

Two Jupyter notebooks for extracting text from scanned PDF editions of **TiKAMBE**, a Malawi newspaper written in Chichewa.

| Notebook | What it does | Requires API key? |
|----------|-------------|-------------------|
| `pdf_text_extraction.ipynb` | Extracts all text from the PDF into a single `.txt` file | No |
| `article_extraction_jupyter.ipynb` | Uses Claude to split content into individual articles, each saved as its own `.md` file | Yes |

**Start with the text extraction notebook** to see the raw content, then run the Claude notebook to see what structured article extraction looks like on top of it.

---

## Notebooks

### 1. `pdf_text_extraction.ipynb` — Full text extraction

Extracts everything PyMuPDF can read from the PDF and saves it as one `.txt` file, with a `=== Page N ===` header before each page. No AI, no splitting, no API key needed.

**Example output (`tikambe_march2025.txt`):**
```
=== Page 1 ===

Mtsogoleri Akambira za Nthaka
Chisomo Banda
Boma la Malawi lakhulupirira kuti...

=== Page 2 ===

Aphunzitsi Akanena za Maphunziro
...
```

If any pages come back empty (common with older scans that have no embedded text layer), the notebook tells you which pages and explains how to switch to OCR.

---

### 2. `article_extraction_jupyter.ipynb` — Claude article extraction

Sends each page to Claude as an image and asks it to identify and extract every article. Each article is saved as its own numbered `.md` file, with headline, byline, topic domain, and body text.

**Example output:**
```
articles_output/
├── 01_Mtsogoleri Akambira za Nthaka.md
├── 02_Aphunzitsi Akanena za Maphunziro.md
├── 03_Timu ya Bullets Yapambana.md
└── ...
```

Each file looks like this:

```markdown
# Mtsogoleri Akambira za Nthaka

**By Chisomo Banda**
**Domain:** Politics
**Page:** 3

---

Article body text here...
```

Claude also skips non-article content automatically: ads, page numbers, staff credits, section banners, photo captions, and English-only pages.

---

## Quickstart — GitHub Codespaces (recommended for training)

The easiest way to run these notebooks is in a GitHub Codespace. Everything — Python, Jupyter, all packages, and the OCR engine — is pre-installed automatically. No local setup needed.

### Step 1 — Open a Codespace

1. Go to the repository on GitHub
2. Click the green **Code** button → **Codespaces** tab → **Create codespace on main**
3. Wait about 60–90 seconds for the environment to build
4. A browser-based VS Code opens, ready to go

### Step 2 — Add your Anthropic API key

Only needed for `article_extraction_jupyter.ipynb`. Store it as a **Codespaces secret** — never paste keys into notebooks.

1. Go to [github.com/settings/codespaces](https://github.com/settings/codespaces)
2. Under **Secrets**, click **New secret**
3. Name: `ANTHROPIC_API_KEY` — Value: your key
4. Under **Repository access**, add this repository
5. The key appears automatically as an environment variable in any Codespace you open

> If you added the secret after opening your Codespace, rebuild it: **Ctrl+Shift+P** → `Codespaces: Rebuild Container`.

### Step 3 — Add your PDF

Drag and drop your newspaper PDF into the file explorer panel on the left. Note the filename.

### Step 4 — Run a notebook

**Text extraction (no API key needed):**
1. Open `pdf_text_extraction.ipynb`
2. In Cell 1, set `PDF_PATH` to your PDF filename
3. Click **Run All**

**Claude article extraction:**
1. Open `article_extraction_jupyter.ipynb`
2. In Cell 1, set `PDF_PATH` to your PDF filename and leave `API_KEY = ""`
3. Click **Run All**

---

## Local setup (alternative)

```bash
# Install system dependency for OCR
sudo apt-get install -y tesseract-ocr   # Linux
# brew install tesseract                # macOS

# Install Python packages
pip install anthropic pymupdf pytesseract pillow jupyter

# Launch Jupyter
jupyter notebook
```

---

## Configuration reference

### `pdf_text_extraction.ipynb` (Cell 1)

| Variable | Default | Description |
|----------|---------|-------------|
| `PDF_PATH` | `"your_newspaper.pdf"` | Path to the input PDF |
| `OUTPUT_FILE` | `""` | Output `.txt` filename. Leave blank to auto-name from the PDF |

### `article_extraction_jupyter.ipynb` (Cell 1)

| Variable | Default | Description |
|----------|---------|-------------|
| `PDF_PATH` | `"your_newspaper.pdf"` | Path to the input PDF |
| `OUTPUT_FOLDER` | `"articles_output"` | Folder for output `.md` files (created automatically) |
| `API_KEY` | `""` | Anthropic API key. Leave blank to use the `ANTHROPIC_API_KEY` environment variable |
| `MODEL` | `"claude-sonnet-4-6"` | Claude model. Sonnet is a good balance of quality and speed |
| `MAX_TOKENS` | `16000` | Max response length per page. Increase if you see `max_tokens` warnings |

---

## What Claude extracts

For each article on a page, Claude returns:

| Field | Description |
|-------|-------------|
| `headline` | Article headline |
| `byline` | Author name, or `null` if not present |
| `domain` | Topic category — Politics, Health, Education, Agriculture, Sports, Crime, Community, Economy, Religion, Entertainment, Opinion, Obituary, or Other |
| `body` | Full article text with paragraph breaks |

---

## Error handling (Claude notebook)

- Each page is retried up to **3 times** before being skipped
- JSON parse errors trigger an automatic repair attempt before retrying
- If Claude hits the token limit (`stop_reason: max_tokens`), increase `MAX_TOKENS` in Cell 1
- Failed page numbers are listed in the summary at the end of the run

---

## Requirements

| Requirement | Detail |
|-------------|--------|
| Python | 3.9 or later |
| `pymupdf` | PDF reading and page rendering (both notebooks) |
| `anthropic` | Anthropic Python SDK (Claude notebook only) |
| `pytesseract` + `pillow` | OCR fallback for image-only PDFs (optional) |
| `tesseract-ocr` | System-level OCR engine required by `pytesseract` |
| Anthropic API key | Claude notebook only — get one at [console.anthropic.com](https://console.anthropic.com) |

All of the above are installed automatically when using Codespaces.

---

## Repository structure

```
.
├── pdf_text_extraction.ipynb          # Simple full-text extraction, no API key
├── article_extraction_jupyter.ipynb   # Claude-powered article extraction
├── .devcontainer/
│   └── devcontainer.json              # Codespaces environment config
└── README.md                          # This file
```

---

## Background

The Claude extraction pipeline was originally built for the Databricks platform using Azure Data Lake Storage. These notebooks are adapted for a single-PDF, standard Jupyter workflow — no cloud storage, no Spark, no Databricks endpoint required. The extraction prompt and retry logic are identical to the Databricks version.
