# Country Border Graph Demo

This repo contains a small end-to-end demo that builds a **country graph** from tabular + text data and feeds it into a **Graph Neural Network (GNN) / DGL** workflow.

The core idea:

- Countries are **nodes**  
- Borders (land / sea) are **edges**  
- Each country node gets a **vector embedding** derived from background + geography text (e.g., CIA World Factbook–style descriptions)

The main notebook walks through loading the data, turning it into a DGL graph, attaching node features, and preparing everything for further Graph AI experiments.

---

## Project structure

```text
.
├─ country_borders/
│  ├─ country_edges_land_sea.csv
│  ├─ country_geo_bg_vectors.csv    # or .parquet in the original Colab
│  └─ (notebook lives here or in parent dir)
├─ README.md
└─ <notebook>.ipynb
