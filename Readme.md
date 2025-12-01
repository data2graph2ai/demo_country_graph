## What this project does

This project is a small, concrete demo of a bigger idea:

> **Compare what we learn from pure LLM embeddings vs. what we learn from LLM + GNN link prediction on a graph.**

We use a **country borders graph** and **CIA World Factbook** text as a case study, but the pattern applies to any domain where you have:

- Text → encoded by an LLM / sentence encoder  
- Structure → encoded as pre-final vectors from GNN Link Prediction model graph   
- A need to see how **LLM-only** vs **LLM+GNN** change the “map” of similarities between items

### Case study: countries as a graph

In this demo:

- **Nodes** = countries  
- **Edges** = land and sea borders between countries  
- **Node text** = cleaned “Background” + “Geography” sections from the CIA Factbook for each country

From that, we build two kinds of embeddings for each country:

1. **LLM embeddings**  
   - We embed the CIA Factbook text with an LLM/sentence encoder.  
   - We get a vector for each country that reflects its background + geography text.

2. **LLM + GNN embeddings**  
   - We treat the borders as graph edges.  
   - We use the LLM embeddings as initial node features.  
   - We train a **GNN link prediction model** (e.g., GraphSAGE).  
   - After training, we take the GNN’s hidden vectors as **graph-aware embeddings** (LLM + structure).

### What we compare

For every pair of countries, we compute:

- **Cosine similarity in LLM space** → `sim_llm`
- **Cosine similarity in LLM+GNN space** → `sim_gnn`

Then we:

- Compare `sim_llm` vs `sim_gnn` across all pairs
- Break results down by **border type** (land / sea / both / none)
- Look at:
  - Pairs that are similar in text but not in graph space  
  - Pairs that the GNN pulls together because of shared neighbors / structural patterns  
  - Where “LLM only” and “LLM + GNN” strongly disagree

This gives a clear, data-driven way to answer:

- What does the **LLM alone** think is similar?  
- How does that picture change once we respect **graph structure** and train for a specific task (link prediction)?

### Why it’s reusable

While the demo focuses on **countries + borders + Factbook**, the method is generic:

- Replace **countries** with users, products, companies, documents, accounts, etc.
- Replace **borders** with follows, purchases, trades, citations, co-authorship, shared IP, etc.
- Replace **Factbook text** with descriptions, bios, reviews, profiles, tickets, metadata, or any text you already feed into an LLM.

The pipeline stays the same:

1. Build **LLM embeddings** for entities from text.
2. Build a **graph** connecting entities with edges that matter for your domain.
3. Train a **GNN link prediction model** on that graph.
4. Extract **GNN embeddings** and compare **LLM vs**


## Notebooks

- **01_country_graph_setup.ipynb**  
  Mounts Google Drive, explores the project folder, and builds the **country border graph** from land/sea shapefile data. Extracts land and sea neighbor relationships and saves a combined edge list  
  (`outputs/country_edges_land_sea.csv`).

- **02_text_from_factbook.ipynb**  
  Works with the CIA World Factbook JSON dump. Extracts each country’s **Background** and **Geography** text, cleans HTML, builds two text variants (background→geography and geography→background), maps country names to ISO3 codes (including fuzzy matching), and saves the result as  
  `outputs/country_bg_geo_factbook.csv`.

- **03_text_and_borders.ipynb**  
  Loads the Factbook text table and uses a sentence-embedding model to create **LLM text embeddings** for each country (`country_geo_bg_vector`). Computes pairwise cosine similarities between countries, joins these with the land/sea/both border information, and writes the embedding table to  
  `outputs/country_geo_bg_vectors.parquet` plus summary CSVs for analysis.

- **04_country_dgl_link_prediction.ipynb**  
  Runs in SageMaker / Python to build a **DGL graph** from the border edge list and the Factbook text embeddings as node features. Trains a GraphSAGE **link prediction** model on land/sea edges, evaluates AUC, and exports the learned **GNN node embeddings** to  
  `country_nodes_with_h.parquet` (country, iso3, nid, h_vector).

- **05_country_results.ipynb**  
  Loads both embedding tables (LLM and GNN), aligns them by ISO3, and computes pairwise cosine similarities in each space. Compares **LLM vs GNN similarity** for every country pair, breaks results down by border type (land / sea / both / none), and inspects top/bottom pairs and where the two spaces disagree most.

