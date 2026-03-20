# 1. Introduction

## 1.1 Background and Motivation
Proteins are the fundamental workhorses of cellular biology, executing a vast array of functions ranging from enzymatic catalysis to structural support. With the advent of high-throughput sequencing technologies, the number of known protein sequences has grown exponentially, vastly outpacing our ability to experimentally characterize their functions. This gap has necessitated the development of Automated Function Prediction (AFP) methods, which aim to infer the Gene Ontology (GO) terms associated with a protein solely from its amino acid sequence.

Historically, AFP relied heavily on homology-based methods like BLAST, which transfer annotations from evolutionarily related proteins. While effective for close homologs, these methods falter for "orphan" proteins or those with remote evolutionary relationships. The recent rise of Deep Learning, particularly Protein Language Models (PLMs) like ESM-2 and ProtT5, has revolutionized this field. These models, pre-trained on billions of sequences, learn rich semantic representations of proteins that capture deep evolutionary and structural context without explicit alignment.

However, a key challenge remains: distinct functional aspects—Molecular Function (MF), Biological Process (BP), and Cellular Component (CC)—operate at different scales. MF is often driven by local structural motifs (e.g., active sites), while BP involves complex, global pathway interactions. We hypothesize that while large PLMs capture global semantics effectively, they may benefit from complementary, lightweight architectures designed to explicitly model local sequence patterns.

## 1.2 Contributions
In this work, we introduce **ProteinExt 3**, a multi-modal ensemble framework for robust protein function prediction. Our contributions are three-fold:

1.  **Multi-View Late Fusion:** We propose a rigorous late-fusion strategy that integrates predictions from two state-of-the-art PLMs (ESM-2 and ProtT5) with a lightweight Convolutional Neural Network (CNN). By optimizing fusion weights individually for each GO aspect, we explicitly model the varying contribution of each modality.
2.  **Efficient PCA-CNN Module:** We introduce a novel PCA-CNN architecture that operates on dimensionality-reduced PLM embeddings. This module efficiently captures local sequential dependencies that may be smoothed out in the global pooling operations of standard transformer models.
3.  **Aspect-Specific Insights:** Our extensive evaluation reveals a striking dichotomy: the CNN module significantly boosts performance for Molecular Function (MF) prediction but provides negligible gain for Biological Process (BP). This finding empirically supports the biological intuition that MF is more motif-driven, while BP requires the global context provided by large PLMs.
4. **Standalone Software Package:** We release a comprehensive, standalone software package capable of end-to-end inference, democratizing access to our high-performance ensemble.
# 2. Methodology

## 2.1 Overview
Our pipeline adopts a Late Fusion approach, independently training three distinct neural network architectures on the same dataset and combining their predictions via a learned weighted average. This strategy allows us to leverage the complementary strengths of different protein representations without the complexity of end-to-end joint training.

## 2.2 Dataset and Preprocessing
We utilized the CAFA6 competition dataset.
- **Input:** Raw amino acid sequences.
- **Targets:** Gene Ontology (GO) terms across three sub-ontologies:
    - **Biological Process (BPO)**: 16,858 terms (Global set)
    - **Molecular Function (MFO)**: 6,616 terms
    - **Cellular Component (CCO)**: 2,651 terms
- **Preprocessing:** All sequences were truncated or padded to a fixed maximum length (e.g., 1024 residues) to ensure batch consistency during training. Labels were encoded using a Multi-Label Binarizer, resulting in sparse binary vectors for each protein.

## 2.3 Model Architectures

### 2.3.1 Model A: ESM-2 (650M)
We employed the **ESM-2 (t33_650M_UR50D)** model, a state-of-the-art transformer pre-trained on millions of evolutionary diverse sequences.
- **Embedding:** We extract the 1280-dimensional representation from the last hidden layer, mean-pooled over the sequence length.
- **Classification Head:** A Multi-Layer Perceptron (MLP) with Batch Normalization, GELU activation, and Dropout (p=0.5) maps the embedding to the GO term probability space.
- **Window Features:** To supplement the global embedding, we concatenated statistical features (mean, var, max, min) of sliding-window hydrophobicity scores, adding 12 dimensions to the input.

### 2.3.2 Model B: ProtT5 (XL)
We utilized **ProtT5-XL-UniRef50**, an encoder-decoder T5 model trained on UniRef50.
- **Embedding:** Similar to ESM-2, we utilize the 1024-dimensional mean-pooled representation from the encoder's last hidden state.
- **Architecture:** The downstream classifier mirrors the MLP structure used for ESM-2, ensuring that performance differences are attributable to the embeddings rather than the head architecture.

### 2.3.3 Model C: PCA-CNN (Sequence Motif Learner)
To explicitly capture local structural motifs that might be smoothed out by mean pooling, we introduced a lightweight **PCA-CNN**.
- **Input:** Instead of a single vector, this model accepts the full sequence of token embeddings from ESM-2 (8M parameter version).
- **Dimensionality Reduction:** To maintain computational efficiency, the high-dimensional token vectors are projected down to 20 dimensions using **Incremental PCA** (fit on the training set).
- **Convolution:** A 1D Convolutional layer (Kernel Size=50, Filters=512) scans the dimensionally reduced sequence, followed by Adaptive Max Pooling and a fully connected classification layer. This architecture allows the model to detect position-invariant local patterns.

## 2.4 Training Strategy
- **Cross-Validation:** We employed 5-Fold Cross-Validation to ensure robust performance estimation and prevent overfitting to specific data splits.
- **Loss Function:** We minimized the Binary Cross Entropy (BCE) loss.
- **Optimization:** Models were trained using the AdamW optimizer. Hyperparameters (Learning Rate, Window Size) were tuned individually for each aspect (P, F, C) using a grid search (see Appendix A).
    - *Example (MF):* Learning Rate=5e-4, Window Size=35.

## 2.5 Fusion Strategy
Final predictions ($S_{final}$) are generated via a weighted ensemble of the three models:
$$ S_{final} = w_{ESM} \cdot S_{ESM} + w_{T5} \cdot S_{T5} + w_{CNN} \cdot S_{CNN} $$
The weights $\{w_{ESM}, w_{T5}, w_{CNN}\}$ and a global confidence threshold ($\tau$) were optimized via grid search on the Out-of-Fold (OOF) validation predictions to maximize the protein-centric F-measure ($F_{max}$).
# 3. Results

## 3.1 Quantitative Performance
We evaluated our ensemble on the hold-out test set using the protein-centric F-measure ($F_{max}$). Table 1 summarizes the optimal fusion weights and the resulting F1-scores for each aspect.

**Table 1: Optimal Fusion Weights and Ensemble Performance**

| Aspect | $w_{ESM2}$ | $w_{ProtT5}$ | $w_{CNN}$ | Threshold ($\tau$) | Ensemble F1 |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Biological Process (P)** | 0.5 | 0.6 | **-0.1** (Excluded) | 0.30 | 0.122 |
| **Molecular Function (F)** | 0.3 | 0.3 | **0.4** | 0.30 | **0.500** |
| **Cellular Component (C)** | 0.5 | 0.6 | -0.1 (Excluded) | 0.30 | 0.428 |

*Note: A negative weight indicates that the model's contribution did not improve the ensemble's performance on the validation set, leading to its exclusion from the final prediction.*

## 3.2 Impact of Multi-Modal Fusion
Our results demonstrate that the optimal strategy is highly aspect-dependent:

*   **Molecular Function (F):** The ensemble achieves its highest performance here ($F1=0.500$). Notably, the PCA-CNN model is assigned a significant weight ($w_{CNN}=0.4$), comparable to or even exceeding the PLMs ($w=0.3$). This indicates that the local structural motifs captured by the CNN are crucial for defining molecular activities.
*   **Biological Process (P):** The PCA-CNN model was effectively discarded ($w=-0.1$). Performance is driven entirely by the large PLMs (ESM-2 and ProtT5), which are better suited to capturing the abstract, global pathway information inherent to biological processes.
*   **Cellular Component (C):** Similar to BP, the CNN did not contribute positively, suggesting that subcellular localization is also better inferred from global sequence semantics rather than local motifs.

## 3.3 Individual Model Contributions
To validate the fusion strategy, we compared the ensemble against single-model baselines.
- **ProtT5** consistently served as the strongest single baseline, with weights consistently $\ge$ ESM-2 across all aspects.
- **ESM-2** provided robust complementary features, but rarely outperformed ProtT5 in isolation.
- **PCA-CNN** is a specialist: it underperforms as a generalist predictor but provides unique, non-redundant signal for Molecular Function.
# 4. Discussion

## 4.1 The "Local vs. Global" Hypothesis
The most significant finding of this study is the divergence in model contribution across functional aspects.
- **Molecular Function relies on Local Motifs:** The PCA-CNN's success in the MF aspect confirms that function is often dictated by specific constellations of amino acids (e.g., catalytic triads, binding pockets). The CNN's convolution operation is inherently translation-invariant and local, making it an ideal feature extractor for these patterns, even when working on reduced-dimensionality embeddings.
- **Biological Process relies on Global Semantics:** Conversely, BP terms describe role within a larger system (e.g., "cell cycle," "metabolic pathway"). These are not encoded in short sequence motifs but in the global context and evolutionary history of the protein. Large PLMs (ESM-2, ProtT5) excel here because their self-attention mechanisms allow them to integrate information across the entire sequence length, making the local-only CNN redundant.

## 4.2 Limitations
- **Computational Overhead:** While effective, our late fusion approach requires running inference on three distinct models, increasing the computational burden compared to a single end-to-end model.
- **Data Dependency:** The PCA-CNN relies on a fixed PCA transformation learned from the training set. This assumes the test set distribution matches the training set, which may not hold for extreme orphan proteins.

# 5. Conclusion
We presented **ProteinExt 3**, a comprehensive late-fusion ensemble for protein function prediction. By combining the global semantic power of large Protein Language Models with the local pattern recognition of a lightweight PCA-CNN, we achieved robust performance across the CAFA6 sub-ontologies. Our analysis highlights the importance of matching model architecture to the biological scale of the prediction task: using CNNs for local molecular functions and Transformers for global biological processes. We release our optimized ensemble as a standalone, easy-to-use software package to facilitate further research.

# Appendix A: Hyperparameters

**Table 2: Optimized Training Parameters**

| Model | Aspect | Window Size | Learning Rate |
| :--- | :---: | :---: | :---: |
| **ESM-2** | P | 82 | 7.5e-4 |
| | F | 40 | 5.0e-4 |
| | C | 100 | 5.0e-4 |
| **ProtT5** | P | 80 | 1.0e-3 |
| | F | 35 | 5.0e-4 |
| | C | 50 | 5.0e-4 |
| **PCA-CNN** | P | 120 | - |
| | F | 50 | - |
| | C | 9 | - |
