# PDF Question Answering Assistant (RAG)

A notebook-based question answering assistant that retrieves relevant passages from a PDF and uses a Groq-hosted language model to generate answers. The project demonstrates a Retrieval-Augmented Generation (RAG) pipeline and includes a small Gradio interface for asking questions.

## What it does

The notebook downloads an Object-Oriented Programming (OOP) concepts PDF, loads and splits its pages into overlapping text chunks, embeds those chunks, and indexes them in a Chroma vector database. When a user asks a question, the retriever finds relevant chunks and the Groq LLM generates an answer using that retrieved context. The chain can also return the source documents.

## Architecture

```text
PDF URL
  │
  ▼
Download and save PDF (OOPSConcepts.pdf)
  │
  ▼
PyPDFLoader ──► page documents
  │
  ▼
CharacterTextSplitter (2,000 characters, 400 overlap)
  │
  ▼
Hugging Face sentence embeddings
  │
  ▼
Chroma vector database (vector_db/)
  │
  ▼
Retriever ──► relevant document chunks
                 │
User question ───┤
                 ▼
       LangChain RetrievalQA chain ◄── Groq ChatGroq LLM
                 │
                 ▼
          Answer in Gradio UI
```

## RAG workflow

1. **Ingest:** The notebook fetches `OOPConcepts.pdf` from the URL in the notebook and saves it in the current working directory.
2. **Load:** `PyPDFLoader` extracts the PDF into LangChain document objects, generally one per page.
3. **Split:** `CharacterTextSplitter` breaks the content into 2,000-character chunks with 400 characters of overlap. Overlap helps preserve context across chunk boundaries.
4. **Embed and index:** `HuggingFaceEmbeddings` converts each chunk into a vector. `Chroma` stores the vectors and associated document text in `vector_db/`.
5. **Retrieve:** The question is embedded and the vector store returns relevant chunks through a retriever.
6. **Generate:** `RetrievalQA` uses the `ChatGroq` model and the retrieved chunks (`chain_type="stuff"`) to form a context-grounded answer. Source documents are enabled in the chain response.
7. **Present:** A Gradio Blocks interface accepts a question and displays the generated answer.

## Technologies

- **Python and Jupyter / Google Colab** — notebook runtime. The first notebook cell imports Google Drive support, so the notebook as written expects Colab.
- **LangChain** — document handling and retrieval QA chain orchestration.
- **PyPDFLoader (`langchain-community`)** — PDF text extraction.
- **CharacterTextSplitter (`langchain-text-splitters`)** — chunking long document text.
- **Hugging Face Sentence Transformers** — local text embeddings (the notebook relies on the package's default model).
- **Chroma (`langchain-chroma`)** — persistent vector store.
- **Groq (`langchain-groq`)** — hosted chat model, configured in the notebook as `qwen/qwen3.8-27b` with temperature 0.
- **Gradio** — simple browser-based question and answer interface.

## Repository structure

```text
RAG_Documemt_QA_Project/
├── README.md
├── Retrieval_Augmented_Generation_(RAG)_DocumentQA__Assistant.ipynb
├── requirements.txt.txt   # Existing dependency list; rename to requirements.txt for pip's default convention
├── OOPSConcepts.pdf        # Downloaded by the notebook at runtime; not currently included in the project folder
└── vector_db/              # Chroma persistence directory, created when the notebook runs
```

`OOPSConcepts.pdf` and `vector_db/` are runtime artifacts. They are not present in the provided project folder before execution. Avoid committing a Groq API key or other credentials to GitHub.

## Setup and run

The notebook is currently written for **Google Colab** (it imports `google.colab.drive`). Open the notebook in Colab, then run its cells from top to bottom. The dependency-install cell currently has its commands commented out; uncomment and run the required installs, or install dependencies in the runtime first.

The repository's dependency file is currently named `requirements.txt.txt`. If using a local Python environment, rename it to `requirements.txt`, ensure the notebook's direct dependencies are included (including `pypdf`, `requests`, `gradio`, and the LangChain package that provides `langchain_classic`), then install with:

```bash
python -m venv .venv
# Activate the virtual environment for your operating system
python -m pip install -r requirements.txt
```

Next, open the notebook in Jupyter or Colab and run all cells in order. The notebook prompts for a Groq API key using `getpass`, which hides the key as you enter it. You can create/manage a key through Groq's developer console. Never paste a real key into a notebook cell or commit it to version control.

When running locally, remove or adapt the `from google.colab import drive` / `drive.mount(...)` cell, since it is Colab-specific.

## Security

The Groq key is requested interactively with `getpass` and placed in the runtime environment as `GROQ_API_KEY`. Keep it private, avoid saving it in notebook outputs, and rotate it if it is accidentally published.
