# **CDM - RAG Assistant**
**Ask questions about Clinical Data Management documents in plain English**

The assistant searches only your own documents and always tells you exactly which file and page the answer came from.

**Example:**
> **You ask:** *"What skills does a Clinical Data Manager need to transition to Data Science?"*
>
> **Assistant answers:** *"According to the SCDM Position Paper (page 7), Clinical Data Scientists need skills in... (Source: SCDM-Position-Paper.pdf, Page 7)"*

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
pip install sentence-transformers chromadb pypdf openpyxl python-docx requests tqdm jupyter ipywidgets
```

### Step 3 — Open the notebook

```bash
jupyter notebook 01_rag_basics.ipynb
```

Run each block **from top to bottom**, in order.

## 📁 Adding Documents

### Supported file types

| Format     | Extension          | How it's read                  |
|------------|--------------------|--------------------------------|
| PDF        | `.pdf`             | Page by page                   |
| Excel      | `.xlsx`, `.xls`    | Each sheet becomes one section |
| Word       | `.docx`            | Paragraph by paragraph         |
| JSON       | `.json`            | Converted to readable text     |
| XML        | `.xml`             | Element text extracted         |
| CSV        | `.csv`             | Rows joined as text            |
| Plain text | `.txt`             | Read as-is                     |

### How to add a new document

1. Copy the file into `data/raw/` (subfolders are fine):
   ```
   CDM-RAG-Assistant/
   └── data/
       └── raw/
           ├── your-new-document.pdf
           └── subfolder/
               └── another-doc.pdf
   ```

2. Re-run **Blocks 3, 4, and 5** in the notebook:
   - Block 3 — reloads all documents including the new one
   - Block 4 — re-chunks the text
   - Block 5 — rebuilds the search index

3. Ask your question in Block 7 or 8 — the new document is now included.

> 💡 You do **not** need to re-run Blocks 1, 2, or 6 unless you restart Jupyter.

## 💬 Reading the Output

After asking a question you'll see two sections:

### Answer
The LLM's response, drawn only from your documents.

### Sources (sorted most relevant first)
Each source is colour-coded by how closely it matched your question:

| Colour | Score | Meaning |
|--------|-------|---------|
| 🟢 **HIGH** | < 0.3 | Strong match — very likely relevant |
| 🟡 **MEDIUM** | 0.3 – 0.6 | Partial match — probably useful |
| 🔴 **LOW** | > 0.6 | Weak match — may not add much |

The score is a **cosine distance** (0 = identical meaning, 1 = completely unrelated). Sources are automatically deduplicated — if the same page appeared in multiple retrieved chunks, it shows only once.

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
| *"Folder not found"* | Check the `DOCS_FOLDER` path in Block 3 |
| Answers seem wrong or vague | Rephrase your question more specifically, or increase `TOP_K` to 7 in Block 7 |
| Block 5 is slow | Normal on the first run — the model downloads once and is cached after that |
| Want more precise citations | Decrease `CHUNK_SIZE` from 500 to 300 in Block 4 |

## 🗂️ Project Structure

```
CDM-RAG-Assistant/
├── notebooks/
│   └── 01_rag_basics.ipynb    ← main notebook
├── README.md                  ← this file
└── data/
    └── raw/                   ← put your documents here
        ├── subfolder/
        │   └── ...
        ├── *.pdf
        ├── *.xlsx
        └── ...
```

## ✅ Current Status

| Feature | Status |
|---------|--------|
| Load PDFs, XLSX, DOCX, JSON, XML, CSV, TXT | ✅ |
| Chunk text into overlapping segments | ✅ |
| Generate embeddings (`all-MiniLM-L6-v2`) | ✅ |
| Fast vector search (ChromaDB, cosine similarity) | ✅ |
| Results sorted by relevance, colour-coded | ✅ |
| Deduplicated sources in output | ✅ |
| Local LLM via Ollama | ✅ |
| Interactive Q&A loop (Block 8) | ✅ |

## 🔮 Future Ideas

- **Hybrid search** — combine the current semantic search with keyword search (BM25) to handle specific terms, IDs, and codes that semantic similarity alone can miss
- **Agentic retrieval** — let the LLM decide when to search again if its first results aren't sufficient, and rewrite its own search queries for better results
- **Improved heading extraction** from PDFs for more accurate source references
- **Automatic summarisation** of long document sections
- **CLI tool** for faster queries without opening Jupyter
- **Docker packaging** for reproducible deployment

> 💡 Note: PDFs & other files are not included in this repository. Place your own reference documents in `data/raw/`.
