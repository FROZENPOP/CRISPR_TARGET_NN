# CRISPR gRNA On-Target Efficiency Prediction

Deep-learning models that predict how efficiently a CRISPR-Cas9 guide RNA will cut at its intended
target, and a comparison against **Azimuth**, the established model-based approach.

This repository holds the code, data and supplementary material for:

> **Investigating the Efficiency of Model-Based Approaches in Designing Guide RNAs for CRISPR**
> Shreyas Sindhaval

---

## Why this exists

In CRISPR-Cas9 editing, the guide RNA (gRNA) determines *where* Cas9 cuts. A good gRNA maximizes
**on-target activity** while minimizing **off-target** cleavage, so guide choice largely decides
whether an edit succeeds. Testing candidate guides in the lab is slow and expensive, which is why
predictive models are used to rank guides before anyone touches a bench.

The study asks a direct question: **how reliable are those models, and can a small neural network
beat them?** Azimuth — which combines Rule Set 2 on-target scoring with CFD off-target scoring —
was scored against Doench et al. knockout-efficiency data and reached an **MSE of 0.001245**,
confirming that conventional machine learning is already dependable. Two neural networks were then
trained on the same standardized data, and the recurrent one edged past that baseline.

---

## Results

Mean Squared Error against the normalized efficiency labels (lower is better):

| Model | Sequence branch | Held-out test (80/20) | Retrained on all rows |
|---|---|---|---|
| `crispr-keras` | CNN only | 0.0025 | 0.00072 |
| `crispr-rnn` **(recommended)** | CNN + GRU | **0.0021** | **0.0012** |
| Azimuth (baseline) | — | 0.001245 | — |

**Takeaways**

- Both networks reach low error, and the **GRU model beats the CNN-only model** at every split —
  dropping the recurrent layers raised the risk of under/over-fitting.
- Trained on the full dataset, the GRU model reaches **0.0012 vs Azimuth's 0.001245**, i.e. a simple
  neural network matches and slightly surpasses an advanced, purpose-built machine-learning model.

> **Note on evaluation.** The "retrained on all rows" column is measured on rows the model also
> trained on, so it is not a held-out estimate; it is reported because it is the configuration used
> for the headline comparison in the paper. The 80/20 column is the honest generalization figure.

---

## Repository contents

| File | What it is |
|---|---|
| `crispr-rnn.ipynb` | CNN + GRU hybrid model. **Best performer.** |
| `crispr-keras.ipynb` | CNN-only variant, kept for the ablation comparison. |
| `FINAL_FILTERED_CRISPR_DATA.csv` | Training data — 5,310 guides, filtered and standardized. |
| `SupplementaryTable1.xlsx` | Full supplementary table (same schema, unfiltered). |

### Dataset columns

Assembled and standardized from **DepMap**, **GeCKOv1** and **Doench et al.** knockout-efficiency
data, so that predictions from different sources could be compared on one scale.

| Column | Role |
|---|---|
| `30mer` | Guide sequence — model input |
| `Percent Peptide` | Model input |
| `Amino Acid Cut position` | Model input |
| `score_drug_gene_rank` | Model input |
| `predictions` | **Training target** — normalized on-target efficiency |
| `model_predictions`, `predictions(30mer)` | Azimuth outputs, for comparison |
| `Target gene`, `drug`, `score_drug_gene_threshold` | Metadata, not fed to the model |

The `30mer` is 4 nucleotides upstream + the 23-nucleotide protospacer with a canonical **NGG** PAM
+ 3 nucleotides downstream.

---

## Model architecture

Both models are two-branch Keras functional models. The branches differ in one layer only, which is
precisely what the ablation tests.

**Sequence branch** — nucleotides are tokenized at character level and embedded, rather than
one-hot encoded, to avoid the high dimensionality that hurts generalization:

```
30mer → Embedding(5 → 16) → Conv1D(64, k=3, relu) → MaxPool(2)
                          → Conv1D(128, k=3, relu) → MaxPool(2)
                          → Flatten        # crispr-keras
                          → GRU(64)        # crispr-rnn
```

The convolutions pick up local motifs — effectively learned k-mers — while pooling cuts
dimensionality and the GRU captures dependencies spanning the whole 30mer.

**Numeric branch** — three tabular features, with batch norm and dropout against overfitting:

```
[Percent Peptide, Amino Acid Cut position, score_drug_gene_rank]
  → BatchNorm → Dense(256, relu) → BatchNorm → Dropout(0.3)
              → Dense(256, relu) → BatchNorm → Dropout(0.3)
              → Dense(128, relu)
```

**Head** — `Concatenate → Dense(64, relu) → Dense(1, linear)`, giving an efficiency score in the
0–1 range.

**Training** — Adam, MSE loss, 28 epochs, batch size 32, 80/20 split (`random_state=42`).
Epoch count was chosen from the loss curves and dataset size. `StandardScaler` is fit on the
training split only; the character tokenizer is fit on the sequences.

---

## Usage

The notebooks were written on Kaggle and read the dataset from `../input/`. To run them locally,
point `pd.read_csv` at the CSV in this repository:

```python
cas = pd.read_csv('FINAL_FILTERED_CRISPR_DATA.csv')
```

Install the dependencies with:

```bash
pip install -r requirements.txt
```

No trained weights are checked in, so run `crispr-rnn.ipynb` top to bottom first. That leaves
`model` (80/20 split), `modelall` (all rows), `tokenizer` and `scaler` in scope, after which you can
score a new guide:

```python
test_sequence = "CAGAAAAAAAAACACTGCAACAAGAGGGTA"   # 30mer

# A DataFrame with named columns avoids a feature-name mismatch in the scaler
test_num_feat = pd.DataFrame(
    [[72.87, 360, 0.083682008]],
    columns=['Percent Peptide', 'Amino Acid Cut position', 'score_drug_gene_rank']
)

test_sequence_padded = pad_sequences(
    tokenizer.texts_to_sequences([test_sequence]), maxlen=30
)
test_numerical_features_scaled = scaler.transform(test_num_feat)

prediction = modelall.predict([test_sequence_padded, test_numerical_features_scaled])
print(f"Predicted Efficiency: {prediction[0][0]}")
```

The sequence must be exactly 30 nucleotides in the layout described above, or the score is
meaningless — the model learned position-specific motifs.

---

## Conclusion

Conventional machine learning for on-target specificity is already reliable, but deep learning gives
a more robust tool and higher reliability for gRNA design. That a *simple* neural network can
outperform an advanced machine-learning method points to real headroom for deep learning here, and
to more accessible CRISPR-based therapeutics as these models improve.

One caveat for anyone extending this work: adding parameters such as GC content or cell type will
require considerably more training data before the model stays reliable.

---

## Citation

If you use these models, please cite:

> Sindhaval, S. *Investigating the Efficiency of Model-Based Approaches in Designing Guide RNAs for
> CRISPR.*

---

## License

The code in this repository is released under the [MIT License](LICENSE).

The data files — `FINAL_FILTERED_CRISPR_DATA.csv` and `SupplementaryTable1.xlsx` — are **not**
covered by it. They are derived from the supplementary data of Doench et al. (2016) together with
DepMap and GeCKOv1, each of which carries its own terms; cite those sources directly if you reuse
the data. See [NOTICE](NOTICE) for details.

---

## References

- Doench, J. G. et al. "Optimized sgRNA design to maximize activity and minimize off-target effects
  of CRISPR-Cas9." *Nature Biotechnology* 34(2), 2016. https://doi.org/10.1038/nbt.3437
- Konstantakos, V. et al. "CRISPR-Cas9 gRNA efficiency prediction: an overview of predictive tools
  and the role of deep learning." *Nucleic Acids Research* 50(7), 2022.
  https://doi.org/10.1093/nar/gkac192
- Sherkatghanad, Z. et al. "Using traditional machine learning and deep learning methods for on- and
  off-target prediction in CRISPR/Cas9: a review." *Briefings in Bioinformatics* 24(3), 2023.
  https://doi.org/10.1093/bib/bbad131
- Li, J. et al. "Machine learning-based prediction models to guide the selection of Cas9 variants for
  efficient gene editing." *Cell Reports* 43(2), 2024. https://doi.org/10.1016/j.celrep.2024.113765
- Redman, M. et al. "What is CRISPR/Cas9?" *Archives of Disease in Childhood: Education and Practice*,
  2016. https://pmc.ncbi.nlm.nih.gov/articles/PMC4975809/
- Wilkinson, R. A. et al. "CRISPR RNA-guided autonomous delivery of Cas9." *Nature Structural &
  Molecular Biology*, 2018. https://www.nature.com/articles/s41594-018-0173-y

**Data sources** — [DepMap](https://depmap.org/portal/data_page/?tab=allData) ·
[GeCKOv1](https://sourceforge.net/projects/mageck/files/libraries/GeCKOv1.txt.zip/download) ·
Doench et al. Rule Set 2 data
