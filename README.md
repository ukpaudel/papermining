**Concept Flow**
┌──────────────────────────────────────────┐
│        1. PAPER RETRIEVAL                │
└──────────────────────────────────────────┘
        │
        ├───► From ArXiv (latest preprints)
        │       - Categories: cs.SD, cs.CL, eess.AS
        │       - API: http://export.arxiv.org/api/query
        │       - Limit: up to 1000 papers
        │
        └───► From OpenAlex (highly cited references)
                - Year range: 2014–2024
                - Min citations: 50
                - Max results per query: 100
                - Queries:
                    • "audio machine learning"
                    • "transformer model"
                    • "attention mechanism"
                    • "large language models"
                    • "self-supervised learning"
                    • "TTS"
                    • "STT"
                    • "STS"
                    • "Agentic AI"
                - Force include:
                    • "Attention Is All You Need"
                    • "BERT: Pre-training..."
                    • "Language Models are Few-Shot Learners"

                          ▼

┌──────────────────────────────────────────┐
│      2. FILE DOWNLOAD & METADATA         │
└──────────────────────────────────────────┘
        │
        ├── Download PDFs via arXiv links
        ├── Save to:
        │       → `data/raw/` (PDFs)
        │       → `data/high_impact/` (initial JSON for OpenAlex)
        └── All metadata merged later into `data/results/`

                          ▼

┌──────────────────────────────────────────┐
│      3. PARSING & SECTION EXTRACTION     │
└──────────────────────────────────────────┘
        │
        ├── Extract LaTeX source from `.tar.gz` (if available)
        ├── Parse:
        │       • \begin{abstract}
        │       • \section{summary}
        │       • \section{conclusion}
        ├── Extract figure captions via `pdffigures2`
        └── Output to: `data/processed/{hash_id}.json`

                          ▼

┌──────────────────────────────────────────┐
│          4. LLM-BASED GRADING            │
└──────────────────────────────────────────┘
        │
        ├── LLM: GPT-4 Turbo
        ├── Prompt Engineering:
        │   You are an experienced AI researcher...
        │   - Read abstract, summary, conclusion, figure captions
        │   - Output structured JSON:
        │       • technical_summary
        │       • summary
        │       • novelty_score ∈ [0, 1]
        │       • verdict (e.g. "Major leap", "Incremental")
        │       • prior_art [list]
        │       • criticisms [list]
        └── Response is sanitized & saved into: `llm_summary`

                          ▼

┌──────────────────────────────────────────┐
│      5. TEXT EMBEDDING & REDUCTION       │
└──────────────────────────────────────────┘
        │
        ├── Model: `text-embedding-3-large` (OpenAI)
        │       • Applied only to `technical_summary`
        │       • Embedding dim = 3072
        ├── PCA to 50D 
        └── t-SNE → 2D projection
                → Saved as `llm_summary['umap_2d']`

                          ▼

┌──────────────────────────────────────────┐
│   6. INTERACTIVE VISUALIZATION (Plotly)  │
└──────────────────────────────────────────┘
        │
        ├── Scatter plot:
        │       • x/y from UMAP
        │       • color = novelty_score
        │       • shape = source (high_impact = diamond)
        ├── Hover:
        │       • title, summary, verdict, prior_art, criticisms
        │       • abstract (clipped and wrapped)
        ├── Click = open arXiv page in new tab
        └── Output: `output/semantic_map.html`

                          ▼

┌──────────────────────────────────────────┐
│      7. SHARE ONLINE (GitHub Pages)      │
└──────────────────────────────────────────┘
        │
        ├── Commit HTML file to GitHub repo
        ├── Enable Pages → Deploy from root or `/docs`
        └── Public URL ready to share














**Coding flow**


                        +------------------------+
                        |   arxiv_scraper.py     |  ← fetch recent papers from arXiv
                        |------------------------|
                        | Inputs: config (topics, max_results)
                        | Outputs: metadata + PDFs
                        | Stores: data/results/*.json, data/raw/*.pdf
                        +------------------------+
                                 ↓
                        +------------------------+
                        | high_impact_scraper.py |  ← pulls top-cited papers via OpenAlex
                        |------------------------|
                        | Inputs: query, citation filter
                        | Outputs: metadata + PDFs
                        | Stores: data/high_impact/*.json + data/results/*.json
                        +------------------------+
                                 ↓
                        +------------------------+
                        |   run_pipeline.py      |
                        |------------------------|
                        | Merges high_impact + recent papers
                        | Calls structure_parser, figure_writer, summarizer, embedder, reducer
                        +------------------------+
                                 ↓
    +-------------------+      +---------------------+
    | structure_parser  |      |    figure_writer    |
    |-------------------|      |---------------------|
    | Inputs: .tar.gz            | Inputs: PDF
    | Outputs: abstract,         | Outputs: figure captions
    |          conclusion        |---------------------
    | Stores: data/processed/*.json
    +-------------------+      +---------------------+
             ↓                             ↓
                      +-----------------------------+
                      |       merge_results.py      |
                      |-----------------------------|
                      | Combines structure + figures + metadata
                      | Stores: data/results/{hash_id}.json
                      +-----------------------------+
                                 ↓
                        +------------------------+
                        |     summarizer.py      |
                        |------------------------|
                        | Inputs: parsed paper JSON
                        | Outputs: llm_summary
                        | Fields: technical_summary, verdict, prior_art, etc.
                        +------------------------+
                                 ↓
                        +------------------------+
                        |     embedder.py        |
                        |------------------------|
                        | Inputs: technical_summary
                        | Outputs: concept_vector (embedding)
                        +------------------------+
                                 ↓
                        +------------------------+
                        |      reducer.py        |
                        |------------------------|
                        | Inputs: concept_vectors
                        | Outputs: 2D points (umap_2d)
                        | Updates: data/results/*.json
                        +------------------------+
                                 ↓
                        +------------------------+
                        |      plotter.py        |
                        |------------------------|
                        | Inputs: all results/*.json
                        | Outputs: output/semantic_map.html
                        | Features: hover details, click-to-open paper
                        +------------------------+
