# Documentation: Predicting Phage–Host Interactions

This document covers the background, methods and results of the project. For a short overview and setup instructions, see the [README](README.md).

## Background

The overuse of antibiotics has driven the spread of antibiotic resistance, and developing new antibiotics with current strategies is slow and expensive. **Phage therapy** offers another approach: it uses lytic bacteriophages (viruses that infect bacteria) to kill pathogenic bacteria.

Phage–host interactions are highly specific, so a phage only works against the right bacterial host. This project tests whether machine learning can identify compatible phage–host pairs from features computed on protein data.

## Pipeline

```
PhagesDB positive pairs ──► GenBank protein sequences ──► ProtParam features
                                                              │
                                                              ▼
                              K-means sampling of reliable negatives
                                                              │
                                                              ▼
                       Balanced dataset ──► Random Forest / KNN / Neural Network
```

### 1. Dataset creation

- Positive (known) interactions come from [PhagesDB.org](https://phagesdb.org).
- GenBank accession codes are retrieved for phages and hosts that have complete genomes.
- Unnamed proteins, hypothetical proteins and sequences shorter than 30 residues are excluded.

### 2. Feature encoding

Genomes contain different numbers of proteins, so each genome is summarised with statistics computed across all of its proteins. Protein primary structures are analysed with the Biopython `ProtParam` module.

For each protein, 26 base features are computed:

- 20 amino-acid counts
- 5 elemental compositions (C, H, O, N, S)
- 1 molecular weight

The **mean**, **standard deviation** and **variance** of each base feature are then taken across the genome (26 × 3 = 78 features per organism). Doing this for both the phage and the host gives **156 numeric features** per phage–host pair.

Column naming:

| Pattern | Meaning | Example |
|---|---|---|
| `<feature>_avg/_std/_var` | statistic across the genome | `A_avg_p` |
| `_p` / `_h` suffix | phage or host feature | `MolW_var_h` |
| trailing `-` | element rather than amino-acid code | `C-_std_p` (carbon) vs `C_std_p` (cysteine) |

### 3. Building negative examples with K-means ([k-means-clustering.ipynb](k-means-clustering.ipynb))

No public database records experimentally verified *negative* phage–host interactions. The notebook builds a putative negative set as follows:

1. Load `final_interaction_features.csv` (417,529 pairs × 159 columns) and drop rows with missing values, leaving 403,788 pairs (2,591 positive and 401,197 unlabelled/negative).
2. Min–max normalise the numeric features.
3. Choose *k* for K-means on the negative pairs with the elbow method (*k* = 7).
4. From each cluster, keep the 20% of points closest to the centroid. This keeps the variety in the data and gives 80,236 representative negatives.
5. Compute the mean Euclidean distance from each representative to every positive pair, and keep the 2,591 points that are furthest from the positives. These are treated as the most reliable negatives.
6. Combine them with the positives into a **balanced dataset** of 5,182 pairs, saved as `balanced_dataset.csv`.

### 4. Classification models ([all-ML-models.ipynb](all-ML-models.ipynb))

The balanced dataset is split into training, validation and test sets (81% / 9% / 10%, `random_state=42`). The `phage`, `host` and `interaction` columns are removed before training, so each model sees only the 156 features.

| Model | Tuning |
|---|---|
| **Random Forest** | `RandomizedSearchCV` (5-fold) over `n_estimators` ∈ {100–400} and `max_depth` ∈ {1–30}. Best: 100 trees, depth 20 |
| **K-Nearest Neighbours** | 5-fold CV over *k* = 2–29; model uses *k* = 3 |
| **Neural Network** (PyTorch) | 156 → 128 → 64 → 1 MLP with ReLU and sigmoid output, BCE loss, AdamW, StepLR scheduler, batch size 32; grid search over learning rate (4e-5 to 5e-3) and epochs (10–35) |

## Results

Performance on the held-out test set (519 pairs):

| Model | Accuracy | Precision | Recall |
|---|---|---|---|
| Random Forest | **0.996** | **0.996** | **0.996** |
| K-Nearest Neighbours | 0.960 | 0.961 | 0.958 |
| Neural Network | 0.929 | 0.924 | 0.935 |

Random Forest performed best by a clear margin.

> **Note:** The negatives were chosen to be as *dissimilar* from the positives as possible. That makes the classes easy to separate, so these scores probably overestimate how well the models would do on real, unscreened phage–host pairs.

## References

- Leprince, A., & Mahillon, J. (2023). Phage adsorption to gram-positive bacteria. *Viruses*, 15(1), 196.
- [PhagesDB](https://phagesdb.org): source of the positive phage–host interactions.
