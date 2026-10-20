---
title: "Feature-Decorrelation Self-Supervised Learning"
tags: ["self-supervised-learning", "feature-decorrelation", "redundancy-reduction", "whitening", "information-maximization", "barlow-twins", "vicreg"]
last_updated: "2026/08/21"
---

## Summary

Feature-decorrelation SSL is a negative-free family that avoids collapse by regularizing the *embedding dimensions* — pushing feature axes to be decorrelated (and to have non-zero variance) — rather than by contrasting samples or using asymmetric predictors. Barlow Twins (cross-correlation toward the identity) and VICReg (variance–invariance–covariance) are the exemplars; whitening (W-MSE), kernel-dependence (HSIC), and entropy-coding (MEC) methods share the principle. Because it uses symmetric Siamese branches with no predictor, stop-gradient, or momentum, it is mechanistically the cleanest negative-free approach, and a body of theory shows it is dual to [[contrastive_ssl]] and is what [[non_contrastive_ssl]] asymmetry achieves implicitly.

## Related Concepts
[[self_supervised_learning]]
[[contrastive_ssl]]
[[non_contrastive_ssl]]
[[cluster_based_ssl]]
[[teacher_student_ssl]]
[[multimodal_ssl]]
[[dimensional_collapse]]
[[whitening]]
[[mutual_information_maximization]]
[[alignment_and_uniformity]]
[[spectral_methods_in_ssl]]
[[kernel_methods]]
[[image_augmentation_methods]]

## Open Questions

Whether decorrelation, whitening, variance regularization, and entropy maximization are genuinely distinct principles or reparameterizations of one objective is unsettled — MEC recovers both Barlow Twins and SimSiam as leading Taylor terms {2022_Liu_MEC}, and HSIC, InfoNCE, and SSL-HSIC bound the same kernel-dependence quantity {2021_Li_HSIC}{2021_Tsai}. The interaction between decorrelation and embedding normalization is subtle and under-explored: normalization can interfere with decorrelation {2021_Hua}, yet is what separates dimension- from sample-contrastive losses {2023_Garrido}. Why these methods need very high-dimensional projectors, and how whitening stability degrades at scale, remain practically important and incompletely understood {2021_Zbontar}{2021_Ermolov}. Their behavior beyond ImageNet-scale vision (larger models, other modalities) is thinly evidenced.

## Synthesis

### The two exemplars

Barlow Twins {2021_Zbontar} computes the cross-correlation matrix between the embeddings of two augmented views (over a batch) and drives it toward the identity: on-diagonal terms toward 1 (invariance to augmentation) and off-diagonal terms toward 0 (redundancy reduction, after Barlow's neuroscience principle). It uses symmetric branches with no negatives, predictor, or stop-gradient; it works at small batch sizes but prefers very high projector dimensions (still improving at 16384) and depends heavily on strong augmentation, especially color jitter. VICReg {2022_Bardes} replaces the cross-correlation with three explicit terms — a variance hinge keeping each embedding dimension's standard deviation above a threshold (anti-collapse), an invariance MSE between positive pairs, and a covariance term driving within-branch off-diagonal covariances to zero. Because the two branches interact only through the invariance term, VICReg tolerates different architectures/weights per branch, enabling multimodal use. The VICReg paper also introduced an influential four-way taxonomy of SSL — contrastive, clustering, distillation, and information-maximization — that is widely reused.

### Whitening and the decorrelation principle

W-MSE {2021_Ermolov} makes the principle explicit: whiten the batch embeddings (ZCA, so covariance = identity) and then apply MSE on positive pairs; batch normalization alone, without the covariance-flattening of whitening, does not prevent collapse. The decorrelation view was crystallized by {2021_Hua}, one of the first to name *dimensional collapse* in SSL: with a 2-D toy setup, no normalization collapses to a point, plain batch norm collapses to a line (dimensional collapse), and Decorrelated Batch Norm (ZCA whitening of sub-groups) fills the space; they add Shuffled-DBN to decorrelate across randomly permuted dimension groups. A caution raised there — embedding normalization can interfere with decorrelation because it couples dimensions — recurs in later theory. A dedicated study of whitening losses and a channel-whitening variant (CW-RGP) exists but was not consulted here (unread). See [[whitening]], [[dimensional_collapse]].

### Kernel and information-theoretic formulations

Several works recast decorrelation as dependence or entropy maximization. Barlow Twins is shown to (approximately) maximize the Hilbert–Schmidt Independence Criterion — HSIC = MMD between the joint and product-marginal distributions, whose empirical linear-kernel form is the squared Frobenius norm of the cross-correlation — motivating an HSIC-variant loss {2021_Tsai}. SSL-HSIC {2021_Li_HSIC} maximizes kernel dependence between embeddings and image identity, proves InfoNCE and SSL-HSIC bound the same quantity, and connects the objective to alignment/uniformity and to k-means. Maximum Entropy Coding {2022_Liu_MEC} minimizes a log-det ("lossy coding length") surrogate for embedding entropy and, via a Taylor expansion, exhibits SimSiam as the leading term of its batch-wise form and Barlow Twins as the leading terms of its feature-wise form — explicitly bridging *batch-wise* (sample) and *feature-wise* (dimension) methods. An information-maximization-criterion method (Ozsoy et al.) is also relevant but was not consulted here (unread). See [[mutual_information_maximization]], [[kernel_methods]].

### Unification with the other families

Feature decorrelation is now understood as one corner of a connected space. The asymmetry-to-decorrelation bridge shows BYOL/SimSiam implicitly satisfy $W_p^T W_p = F F^T$ (predictor Gram matches the feature auto-correlation), so their asymmetry *is* an implicit decorrelation, and an explicit covariance regularizer can replace the stop-gradient {2022_Liu_Decorrelation} — echoing the implicit-variance-regularization result in [[non_contrastive_ssl]]. Spectral analysis places VICReg as recovering local spectral embeddings (Laplacian eigenmaps / LPP) and Barlow Twins as recovering canonical correlation analysis, while SimCLR recovers global spectral methods (MDS / kernel PCA); because Barlow Twins and SimCLR force lower-rank optima than VICReg, VICReg is argued to be preferable for downstream tasks {2022_Balestriero}. The duality result makes this sharp: dimension-contrastive (decorrelation) and sample-contrastive (InfoNCE) losses are equal up to row/column normalization, and VICReg can be morphed into SimCLR through intermediate variants with matching accuracy {2023_Garrido}. See [[contrastive_ssl]], [[spectral_methods_in_ssl]].

### Terminology

The same family is called *feature decorrelation*, *redundancy reduction*, *whitening*, *information/entropy maximization*, and *dimension-contrastive*; *negative-free* and *negative-sample-free* are broader still (they also cover [[non_contrastive_ssl]] distillation methods). VICReg additionally names the failure mode *informational collapse*, distinct from complete and dimensional collapse. These labels are largely interchangeable in practice but signal different theoretical lenses.

## References

{2021_Ermolov} 2021_Ermolov_Whitening_for_Self-Supervised_Representation_Learning
{2021_Hua} 2021_Hua_On_Feature_Decorrelation_in_Self-Supervised_Learning
{2021_Li_HSIC} 2021_Li_Self-Supervised_Learning_with_Kernel_Dependence_Maximization
{2021_Tsai} 2021_Tsai_A_Note_on_Connecting_Barlow_Twins_with_Negative-Sample-Free_Contrastive_Learning
{2021_Zbontar} 2021_Zbontar_Barlow_Twins_Self-Supervised_Learning_via_Redundancy_Reduction
{2022_Balestriero} 2022_Balestriero_Contrastive_and_Non-Contrastive_Self-Supervised_Learning_Recover_Global_and_Local_Spectral_Embedding_Methods
{2022_Bardes} 2022_Bardes_VICReg_Variance-Invariance-Covariance_Regularization_for_Self-Supervised_Learning
{2022_Liu_Decorrelation} 2022_Liu_Bridging_the_Gap_from_Asymmetry_Tricks_to_Decorrelation_Principles_in_Non-Contrastive_Self-Supervised_Learning
{2022_Liu_MEC} 2022_Liu_Self-Supervised_Learning_via_Maximum_Entropy_Coding
{2023_Garrido} 2023_Garrido_On_the_Duality_between_Contrastive_and_Non-Contrastive_Self-Supervised_Learning
