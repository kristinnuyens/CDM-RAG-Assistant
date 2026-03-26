# **CDM - RAG Assistant**

**Personal Clinical Data Management / Data Science RAG Assistant**

This notebook lets you load CDM/CDS documents and ask questions about them in plain English.
The assistant will answer using *only* information from the loaded documents, and will always tell which file and page the answer came from.

Example for MVP:
> **You ask:** "What skills does a Clinical Data Manager need to transition to Data Science?"
>
> **Assistant answers:** "According to the SCDM Position Paper (page 7), Clinical Data Scientists need skills in... (Source: SCDM-Position-Paper.pdf, Page: 7)"

## ⚙️ One-time Setup

### Step 1 — Install Ollama (the local AI engine)

1. Go to **[https://ollama.com](https://ollama.com)** and download the Mac app
2. Install it (drag to Applications)
3. Open **Terminal** and run:
   ```
   ollama pull mistral
   ```
   This downloads the Mistral language model (~4 GB). You only do this once.

4. Ollama will now run automatically in the background whenever your Mac is on.

### Step 2 — Set up Python environment

In Terminal, navigate to this project folder and run:

```bash
python -m venv .venv
source .venv/bin/activate
pip install sentence-transformers faiss-cpu pypdf openpyxl python-docx requests tqdm jupyter
```

### Step 3 — Open the notebook

```bash
jupyter notebook 01_rag_basics.ipynb
```

## 📁 How to add new documents

### Supported file types
| Format | Extension | Notes |
|--------|-----------|-------|
| PDF | `.pdf` | Extracted page by page |
| Excel | `.xlsx`, `.xls` | Each sheet becomes a section |
| Word | `.docx` | Extracted paragraph by paragraph |
| JSON | `.json` | Converted to readable text |
| XML | `.xml` | Element text extracted |
| CSV | `.csv` | Rows joined as text |
| Plain text | `.txt` | Read as-is |

### Adding files — 3 easy steps

1. **Copy your file(s)** into the `data/raw/` folder (or any subfolder inside it)
   ```
   CDM-RAG-Assistant/
   └── data/
       └── raw/
           ├── your-new-document.pdf   ← put files here
           ├── competency-table.xlsx
           └── subfolder/
               └── another-doc.pdf     ← subfolders work too!
   ```

2. **Re-run Blocks 3, 4, and 5** in the notebook (in order):
   - Block 3 — reloads all documents including your new one
   - Block 4 — re-chunks the text
   - Block 5 — rebuilds the search index

+++++
❌ 🚨 **Currently Block 5 crashed the kernel - so we are looking into fixing this...** ❌ 🚨 
+++++

3. **Ask your question** in Block 7 or 8 — the new document is now included!

> 💡 You do **not** need to re-run Blocks 1, 2, or 6 unless you restart Jupyter.

---

## 🔧 Changing the AI model

The default model is `mistral`. You can change it in **Block 6** of the notebook:

```python
OLLAMA_MODEL = "mistral"     # default — good balance
OLLAMA_MODEL = "llama3.2"   # alternative — also very good
OLLAMA_MODEL = "phi3"        # fastest — good for quick tests
```

To download a different model, run in Terminal:
```bash
ollama pull llama3.2
```

---

## ❓ Troubleshooting

**"Cannot connect to Ollama"**
→ Make sure Ollama is running. Open the Ollama app from your Applications folder, or run `ollama serve` in Terminal.

**"Folder not found"**
→ Check the `DOCS_FOLDER` path in Block 3. It should point to where your documents are.

**Answers seem wrong or vague**
→ Try rephrasing your question more specifically.
→ Try increasing `TOP_K` from 5 to 7 in Block 7.

**Notebook runs slowly**
→ Normal for the first run (models download from the internet). Subsequent runs are faster.
→ The embedding step (Block 5) can take 1–2 minutes for large document sets — this is normal.

---

## 🗂️ Project structure

```
CDM-RAG-Assistant/
├── 01_rag_basics.ipynb    ← main notebook
├── README.md              ← this file
└── data/
    └── raw/               ← put your documents here
        ├── *.pdf
        ├── *.xlsx
        └── ...
```




## **Current Status**

* ✅ Load PDFs from `data/raw/` and its subfolders
* ✅ Chunk PDFs into manageable text segments
* ✅ Generate embeddings using `all-MiniLM-L6-v2` (SentenceTransformers)
* ✅ Store embeddings in FAISS for fast retrieval
* ✅ Top-k retrieval of relevant chunks with source page/heading context
* ✅ Local LLM (GPT4All-J) integration for answering questions using retrieved chunks

## **Folder Structure**

```
CDM-RAG-Assistant/
├── data/
│   └── raw/               # Place your PDF files here  (ignored by Git)
│       └── subfolder1/    # Subfolders for organizing sources
├── notebooks/             # Jupyter notebooks for RAG workflow
├── models/                # Optional local models (e.g., GPT4All-J, MiniLM)
├── .venv/                 # Python virtual environment
├── .gitignore             # Ignore PDFs, venv, cache files, etc.
├── requirements.txt       # Python dependencies
└── README.md
```

**Note:** PDFs are currently **not included in the repository** due to size and possible privacy considerations. Users can place their own reference PDFs in `data/raw/`.

## **Getting Started**

1. Clone the repository:

```bash
git clone <repo_url>
cd CDM-RAG-Assistant
```

2. Create and activate a virtual environment:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

3. Install dependencies:

```bash
pip install -r requirements.txt
```

4. Place your reference PDFs in `data/raw/`.

5. Run the notebooks to:

   * Extract text from PDFs
   * Generate embeddings
   * Perform top-k retrieval
   * Query the local LLM

## **Future Plans**

* [ ] Improve headings extraction from PDFs for more accurate source references
* [ ] Integrate automatic summarization of long PDF sections
* [ ] Add workflow to fine-tune local LLM on CDM-specific language (optional)
* [ ] Include additional data science progression documents to bridge CDM → CDS
* [ ] Package as a lightweight CLI tool for faster queries
* [ ] Explore local deployment with Docker for reproducibility

