  The CRISPR/Cas9 system provides state-of-the art genome editing capabilities. However, the choice of guide RNA heavily restricts its ability to be practically applied. A good gRNA maximizes on-target activity while minimizing potential off-target effects, making it critical for successful gene editing. Consequently, picking a well-suited gRNA is critically important in using CRISPR, however, determining one can eat up resources and time.

  The deep-learning pipelines above aim to stramline this process by providing a probability between 0 and 1 of a gRNA cutting at its intended target. The parameters needed are: 30 nucleotide long gRNA sequence(which includes 4 nucleotides upstream and 3 nucleotides downstream of the PAM sequence, and the 23 nucleotide protospacer sequence with a canonical PAM sequence (NGG)), the percent peptide, the amino acid cut position, and the gene drug rank score.

  The above models, crispr-keras and crispr-rnn, differ slightly. The crispr-rnn model uses RNN layers in conjunction with CNN layers while the crispr-keras model only uses CNN layers for its semantic encoding. It is reccomended to use crispr-rnn as it has proved to be more reliable.

# Publications: Please cite this paper if using the predictive model:

Shreyas Sindhaval

Investigating the Efficiency of Model-Based Approaches in Designing Guide RNAs for CRISPR


# To use the model to predict use this code:

```
test_sequence = "[30mer]"
test_num_feat = pd.DataFrame(
    [[insertPercentPeptide], [insertAminoAcidCutPosition], [insert_score_drug_gene_rank]], 
    columns=['Percent Peptide', 'Amino Acid Cut position', 'score_drug_gene_rank']
)  # Using dataframes to avoid a feature error


test_sequence_encoded = tokenizer.texts_to_sequences([test_sequence])
test_sequence_padded = pad_sequences(test_sequence_encoded, maxlen=30)
test_numerical_features_scaled = scaler.transform(test_num_feat)


prediction = [model_name].predict([test_sequence_padded, test_numerical_features_scaled])
print(f"Predicted Efficiency: {prediction[0][0]}")
```
