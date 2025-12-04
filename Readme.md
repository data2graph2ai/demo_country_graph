## What This Project Does

This project is a small, practical prototype that compares two approaches to analyzing data:

1. **LLM-only embeddings** – text features are encoded using a sentence encoder or language model.  
2. **LLM + GNN Link Prediction** – text features are combined with graph structure, and the GNN’s pre-final embeddings show how relationships change when structure is included.

The goal is to make it easy to **prototype and visualize** how results differ when you use only text versus when you use **text + topology**.  
This pattern works for any dataset where you have:

- **Nodes** with text features  
- **Edges** representing relationships  
- A need to compare *“LLM similarity”* vs. *“LLM + GNN similarity”*

The example in this repository uses **country borders** and **CIA World Factbook** text because they are public and easy to follow.  
But these datasets are **only for illustration** — you can replace them with your own graph, your own text features, and your own domain.

In short:  
This repo provides a reusable template for **testing LLM-only vs. LLM+GNN link prediction embeddings** and understanding how structure reshapes similarity maps in real data.


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
- 
### Compare Results

#### Country Pairs with highest LLM cosine similarities

| country1    | country2                    | border_type | sim_llm | vector_type |
|------------|-----------------------------|-------------|---------|-------------|
| North Korea| South Korea                 | both        |  0.987  | llm         |
| Saint Martin| Sint Maarten               | both        |  0.947  | llm         |
| Croatia    | Slovenia                    | both        |  0.811  | llm         |
| India      | Pakistan                    | both        |  0.784  | llm         |
| Czechia    | Slovakia                    | land        |  0.859  | llm         |
| Kenya      | Tanzania                    | land        |  0.820  | llm         |
| Estonia    | Latvia                      | land        |  0.816  | llm         |
| Croatia    | Serbia                      | land        |  0.814  | llm         |
| Guam       | Northern Mariana Islands    | sea         |  0.855  | llm         |
| Guernsey   | Jersey                      | sea         |  0.848  | llm         |
| Barbados   | Trinidad and Tobago         | sea         |  0.734  | llm         |
| China      | Taiwan                      | sea         |  0.717  | llm         |


#### Country Pairs with highest LLM + GNN cosine similarities

| country1               | country2         | border_type | sim_gnn | vector_type |
|------------------------|------------------|-------------|---------|-------------|
| Saint Martin           | Sint Maarten     | both        |  1.000  | gnn         |
| Croatia                | Slovenia         | both        |  0.998  | gnn         |
| Costa Rica             | Panama           | both        |  0.998  | gnn         |
| Algeria                | Tunisia          | both        |  0.997  | gnn         |
| Bosnia and Herzegovina | Croatia          | land        |  0.999  | gnn         |
| Bosnia and Herzegovina | Serbia           | land        |  0.999  | gnn         |
| Croatia                | Serbia           | land        |  0.999  | gnn         |
| Albania                | North Macedonia  | land        |  0.998  | gnn         |
| Colombia               | Nicaragua        | sea         |  0.999  | gnn         |
| Niue                   | Tonga            | sea         |  0.998  | gnn         |
| Samoa                  | Tonga            | sea         |  0.998  | gnn         |
| Cook Islands           | Kiribati         | sea         |  0.998  | gnn         |

#### Country Pairs with lowest LLM cosine similarities

| country1                               | country2      | border_type | sim_llm | vector_type |
|----------------------------------------|---------------|-------------|---------|-------------|
| Georgia                                | Turkey        | both        |  0.330  | llm         |
| Syria                                  | Turkey        | both        |  0.389  | llm         |
| Romania                                | Ukraine       | both        |  0.394  | llm         |
| Mozambique                             | South Africa  | both        |  0.425  | llm         |
| Mexico                                 | United States | land        |  0.191  | llm         |
| Afghanistan                            | China         | land        |  0.299  | llm         |
| Eswatini                               | South Africa  | land        |  0.314  | llm         |
| India                                  | Nepal         | land        |  0.315  | llm         |
| French Southern and Antarctic Lands    | Mozambique    | sea         |  0.178  | llm         |
| Saint Kitts and Nevis                  | Venezuela     | sea         |  0.184  | llm         |
| Saint Vincent and the Grenadines       | Venezuela     | sea         |  0.188  | llm         |
| Saint Lucia                            | Venezuela     | sea         |  0.207  | llm         |

#### Country Pairs with lowest LLM + GNN cosine similarities

| country1                             | country2                    | border_type | sim_gnn | vector_type |
|--------------------------------------|-----------------------------|-------------|---------|-------------|
| Israel                               | West Bank                   | both        |  0.665  | gnn         |
| Gaza, Gaza Strip                     | Israel                      | both        |  0.665  | gnn         |
| India                                | Pakistan                    | both        |  0.864  | gnn         |
| Egypt                                | Libya                       | both        |  0.880  | gnn         |
| Egypt                                | West Bank                   | land        |  0.621  | gnn         |
| Egypt                                | Gaza, Gaza Strip            | land        |  0.621  | gnn         |
| Jordan                               | West Bank                   | land        |  0.655  | gnn         |
| Gaza, Gaza Strip                     | Jordan                      | land        |  0.655  | gnn         |
| Canada                               | Saint Pierre and Miquelon   | sea         |  0.578  | gnn         |
| French Southern and Antarctic Lands  | Mozambique                  | sea         |  0.710  | gnn         |
| Netherlands                          | United Kingdom              | sea         |  0.721  | gnn         |
| Oman                                 | Pakistan                    | sea         |  0.732  | gnn         |

### Where LLM backs off but LLM+GNN insists “these are neighbors”

Some of the most interesting cases are **border pairs with low LLM similarity**.  
The text model is distracted by politics, history, or asymmetry, while the GNN pays attention to the *graph around them* and pulls them closer.

#### 1. Mexico – United States (land)

- **LLM view (low sim_llm):**  
  The Factbook pages emphasize very different stories: a large global power vs. an emerging economy; migration, drugs, security, trade disputes.  
  The shared border is only one theme among many, so their **text embeddings drift apart**.
- **LLM+GNN view (higher sim_gnn):**  
  In the border graph, they form **one of the most important land edges on the continent**, with Canada and Mexico as the US’s only land neighbors.  
  Training on land borders forces the GNN to treat this edge as structurally important, so **their graph-aware embeddings move closer together** than the LLM alone suggests.

#### 2. Georgia – Turkey and Syria – Turkey (both)

- **LLM view (low sim_llm):**  
  Text highlights very different identities: NATO member vs. post-Soviet state; EU candidate vs. conflict zones, sanctions, civil war.  
  The narrative focuses on **political blocks and conflicts**, which pushes their text vectors apart.
- **LLM+GNN view (higher sim_gnn):**  
  The graph doesn’t care about alliances; it sees a **continuous corridor of neighbors** linking the Caucasus, Anatolia, and the Levant.  
  Because Turkey is a central hub in that corridor, the GNN learns that Georgia–Turkey and Syria–Turkey are **structurally strong edges** and tightens their embeddings accordingly.

#### 3. India – Nepal (land)

- **LLM view (low sim_llm):**  
  India’s page is dominated by scale, internal politics, global role, and regional rivalries.  
  Nepal’s page focuses on Himalayan geography, tourism, and development challenges.  
  They share a huge open border, but the **text spends more time on how different they are**.
- **LLM+GNN view (higher sim_gnn):**  
  In the graph, India and Nepal are **deeply entangled neighbors** along the Himalayas, with many shared border segments and overlapping neighborhoods.  
  Link prediction “sees” that pattern and **pulls them closer than the text alone would**.

#### 4. Caribbean islands vs Venezuela (sea)

- **LLM view (low sim_llm):**  
  Small island states like **Saint Kitts and Nevis, Saint Vincent and the Grenadines, Saint Lucia** are described as independent micro-economies and tourist destinations.  
  Venezuela’s page is dominated by oil, internal politics, and regional alliances.  
  The maritime connection is just a line or two, so **their text similarity ends up near the bottom of our list**.
- **LLM+GNN view (higher sim_gnn):**  
  In the sea-border graph, several of these islands share **the same mainland neighbor: Venezuela**.  
  From the GNN’s perspective, they form a **small maritime neighborhood around the same hub**, so the model increases their similarity to Venezuela and to each other.

#### 5. French Southern and Antarctic Lands – Mozambique (sea)

- **LLM view (very low sim_llm):**  
  One is an almost uninhabited French territory in the southern Indian Ocean; the other is a large East African state.  
  Text makes them look almost unrelated.
- **LLM+GNN view (higher sim_gnn):**  
  On the sea-border graph, these islands sit **right off Mozambique’s coast**.  
  The GNN learns that, structurally, they are neighbors in the same maritime region and **corrects the LLM’s intuition by pulling them closer**.



## Why it’s reusable

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
