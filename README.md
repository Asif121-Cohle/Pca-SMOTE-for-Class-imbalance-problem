# PCA-SMOTE: Diagnosing and Correcting Synthetic Overfitting

**Paper:** *PCA-SMOTE: Diagnosing and Correcting Synthetic Overfitting in Deep Latent Space Oversampling for Imbalanced Image Classification*

This repository accompanies research on deep latent-space oversampling under **extreme class imbalance**. The paper investigates why aggressively generating synthetic minority-class images can hurt downstream classification and proposes **PCA-SMOTE**, which restricts SMOTE interpolation to a lower-dimensional principal-component subspace.

## Overview

Class imbalance can cause image classifiers to favor majority classes and perform poorly on underrepresented classes. DeepSMOTE addresses this by encoding images into a latent space, applying SMOTE to minority-class embeddings, and decoding the synthetic embeddings back into images.

The paper identifies a failure mode called **synthetic overfitting**: when a minority class has very few real samples, oversampling it all the way to the majority-class count can produce a synthetic distribution that does not represent the true class distribution. The resulting classifier may perform worse than a model trained without synthetic augmentation.

PCA-SMOTE addresses this issue by applying Principal Component Analysis (PCA) to minority-class latent vectors before SMOTE interpolation, then mapping the synthetic vectors back into the original latent space for decoding.

## Key Contributions

- **Synthetic overfitting analysis:** Examines how aggressive latent-space oversampling can reduce minority-class classification accuracy.
- **Latent-space sparsity diagnostic:** Uses a normalized nearest-neighbor distance ratio to characterize weak local structure in extremely sparse minority classes.
- **PCA-SMOTE:** Restricts interpolation to a lower-dimensional PCA subspace to reduce the effect of high-dimensional noise.
- **Target-count selection:** Finds that generating approximately five times the number of real minority samples works well in the evaluated settings, rather than always balancing every class completely.
- **FID–accuracy decoupling:** Shows that better Fréchet Inception Distance (FID) does not necessarily imply better downstream classification performance.
- **Statistical evaluation:** Compares methods across three random seeds and reports Wilcoxon signed-rank tests.

## Method

The PCA-SMOTE pipeline follows these steps:

1. **Train an autoencoder:** Train the DeepSMOTE-style encoder–decoder on the imbalanced image dataset.
2. **Encode minority-class images:** Obtain latent vectors for the minority class.
3. **Project with PCA:** Fit PCA to the minority-class latent vectors and retain the leading principal components. The paper uses `k = 20` in its experiments.
4. **Apply SMOTE:** Generate synthetic vectors in the reduced-dimensional subspace.
5. **Inverse-project:** Map the synthetic vectors back to the original latent space.
6. **Decode synthetic images:** Use the trained decoder to produce synthetic minority-class images.
7. **Select a target count:** Use a validation-guided target rather than automatically oversampling to full class balance.
8. **Train and evaluate the classifier:** Combine real and synthetic training samples and evaluate minority-class classification performance.

## Datasets and Experimental Setup

| Dataset | Imbalance configuration | Minority class |
|---|---|---|
| MNIST | 100:1 | Digit 9; 40 real training samples |
| Four-class CIFAR-10 subset | 25:1 | Airplane; 80 real training samples |

The CIFAR-10 subset contains **Ship, Dog, Bird, and Airplane**. A lightweight SmallCNN is trained from scratch for downstream classification to isolate the effect of the oversampling strategy. Experiments use random seeds **10, 20, and 60**.

## Reported Results

### MNIST: Minority-class accuracy

| Method | Seed 10 | Seed 20 | Seed 60 | Mean |
|---|---:|---:|---:|---:|
| Baseline (unaugmented) | 87.6% | 87.1% | 88.6% | 87.8% |
| DeepSMOTE (full balance) | 79.9% | 84.1% | 82.4% | 82.1% |
| PCA-SMOTE (full balance) | 85.6% | 87.1% | 87.0% | 86.6% |
| PCA-SMOTE (optimal target) | 88.9% | 88.9% | 88.6% | **88.8%** |

### CIFAR-10: Airplane minority-class accuracy

| Method | Seed 10 | Seed 20 | Seed 60 | Mean |
|---|---:|---:|---:|---:|
| Baseline (unaugmented) | 23.5% | 18.3% | 24.9% | 22.2% |
| DeepSMOTE (full balance) | 19.9% | 14.8% | 14.2% | 16.3% |
| PCA-SMOTE (full balance) | 22.2% | 17.1% | 20.4% | 19.9% |
| PCA-SMOTE (optimal target) | 22.2% | 25.1% | 26.9% | **24.7%** |

The paper reports a Wilcoxon signed-rank test result of **p = 0.0013** for PCA-SMOTE's improvement over DeepSMOTE across the evaluated paired configurations. It also reports **p = 0.0020** for the degradation of standard DeepSMOTE relative to the unaugmented baseline under extreme imbalance.

### FID versus downstream accuracy

For MNIST Class 9 (seed 10), the paper reports:

| Method | FID (lower is better) | Minority-class accuracy |
|---|---:|---:|
| DeepSMOTE | **20.5** | 79.9% |
| PCA-SMOTE | 32.2 | **85.6%** |

This comparison illustrates that image-level perceptual quality and downstream classifier utility are not interchangeable measures.

## Main Findings

- Aggressive oversampling can make minority-class accuracy worse than the unaugmented baseline.
- Restricting interpolation to a PCA subspace can mitigate harmful high-dimensional latent-space interpolation.
- The best synthetic target in these experiments is approximately **5× the real minority-class count**, rather than full class balance.
- FID alone is not sufficient for choosing an oversampling method.
- PCA-SMOTE improves mean minority-class accuracy over full-balance DeepSMOTE on both evaluated benchmarks.

## Limitations and Future Work

The study evaluates one minority class per dataset and uses three random seeds. Its results are limited to MNIST and a four-class subset of CIFAR-10, so broader validation is needed. Future work identified in the paper includes:

- Evaluating more minority classes and additional random seeds.
- Conducting systematic ablation and sensitivity studies for the PCA dimension and synthetic target count.
- Exploring nonlinear dimensionality-reduction methods, such as UMAP, before latent-space interpolation.
- Evaluating more expressive encoder–decoder architectures and larger image datasets.

## Reproducibility

The paper describes a DeepSMOTE-style autoencoder, PCA-based latent-space projection, SMOTE interpolation, inverse PCA projection, decoding, and downstream SmallCNN classification. Exact setup commands, dependency versions, dataset preparation scripts, and training configurations should be documented alongside the implementation when available.

**Note:** The README reports results and settings described in the accompanying paper. It does not claim that experiments can be reproduced from this repository until the required code, data instructions, and environment specifications are included.

## Citation

If you use this work, please cite the paper:

> Shahad Abir, Shahriar Khaled Fahim, Abdullah Mohammed Asif, Piash Islam, and Mahfuzulhoq Chowdhury. “PCA-SMOTE: Diagnosing and Correcting Synthetic Overfitting in Deep Latent Space Oversampling for Imbalanced Image Classification.” 2026 IEEE International Conference on Optics, Machine Learning and Emerging Technology (OMLET), Nairobi, Kenya, 29–31 October 2026.

## Authors

- Shahad Abir
- Shahriar Khaled Fahim
- Abdullah Mohammed Asif
- Piash Islam
- Mahfuzulhoq Chowdhury

**Affiliation:** Department of Computer Science and Engineering, Chittagong University of Engineering & Technology (CUET), Bangladesh.
