# **CDM - RAG Assistant**

**Personal Clinical Data Management / Data Science RAG Assistant**

This project builds a **local-first Retrieval-Augmented Generation (RAG) assistant** for clinical data management (CDM) documents. It combines PDF knowledge, embeddings, and a local LLM to answer questions with references to source PDFs, pages, and headings.


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

4. Place your reference PDFs in `data/raw_pdfs/`.

5. Run the notebooks to:

   * Extract text from PDFs
   * Generate embeddings
   * Perform top-k retrieval
   * Query the local LLM

---

## **Future Plans**

* [ ] Improve headings extraction from PDFs for more accurate source references
* [ ] Integrate automatic summarization of long PDF sections
* [ ] Add workflow to fine-tune local LLM on CDM-specific language (optional)
* [ ] Include additional data science progression documents to bridge CDM → CDS
* [ ] Package as a lightweight CLI tool for faster queries
* [ ] Explore local deployment with Docker for reproducibility

---

## **Acknowledgments**

* [Sentence-Transformers](https://www.sbert.net/) — for embeddings
* [FAISS](https://github.com/facebookresearch/faiss) — for vector similarity search
* [GPT4All](https://gpt4all.io/) — local model for text generation