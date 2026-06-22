# 📰 TiKAMBE Newspaper Article Extraction

Extracts individual news articles from scanned PDF editions of **TiKAMBE**, a Malawi newspaper written in Chichewa. Each page is sent to Claude as an image, and every article is saved as its own Markdown file — ready for search indexing, RAG pipelines, or fine-tuning datasets.

---

## What this does

1. Opens a scanned newspaper PDF
2. Renders each page as a JPEG image
3. Sends the image to Claude with a detailed extraction prompt
4. Parses Claude's response (a JSON array of articles)
5. Saves each article as a numbered `.md` file

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

---

## Quickstart — GitHub Codespaces (recommended for training)

The easiest way to run this notebook is in a GitHub Codespace. Everything is pre-installed — no local setup needed.

### Step 1 — Open a Codespace

1. Go to the repository on GitHub
2. Click the green **Code** button → **Codespaces** tab → **Create codespace on main**
3. Wait about 60 seconds for the environment to build
4. A browser-based VS Code opens with Python, Jupyter, and all dependencies ready

### Step 2 — Add your API key

Your Anthropic API key should be stored as a **Codespaces secret**, not pasted into the notebook.

1. Go to [github.com/settings/codespaces](https://github.com/settings/codespaces)
2. Under **Secrets**, click **New secret**
3. Name: `ANTHROPIC_API_KEY` — Value: your key
4. Under **Repository access**, add this repository
5. The key will be available automatically the next time you open a Codespace

> If you've already opened a Codespace before adding the secret, rebuild it: **Ctrl+Shift+P** → `Codespaces: Rebuild Container`.

### Step 3 — Add your PDF

Drag and drop your newspaper PDF into the file explorer panel on the left. Note the filename.

### Step 4 — Run the notebook

1. Open `article_extraction_jupyter.ipynb`
2. In **Cell 1**, set `PDF_PATH` to your PDF filename (e.g. `"tikambe_march2025.pdf"`)
3. Leave `API_KEY = ""` — it will pick up your Codespaces secret automatically
4. Click **Run All** (or run cells one at a time from top to bottom)

---

## Local setup (alternative)

If you prefer to run locally:

```bash
# Install dependencies
pip install anthropic pymupdf jupyter

# Launch Jupyter
jupyter notebook
```

Then open `article_extraction_jupyter.ipynb`, set `PDF_PATH` and `API_KEY` in Cell 1, and run all cells.

---

## Notebook structure

The notebook has four code cells, each preceded by a markdown cell explaining what it does.

| Cell | Purpose |
|------|---------|
| **Cell 1 — Configuration** | Set PDF path, output folder, API key, model, token limit |
| **Cell 2 — Prompt** | The instructions sent to Claude for each page |
| **Cell 3 — Claude client** | API call, retry logic, JSON parsing and repair |
| **Cell 4 — Main pipeline** | Page rendering loop, article saving |

---

## Configuration reference

All settings are in **Cell 1**:

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
| `domain` | Topic category — one of: Politics, Health, Education, Agriculture, Sports, Crime, Community, Economy, Religion, Entertainment, Opinion, Obituary, Other |
| `body` | Full article text with paragraph breaks |

**Claude skips:** ads, page numbers, staff credits, section banners, photo captions, pull quotes, continuation lines, contact info, and blank or English-only pages.

---

## Error handling

The pipeline is designed to keep running even when individual pages fail:

- Each page is retried up to **3 times** before being skipped
- JSON parse errors (often caused by quotes inside article text) trigger an automatic repair attempt before retrying
- If Claude hits the token limit (`stop_reason: max_tokens`), the page is skipped immediately — retrying won't help, but increasing `MAX_TOKENS` in Cell 1 will
- Failed page numbers are listed in the summary at the end of the run

---

## Requirements

| Requirement | Detail |
|-------------|--------|
| Python | 3.9 or later |
| `anthropic` | Anthropic Python SDK |
| `pymupdf` | PDF rendering (imported as `fitz`) |
| Anthropic API key | Get one at [console.anthropic.com](https://console.anthropic.com) |

If using Codespaces, these are all installed automatically via `.devcontainer/devcontainer.json`.

---

## Repository structure

```
.
├── article_extraction_jupyter.ipynb   # Main notebook
├── .devcontainer/
│   └── devcontainer.json              # Codespaces environment config
└── README.md                          # This file
```

---

## Background

This pipeline was originally built for the Databricks platform using Azure Data Lake Storage. This version is adapted for a single-PDF, standard Jupyter workflow — no cloud storage, no Spark, no Databricks endpoint required. The extraction prompt and retry logic are identical to the Databricks version.
