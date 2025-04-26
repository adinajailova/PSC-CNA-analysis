## Installation
Clone the repository and install dependencies:

```bash
git clone https://github.com/yourusername/infercnv_better.git
cd infercnv_better
pip install -r requirements.txt
```

---

## Usage

You must start with an `AnnData` object (`adata`) that contains:
- Raw **counts** matrix (`adata.X` or a specified layer).
- Gene metadata (`adata.var`) including at least `chromosome`, `start`, and `end` columns.

---

## 1. Inferring CNAs with `infercnv()`

The `infercnv` function detects copy number alterations (CNAs) from single-cell RNA-seq data.  
You can now choose between **reference-based** normalization or **gene-wise Z-score** normalization.

### Example: Using gene Z-score normalization

```python
import infercnvpy as cnv

# Perform CNA inference using Z-scoring
cnv.tl.infercnv(
    adata,
    normalization_mode="zscore",  # 📌 NEW option
    window_size=100,
    step=10,
    lfc_clip=3,
    dynamic_threshold=1.5,
    key_added="cnv_zscore",
)

# Result:
# - CNA smoothed matrix is stored in adata.obsm["X_cnv_zscore"]
# - Genomic position info in adata.uns["cnv_zscore"]
```

---

### Example: Using traditional reference-based normalization

```python
# Perform CNA inference using a reference group
cnv.tl.infercnv(
    adata,
    reference_key="cell_type",
    reference_cat="Normal",
    normalization_mode="reference",  # 📌 Default
    window_size=100,
    step=10,
    lfc_clip=3,
    dynamic_threshold=1.5,
    key_added="cnv_reference",
)

# Result:
# - CNA smoothed matrix is stored in adata.obsm["X_cnv_reference"]
# - Genomic position info in adata.uns["cnv_reference"]
```

---

## 2. Running CopyKAT with Optional Pre-normalization

The `copykat` function calls the R-based CopyKAT method to detect large-scale CNAs.  
You can now **pre-normalize** your data in Python before sending it to R.

### Example: Running CopyKAT with input normalization

```python
cnv.tl.copykat(
    adata,
    normalize_input=True,  # 📌 NEW option
    organism="human",
    window_size=25,
    key_added="copykat_norm",
)

# Result:
# - CopyKAT CNA matrix in adata.obsm["X_copykat_norm"]
# - CopyKAT predicted labels in adata.obs["copykat_norm"]
# - Chromosome info in adata.uns["copykat_norm"]
```

---

## Outputs

| Location | Description |
|:---|:---|
| `adata.obsm['X_<key>']` | Smoothed CNA matrix (cells × genomic bins) |
| `adata.uns['<key>']` | Genomic bin mapping (chromosome, start, end) |
| `adata.obs['<key>']` (only CopyKAT) | Predicted normal/tumor label per cell |

---

## Notes
- **Reference-based normalization** works well when a clean normal cell population is available.
- **Z-score normalization** is recommended when the dataset is heterogeneous or no clear references exist.
- **CopyKAT** requires R and rpy2 installed. See [copykat GitHub](https://github.com/navinlabcode/copykat) for more details.
