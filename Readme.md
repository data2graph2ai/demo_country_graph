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

