# Management Accounting Research — Systematic Literature Review

A reproducible pipeline for systematic literature review and topic modelling of management accounting research. The project analyses 2,625 articles from 10 accounting journals (Scopus export, 2021–2026), identifies latent research topics using BERTopic, and prepares structured input for AI-assisted topic labelling.

---

## Project Structure

```
.
├── 01_Data_pre-processing.ipynb      # Step 1–9: Load, clean, filter, and export corpora
├── 02_Topic_modelling.ipynb          # Step 10–17: Embeddings, BERTopic, visualisations
├── 03_AI_Labelling.ipynb             # Step 18+: AI-assisted topic labelling & analysis
│
├── Scopus_Screening_results.xlsx     # Raw Scopus export (input)
│
├── papers_preprocessed.csv           # All 2,625 articles after cleaning
├── papers_MA_journals.csv            # 314 articles from MA-specialist journals
├── papers_with_topics.csv            # Topic assignments (BERTopic output, integer IDs)
├── papers_with_labels.csv            # Topic assignments with AI-generated labels & methodology
├── sensitivity_check.csv             # Grid-search results across UMAP/HDBSCAN params
├── topic_journal_crosstab.csv        # 11×3 topic × journal contingency table
├── topic_methodology_crosstab.csv    # Topic × methodology breakdown
│
├── embeddings.npy                    # Cached sentence embeddings (384-d, MiniLM-L6-v2)
├── embeddings_mode.txt               # Records which corpus mode the embeddings were built on
├── topic_labeling_input.txt          # Structured per-topic summaries for LLM labelling
│
├── fig_topic_barchart.png/.html      # Top topics by paper count
├── fig_topic_keywords.png            # Per-topic c-TF-IDF keyword heatmap
├── fig_topic_map.png/.html           # 2-D UMAP document map coloured by topic
├── fig_topics_over_time.png/.html    # Topic trend lines (2021–2026)
├── fig_topic_similarity.png          # Topic similarity / distance matrix
├── fig_journal_topic.png             # Stacked bar: topic distribution by journal
├── fig_journal_heatmap.png           # Heatmap: topic × journal proportions
└── fig_methodology_topic.png         # Methodology breakdown by topic
```

---

## Notebooks

### `01_Data_pre-processing.ipynb`

Loads the raw Scopus export and produces clean, analysis-ready CSV files.

| Step | Description |
|------|-------------|
| 1 | Load & inspect raw data (shape, types, missing values) |
| 2 | Filter to `Document Type == Article` (2,675 → 2,633 rows) |
| 3 | Remove papers with missing or placeholder abstracts (→ 2,625 rows) |
| 4 | Text cleaning: produce `abstract_clean` (BoW/TF-IDF) and `text_for_modeling` (raw, for sentence transformers) |
| 5 | Journal × year distribution plots |
| 6 | Abstract length analysis; flag papers < 30 words |
| 7 | Management accounting keyword filter (`is_management_accounting` flag) |
| 8 | Export four corpus variants (see below) |
| 9 | Summary report of all filtering steps |

**Exported corpora**

| File | Rows | Description |
|------|------|-------------|
| `papers_preprocessed.csv` | 2,625 | All articles after cleaning |
| `papers_MA_filtered.csv` | 245 | Articles matching MA keyword list |
| `papers_MA_journals.csv` | 314 | Articles from three MA-specialist journals |
| `papers_MA_strict.csv` | 126 | Intersection: MA journals AND MA keyword match |

**Key packages:** `pandas`, `matplotlib`, `seaborn`, `nltk`, `re`, `openpyxl`

---

### `02_Topic_modelling.ipynb`

Applies BERTopic to the `MA_journals` corpus (314 articles) to identify latent research topics.

| Step | Description |
|------|-------------|
| 10 | Load data and select corpus mode (`full` / `MA_flag` / `MA_journals` / `MA_strict`) |
| 11 | Encode documents with `all-MiniLM-L6-v2` sentence transformer; cache embeddings to `embeddings.npy` |
| 12a | Initial BERTopic model fit (UMAP + HDBSCAN + CountVectorizer) |
| 12b | Sensitivity analysis over 12 parameter combinations (n_neighbors × min_samples × cluster method) |
| 12c | Select final hyperparameters from grid results |
| 12d | Final model fit on full corpus |
| 13 | Topic review: noise inspection (40 papers, 12.7%); Topics 0 and 2 flagged as broad clusters |
| 14 | Export publication-quality visualisations |
| 16 | Generate `topic_labeling_input.txt` for LLM-based labelling |
| 17 | Final summary |

**Model configuration**

| Parameter | Value |
|-----------|-------|
| Embedding model | `all-MiniLM-L6-v2` (384-d) |
| Dimensionality reduction | UMAP |
| Clustering | HDBSCAN (min_cluster_size=8, min_samples=1) |
| Final topics | 11 (original BERTopic output — no manual splits applied) |
| Noise papers | 40 (12.7%) |

**Key packages:** `bertopic`, `sentence_transformers`, `umap`, `hdbscan`, `sklearn`, `plotly`, `numpy`, `pandas`

---

### `03_AI_Labelling.ipynb`

AI-assisted topic labelling and quantitative analysis of the 11 BERTopic clusters.

| Step | Description |
|------|-------------|
| 18 | Load `papers_with_topics.csv`; apply keyword-based methodology classifier (Experimental / Analytical / Qualitative / Survey-Field / Archival / Other-Mixed) |
| 19 | Structured per-topic summaries from `topic_labeling_input.txt` as LLM input |
| 20 | Assign human-readable `AI_LABELS` to all 11 topics (integer IDs 0–10); export `papers_with_labels.csv` |
| 21 | Journal × topic contingency table; χ²(20) = 66.1, Cramér's V = 0.284; export `topic_journal_crosstab.csv` |
| 22 | Methodology × topic breakdown; export `topic_methodology_crosstab.csv` |
| 23 | Visualisations: stacked bar (journal × topic), heatmap, methodology profile per topic, topics-over-time |
| 24 | Final results summary |

**Key packages:** `pandas`, `matplotlib`, `seaborn`, `numpy`, `math`

---

## Data Flow

```
Scopus_Screening_results.xlsx
        │
        ▼
01_Data_pre-processing.ipynb
        │
        ├── papers_preprocessed.csv
        ├── papers_MA_journals.csv   ◄─ primary corpus for topic modelling
        └── ...
                │
                ▼
        02_Topic_modelling.ipynb
                │
                ├── embeddings.npy
                ├── papers_with_topics.csv
                ├── topic_labeling_input.txt
                └── fig_topic_*.png / fig_topic_*.html
                        │
                        ▼
                03_AI_Labelling.ipynb
                        │
                        ├── papers_with_labels.csv
                        ├── topic_journal_crosstab.csv
                        ├── topic_methodology_crosstab.csv
                        └── fig_journal_*.png / fig_methodology_topic.png
```

---

## Requirements

```bash
pip install pandas matplotlib seaborn nltk openpyxl \
            bertopic sentence-transformers umap-learn hdbscan \
            scikit-learn plotly numpy
```

Python 3.9+ recommended.

---

## Data Sources

Articles were exported from [Scopus](https://www.scopus.com) across **10 accounting journals** for the period **2021–2026**:

| Journal |
|---------|
| Accounting Review |
| Contemporary Accounting Research |
| Review of Accounting Studies |
| European Accounting Review |
| Journal of Accounting and Economics |
| Journal of Accounting Research |
| Accounting, Organizations and Society |
| Journal of Management Accounting Research |
| Journal of Management Control |
| Management Accounting Research |
