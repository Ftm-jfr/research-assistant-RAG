# Research Assistant: RAG over CNN Papers

A retrieval-augmented generation (RAG) assistant for studying landmark computer-vision papers. It indexes four papers (AlexNet, VGG, ResNet, EfficientNet), answers free-form questions with retrieved context, and uses LLM **function calling** to run five analysis tools: extract experimental results, compare them with charts, write a mini survey, find research gaps, and compare limitations. A Streamlit chat app provides the interface.

> Course project for **Data Mining**, University of Isfahan.

## Features

- **PDF to Markdown extraction** with `pymupdf4llm`, which keeps tables as Markdown.
- **Table-aware chunking:** tables are split out and indexed as their own chunks (with a repair step for squished table rows), and the remaining prose is chunked with overlap (500 characters, 100 overlap).
- **Vector store:** OpenAI-style embeddings (`text-embedding-3-small`) stored in a persistent **ChromaDB** collection. The index is built once and reused, and it is rebuilt automatically when the set of papers changes.
- **Per-paper retrieval:** the top-k chunks are retrieved from every paper so that cross-paper questions are balanced, with optional filtering to a single paper.
- **Agent with function calling** (`gpt-4o-mini`): the model decides when a question needs one of the tools; otherwise it falls back to plain RAG.
- **Streamlit UI** with a chat box, quick-action buttons, an indexing log, and inline charts.

### The five tools

| Tool | What it does |
|---|---|
| `extract_experimental_results` | Pulls reported results (top-1/top-5 accuracy, parameters, dataset) from the papers as structured data |
| `compare_results` | Builds a comparison table and charts across papers |
| `generate_survey` | Writes a short survey covering all papers, grounded in the extracted results |
| `find_research_gap` | Collects acknowledged limitations, unsolved problems, and future-work directions per paper |
| `compare_limitations` | Compares the limitations of the papers side by side |

## Architecture

```
PDFs ──► pymupdf4llm (Markdown) ──► table + prose chunks ──► embeddings ──► ChromaDB
                                                                              │
user question ──► agent (LLM + tool schema) ──► tool call? ──yes──► tools ◄───┤ retrieval
                                   └──no──► RAG answer from retrieved chunks ◄┘
```

## Papers

The papers are not stored in this repository. Download them and place the PDFs in the `articles/` folder with these file names:

| File name | Paper |
|---|---|
| `AlexNet.pdf` | Krizhevsky, Sutskever, Hinton. *ImageNet Classification with Deep Convolutional Neural Networks.* Communications of the ACM, 2017. [DOI](https://doi.org/10.1145/3065386) |
| `VGGNet.pdf` | Simonyan, Zisserman. *Very Deep Convolutional Networks for Large-Scale Image Recognition.* ICLR 2015. [arXiv:1409.1556](https://arxiv.org/abs/1409.1556) |
| `ResNet.pdf` | He, Zhang, Ren, Sun. *Deep Residual Learning for Image Recognition.* CVPR 2016. [arXiv:1512.03385](https://arxiv.org/abs/1512.03385) |
| `EfficientNet.pdf` | Tan, Le. *EfficientNet: Rethinking Model Scaling for Convolutional Neural Networks.* ICML 2019. [arXiv:1905.11946](https://arxiv.org/abs/1905.11946) |

The project was developed with the Communications of the ACM reprint of AlexNet. Other versions of a paper work too, but extracted numbers and answers may differ slightly. Any PDF placed in `articles/` is indexed, and the app lists the loaded papers in the sidebar.

## Setup



Add your API key:

```bash
cp .env.example .env             # Windows: copy .env.example .env
# edit .env and set RESEARCH_ASSISTANT_API_KEY
```

By default the project calls the OpenAI API (`https://api.openai.com/`). It works with any **OpenAI-compatible** endpoint: uncomment and set `RESEARCH_ASSISTANT_BASE_URL` in `.env` to use a different provider. Never commit your `.env` file; it is listed in `.gitignore`.

Download the four papers into `articles/` (see the table above), then start the app:

```bash
streamlit run app.py
```

The first run extracts the PDFs and builds the vector index, which takes a short while and makes embedding API calls. Later runs reuse the stored index in `chroma_store/`.

## Example questions

- "Extract the experimental results for all four papers."
- "Compare all four papers on metrics, with charts."
- "Generate a mini survey covering all four papers."
- "Find the research gaps for all four papers."
- "Compare the limitations of all four papers side by side."
- Free-form questions such as "How do residual connections help train very deep networks?" are answered with retrieved passages from the papers.

## Repository contents

| File | Description |
|---|---|
| `app.py` | Streamlit chat interface |
| `backend.py` | Extraction, chunking, indexing, retrieval, tools, and the agent |
| `main.ipynb` | Step-by-step notebook version of the pipeline |
| `articles/` | Place the downloaded PDFs here |
| `.env.example` | Template for API configuration |
| `requirements.txt` | Python dependencies |



