---
title: "Cluster-Based Self-Supervised Learning"
tags: ["self-supervised-learning", "deep-clustering", "prototypes", "optimal-transport", "swav", "pseudo-labels"]
last_updated: "2026/08/21"
---

## Summary

Cluster-based SSL derives its supervisory signal from cluster assignments or prototypes: the encoder's embeddings are grouped (by k-means, optimal transport, or online prototypes), the assignments serve as pseudo-labels, and the encoder is trained to predict them, iterating between clustering and representation learning. Because samples in the same cluster share a target, the signal captures semantics *across* instances rather than treating each image as its own class — a contrast with [[contrastive_ssl]] instance discrimination. DeepCluster, SeLa, and SwAV are the exemplars. Collapse (all inputs to one cluster) is the central hazard, countered by balanced-assignment constraints rather than negatives, which ties the family to optimal transport and to [[teacher_student_ssl]].

## Related Concepts
[[self_supervised_learning]]
[[contrastive_ssl]]
[[non_contrastive_ssl]]
[[teacher_student_ssl]]
[[multimodal_ssl]]
[[optimal_transport]]
[[mutual_information_maximization]]
[[dimensional_collapse]]
[[image_augmentation_methods]]
[[spectral_methods_in_ssl]]

## Open Questions

The relationship between clustering, contrastive, and spectral objectives is being actively unified — I-Con casts k-means, SNE, and InfoNCE as one KL-divergence objective {2025_Alshammari}, and clustering appears as a limiting/related case in spectral accounts of [[contrastive_ssl]] — but a clean statement of when a prototype/cluster target is preferable to instance or decorrelation targets is missing. How many prototypes/clusters to use, and how sensitive results are to that choice, remains heuristic {2020_Asano}{2021_Caron_SwAV}. Whether the balanced-equipartition assumption (uniform cluster sizes) is appropriate for naturally imbalanced data is unresolved. The degree of "semantic reshuffling" during training — DeepCluster reassigns a large fraction of clusters even at convergence {2019_Caron} — and its effect on representation stability are not well characterized.

## Synthesis

### From offline k-means to optimal-transport self-labelling

DeepCluster {2019_Caron} established the template: encode images, run k-means on the features to obtain pseudo-labels, train the encoder with a standard classification loss against those labels, and repeat. The pseudo-"classes" are cluster centroids living in feature space, so the encoder learns to distill each image's core feature to classify it. Collapse is a constant threat, mitigated with empty-cluster reinitialization and class-balanced sampling. SeLa {2020_Asano} reframed the assignment step as *optimal transport*: minimize the cross-entropy of assignments subject to an equipartition constraint (each cluster gets an equal share of samples), solved efficiently with the Sinkhorn–Knopp algorithm (the entropic-OT accelerator of Cuturi, and the Sinkhorn–Knopp matrix-scaling procedure — both named but not consulted directly here). This makes explicit that minimizing the assignment cross-entropy under the balance constraint *maximizes mutual information between labels and data*, and that the equipartition constraint is precisely what prevents the trivial single-cluster solution; SeLa also showed self-labelling beats k-means and benefits from multiple clustering heads. See [[optimal_transport]], [[mutual_information_maximization]].

### Online clustering and prototypes

SwAV {2021_Caron_SwAV} moved clustering *online*, inside each batch: embeddings are matched to a learned bank of prototypes to get soft codes (via a within-batch Sinkhorn), and a "swapped" prediction task trains one view to predict the other view's code. This "contrasts cluster assignments" rather than instances, positioning SwAV at the boundary with [[contrastive_ssl]]; it also introduced *multi-crop* (several low-resolution crops plus two high-resolution ones), which subsequently improved many SSL methods including DeepCluster itself. Prototypical Contrastive Learning {2021_Li_ProtoCL} takes an EM view, using cluster centroids as prototypes inside a ProtoNCE loss (a contrastive term against centroids added to instance InfoNCE), with a per-cluster concentration term that moderates the pull toward tight clusters and thereby balances cluster sizes; it reports higher mutual information with ImageNet labels than plain InfoNCE. OBoW {2021_Gidaris} generates an online *bag-of-visual-words* target from a vocabulary of feature centroids, with a teacher (EMA of the student) producing the BoW targets a student predicts from augmented crops — a hybrid of clustering and [[teacher_student_ssl]]. A mutual-information-regularized-assignment method (MIRA) also belongs here but was not consulted (unread).

### Collapse avoidance without negatives

The defining move of the family is replacing negative samples with *assignment-balancing* to prevent collapse: DeepCluster's reinitialization and balanced sampling {2019_Caron}, and the equipartition/marginal constraints enforced by Sinkhorn in SeLa and SwAV {2020_Asano}{2021_Caron_SwAV}. This is mechanistically distinct from the predictor/stop-gradient route of [[non_contrastive_ssl]] and the covariance regularization of [[feature_decorrelation_ssl]], though all three are anti-collapse strategies. The connection is made explicit by DINO, whose ablations show that its momentum-teacher softmax with centering+sharpening can be replaced by Sinkhorn–Knopp to recover the SwAV mechanism {2021_Caron_DINO} — placing DINO and SwAV on a continuum (see [[teacher_student_ssl]]).

### Cross-modal clustering

Clustering extends naturally to multiple modalities by using one modality's clusters as labels for another. XDC {2020_Alwassel} branches from DeepCluster to use cross-modal audio↔video cluster assignments as mutual supervision; SeLaVi {2021_Asano} applies SeLa-style optimal-transport self-labelling to video, treating audio and visual streams as augmentations of one signal; Deep Multimodal Clustering {2019_Hu} clusters audio and visual encodings separately with a differentiable soft-assignment and a triplet objective across modality centroids. These sit at the intersection with [[multimodal_ssl]].

### Unification and terminology

Cluster-based SSL is increasingly folded into broader frameworks: I-Con expresses k-means, contrastive learning, and dimensionality reduction as one KL-divergence objective over neighborhood distributions, and its best method is a debiased InfoNCE clustering that reuses the k-means kernel {2025_Alshammari}; clustering also appears in spectral accounts of contrastive learning (see [[spectral_methods_in_ssl]]). Terminology varies: *cluster assignments*, *codes*, *prototypes*, *pseudo-labels*, and *self-labelling* refer to closely related constructs. Note a genuine acronym collision in this corpus: "PCL" denotes both *Prototypical* Contrastive Learning {2021_Li_ProtoCL} (this family) and *Predictive* Contrastive Learning {2021_Li_FeatSuppress} (a reconstruction-augmented contrastive method under [[contrastive_ssl]]).

## References

{2019_Caron} 2019_Caron_Deep_Clustering_for_Unsupervised_Learning_of_Visual_Features
{2019_Hu} 2019_Hu_Deep_Multimodal_Clustering_for_Unsupervised_Audiovisual_Learning
{2020_Alwassel} 2020_Alwassel_Self-Supervised_Learning_by_Cross-Modal_Audio-Video_Clustering
{2020_Asano} 2020_Asano_Self-Labelling_via_Simultaneous_Clustering_and_Representation_Learning
{2021_Asano} 2021_Asano_Labelling_Unlabelled_Videos_from_Scratch_with_Multi-Modal_Self-Supervision
{2021_Caron_DINO} 2021_Caron_Emerging_Properties_in_Self-Supervised_Vision_Transformers
{2021_Caron_SwAV} 2021_Caron_Unsupervised_Learning_of_Visual_Features_by_Contrasting_Cluster_Assignments
{2021_Gidaris} 2021_Gidaris_OBoW_Online_Bag-of-Visual-Words_Generation_for_Self-Supervised_Learning
{2021_Li_FeatSuppress} 2021_Li_Addressing_Feature_Suppression_in_Unsupervised_Visual_Representations
{2021_Li_ProtoCL} 2021_Li_Prototypical_Contrastive_Learning_of_Unsupervised_Representations
{2025_Alshammari} 2025_Alshammari_I-Con_A_Unifying_Framework_for_Representation_Learning
