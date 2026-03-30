# CDM RAG Assistant
**Ask questions about Clinical Data Management documents in plain English.**

The assistant searches only your own documents and always tells you exactly which file and page the answer came from. Documents are classified by the folder they live in — regulatory requirements and opinion papers are kept separate and treated differently in every answer.

**Example:**
> **You ask:** *"What skills does a Clinical Data Manager need to transition to Data Science?"*
>
> **Assistant answers:** *"ICH E6 requires that... (Source: ICH-E6.pdf, Page 12). The SCDM Position Paper recommends... (Source: SCDM-Position-Paper.pdf, Page 4)"*

## ⚙️ One-Time Setup

### Step 1 — Install Ollama (the local AI engine)

1. Go to **[https://ollama.com](https://ollama.com)** and download the app
2. Install it (drag to Applications on Mac, run the installer on Windows/Linux)
3. Open **Terminal** and run:
   ```bash
   ollama pull mistral
   ```
   This downloads the Mistral language model (~4 GB). You only do this once.

Ollama runs automatically in the background — no further action needed.

### Step 2 — Set up your Python environment

In Terminal, navigate to this project folder and run:

```bash
python3 -m venv .venv
source .venv/bin/activate          # on Windows: .venv\Scripts\activate
pip install sentence-transformers chromadb pypdf openpyxl python-docx python-pptx requests tqdm jupyter ipywidgets
```

### Step 3 — Create the document folders

```bash
mkdir -p data/raw/regulatory
mkdir -p data/raw/opinion
```

### Step 4 — Open the notebook

```bash
jupyter notebook 01_rag_basics.ipynb
```

Run each block **from top to bottom**, in order.

---

## 📁 Adding Documents

### Where to put files — authority by folder

The folder a file lives in determines how the assistant treats it:

| Folder | Authority | Use for | LLM language |
|--------|-----------|---------|--------------|
| `data/raw/regulatory/` | ⚖️ Regulatory | ICH, FDA, EMA, binding standards | *must, is required, mandates, shall* |
| `data/raw/opinion/` | 💬 Opinion | SCDM, JSCDM, white papers, position papers | *recommends, suggests, proposes, advises* |

> ⚠️ Files placed anywhere else (directly in `data/raw/` for example) will cause an error when Block 3 runs — this is intentional so nothing is ever classified by accident.

### Supported file types

| Format      | Extension       | How it's read                       |
|-------------|-----------------|-------------------------------------|
| PDF         | `.pdf`          | Page by page                        |
| PowerPoint  | `.pptx`         | Slide by slide (incl. speaker notes)|
| Excel       | `.xlsx`, `.xls` | Each sheet becomes one section      |
| Word        | `.docx`         | Paragraph by paragraph              |
| JSON        | `.json`         | Converted to readable text          |
| XML         | `.xml`          | Element text extracted              |
| CSV         | `.csv`          | Rows joined as text                 |
| Plain text  | `.txt`          | Read as-is                          |

> ⚠️ Old-format `.ppt` files are not supported. Open them in PowerPoint, choose File → Save As → PowerPoint Presentation (.pptx), then add the converted file.

### How to add a new document

1. Drop the file into the correct subfolder:
   ```
   data/raw/
   ├── regulatory/
   │   ├── ICH-E6-GCP.pdf
   │   └── FDA-21-CFR-Part-11.pdf
   └── opinion/
       ├── SCDM-Position-Paper-V9.pdf
       └── JSCDM-2024-white-paper.pptx
   ```

2. Re-run **Blocks 3, 4, and 5** in the notebook:
   - Block 3 — reloads all documents including the new one
   - Block 4 — re-chunks the text
   - Block 5 — rebuilds the search index

3. Ask your question in Block 7 or 8 — the new document is now included.

> 💡 You do **not** need to re-run Blocks 1, 2, or 6 unless you restart Jupyter.

### Managing document versions

| Situation | What to do |
|-----------|------------|
| A document has been updated and the old version is no longer valid | Delete the old file, drop in the new one with the same filename, re-run Blocks 3–5 |
| You want to keep both versions and compare them | Rename files to include the version — e.g. `SCDM-Position-Paper-v8.pdf` and `SCDM-Position-Paper-v9.pdf` — keep both in the same subfolder, re-run Blocks 3–5. You can then ask *"What changed between v8 and v9 regarding the CDM role?"* |

## 💬 Reading the Output

After asking a question you will see two sections.

### Answer
The LLM's response, drawn only from your documents. Language is automatically adjusted based on source authority — regulatory sources produce *must / shall / is required*, opinion sources produce *recommends / suggests / proposes*. If a regulatory and an opinion source conflict on the same point, the assistant will flag this explicitly.

### Sources
Sources are listed with two badges each:

**Authority badge** — what kind of document this is:

| Badge | Meaning |
|-------|---------|
| ⚖️ REGULATORY | Binding requirement — ICH, FDA, EMA |
| 💬 OPINION | Professional recommendation — SCDM, JSCDM, white papers |

**Relevance badge** — how closely this source matched your question (cosine distance, lower = better):

| Badge | Score | Meaning |
|-------|-------|---------|
| 🟢 HIGH | < 0.3 | Strong match — very likely relevant |
| 🟡 MEDIUM | 0.3 – 0.6 | Partial match — probably useful |
| 🔴 LOW | > 0.6 | Weak match — may not add much |

Regulatory sources are always listed first, then opinion sources, with both groups sorted by relevance within themselves. Sources are automatically deduplicated — if the same page appeared in multiple retrieved chunks, it shows only once.

## 🔧 Changing the AI Model

The default model is `mistral`. To switch, edit **Block 6**:

```python
OLLAMA_MODEL = "mistral"    # balanced — recommended starting point
OLLAMA_MODEL = "llama3.2"   # strong alternative
OLLAMA_MODEL = "phi3"       # fastest — good for quick tests
```

To download a different model:
```bash
ollama pull llama3.2
```

## ❓ Troubleshooting

| Symptom | Fix |
|---------|-----|
| *"Cannot connect to Ollama"* | Open the Ollama app, or run `ollama serve` in Terminal |
| *"Cannot classify: filename.pdf"* | Move the file into `data/raw/regulatory/` or `data/raw/opinion/` |
| *"Skipped (old format): file.ppt"* | Convert to `.pptx` in PowerPoint: File → Save As → PowerPoint Presentation |
| *"Folder not found"* | Check the `DOCS_FOLDER` path in Block 3 |
| Answers seem wrong or vague | Rephrase more specifically, or increase `TOP_K` to 7 in Block 7 |
| Block 5 is slow on first run | Normal — the embedding model downloads once and is cached after that |
| Want more precise citations | Decrease `CHUNK_SIZE` from 500 to 300 in Block 4 |

## 🗂️ Project Structure

```
CDM-RAG-Assistant/
├── notebooks/
│   └── 01_rag_basics.ipynb    ← main notebook
├── README.md                  ← this file
└── data/
    └── raw/
        ├── regulatory/        ← binding documents (ICH, FDA, EMA)
        │   ├── ICH-E6-GCP.pdf
        │   └── ...
        └── opinion/           ← position papers, white papers
            ├── SCDM-Position-Paper-V9.pdf
            └── ...
```

## ✅ Current Status

| Feature | Status |
|---------|--------|
| Load PDF, PPTX, XLSX, DOCX, JSON, XML, CSV, TXT | ✅ |
| Folder-based authority classification (regulatory / opinion) | ✅ |
| Error on unclassified files — nothing slips through unchecked | ✅ |
| Chunk text into overlapping segments | ✅ |
| Generate embeddings (`all-MiniLM-L6-v2`) | ✅ |
| Fast vector search (ChromaDB, cosine similarity) | ✅ |
| Regulatory sources ranked first in results | ✅ |
| Authority-aware LLM language (must vs recommends) | ✅ |
| Colour-coded relevance badges per source | ✅ |
| Deduplicated, wrapped output | ✅ |
| Local LLM via Ollama | ✅ |
| Interactive Q&A loop (Block 8) | ✅ |

## 🔮 Future Ideas

- **Hybrid search** — combine semantic search with keyword search (BM25) to better handle specific terms, IDs, and codes
- **Agentic retrieval** — let the LLM decide when to search again if first results are insufficient, and rewrite its own queries for better results
- **Improved heading extraction** from PDFs for more accurate source references
- **Automatic summarisation** of long document sections
- **CLI tool** for faster queries without opening Jupyter
- **Docker packaging** for reproducible deployment

> 💡 Note: documents are not included in this repository. Place your own reference files in `data/raw/regulatory/` or `data/raw/opinion/`.
