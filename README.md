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
