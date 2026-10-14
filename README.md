# Predicting Phage–Host Interactions for Phage Therapy

## Objective

Phage therapy uses bacteriophages (viruses that infect bacteria) to treat bacterial infections, but each phage only infects specific hosts. This project uses machine learning to predict whether a given phage will infect a given bacterial host.

## What was done

Protein-level features (amino-acid composition, elemental composition and molecular weight) were computed for phage and host genomes, and K-means clustering was used to select reliable negative examples for a balanced dataset. Random Forest, K-Nearest Neighbours and a neural network were then trained on it; Random Forest performed best, with 99.6% test accuracy.

Full details on the methods, results and references are in [DOCUMENTATION.md](DOCUMENTATION.md).

## Tech stack

- Python, Jupyter / Google Colab
- pandas, NumPy
- scikit-learn (K-means, Random Forest, KNN)
- PyTorch (neural network)
- Biopython (`ProtParam` feature extraction)
- matplotlib, seaborn

## Files

| File | Description |
|---|---|
| [k-means-clustering.ipynb](k-means-clustering.ipynb) | Preprocessing, K-means clustering and construction of the balanced dataset |
| [all-ML-models.ipynb](all-ML-models.ipynb) | Training, tuning and evaluation of the Random Forest, KNN and neural network models |
| [Final presentation.pptx](Final%20presentation.pptx) | Project presentation slides |
| [DOCUMENTATION.md](DOCUMENTATION.md) | Detailed background, methods, results and references |

The data files (`final_interaction_features.csv` and `balanced_dataset.csv`) are not included in this repository.

## How to run

1. Install the dependencies:

   ```bash
   pip install pandas numpy scikit-learn matplotlib seaborn torch biopython
   ```

2. The notebooks were written for Google Colab and read data from Google Drive. Change the `read_csv` paths at the top of each notebook to point to your copies of the data files. If you run locally, also remove the `drive.mount` cell.
3. Run `k-means-clustering.ipynb` first to produce the balanced dataset, then run `all-ML-models.ipynb`.
