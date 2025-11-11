# CRISPR gRNA Efficiency Prediction

The **CRISPR/Cas9 system** provides state-of-the-art genome editing capabilities. However, the choice of guide RNA (gRNA) heavily restricts its practical applications.  

A good gRNA must:  
- **Maximize on-target activity**  
- **Minimize potential off-target effects**  

This makes the selection of a well-suited gRNA critically important. However, identifying one can consume significant time and resources.

---

## 🔬 About the Models

The deep-learning pipelines included here streamline gRNA selection by providing a **probability (0 to 1)** that a gRNA will cut at its intended target.  

### Required Input Parameters:
- **30 nucleotide long gRNA sequence**, which includes:
  - 4 nucleotides upstream  
  - 3 nucleotides downstream of the PAM sequence  
  - 23 nucleotide protospacer sequence with canonical PAM (NGG)  
- **Percent peptide**  
- **Amino acid cut position**  
- **Gene drug rank score**

---

## Models

- **`crispr-keras`**  
  - Uses only CNN layers for semantic encoding.  

- **`crispr-rnn`** *(recommended)*  
  - Combines RNN layers with CNN layers.  
  - Demonstrated to be more reliable.  

---

## Publications

If you use these predictive models, please cite:  

**Shreyas Sindhaval**  
*Investigating the Efficiency of Model-Based Approaches in Designing Guide RNAs for CRISPR*  

---

## Usage Example

```python
# Example usage to predict gRNA efficiency
test_sequence = "[30mer]"

test_num_feat = pd.DataFrame(
    [[insertPercentPeptide], 
     [insertAminoAcidCutPosition], 
     [insert_score_drug_gene_rank]], 
    columns=['Percent Peptide', 'Amino Acid Cut position', 'score_drug_gene_rank']
)  # DataFrame avoids feature mismatch errors

# Encode sequence
test_sequence_encoded = tokenizer.texts_to_sequences([test_sequence])
test_sequence_padded = pad_sequences(test_sequence_encoded, maxlen=30)

# Scale numerical features
test_numerical_features_scaled = scaler.transform(test_num_feat)

# Predict efficiency
prediction = [model_name].predict([test_sequence_padded, test_numerical_features_scaled])
print(f"Predicted Efficiency: {prediction[0][0]}")
```
