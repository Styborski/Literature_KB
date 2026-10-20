---
title: "Self-Supervised Learning"
tags: ["self-supervised-learning", "representation-learning", "pretraining", "unsupervised-learning", "transfer-learning"]
last_updated: "2026/08/21"
---

## Summary

Self-supervised learning (SSL) trains representations on unlabeled data by manufacturing supervision from the data itself — either predicting hand-designed pretext targets or enforcing invariance/prediction across augmented views — so the resulting encoder transfers to downstream tasks. It removes the labeling bottleneck and underlies modern visual and multimodal pretraining. This node is the hub for the SSL subgraph: it tracks the field's evolution from pretext tasks to joint-embedding and masked-modeling methods, organizes the main method families ([[contrastive_ssl]], [[non_contrastive_ssl]], [[feature_decorrelation_ssl]], [[cluster_based_ssl]], [[teacher_student_ssl]], [[multimodal_ssl]]), and collects the cross-cutting themes — collapse, augmentation, evaluation, and unifying theory — that recur across them.

## Related Concepts
[[contrastive_ssl]]
[[non_contrastive_ssl]]
[[feature_decorrelation_ssl]]
[[cluster_based_ssl]]
[[teacher_student_ssl]]
[[multimodal_ssl]]
[[pretext_tasks]]
[[masked_image_modeling]]
[[joint_embedding_predictive_architecture]]
[[image_augmentation_methods]]
[[dimensional_collapse]]
[[representation_evaluation]]
[[transfer_learning]]
[[vision_transformers]]
[[mutual_information_maximization]]
[[spectral_methods_in_ssl]]
[[adversarial_examples]]
[[variational_autoencoders]]
[[diffusion_model_semantic_spaces]]

## Open Questions

The field lacks a single accepted account of *why* SSL representations transfer. Generalization bounds exist under conditional-independence or multi-view-redundancy assumptions {2019_Arora}{2021_Lee}{2021_Tosh}, but these assumptions are disputed by augmentation-graph accounts {2022_HaoChen}, and some argue any adequate theory must encode function-class inductive biases (Saunshi et al., not consulted this session). Unifying frameworks — spectral embedding {2022_Balestriero}, contrastive/non-contrastive duality {2023_Garrido}, the I-Con KL objective {2025_Alshammari} — suggest the families are facets of one principle, but whether that abstraction is predictive or merely descriptive is unclear. Whether invariance or equivariance is the better organizing goal is newly contested {2024_Wang_ESSL}. Evaluation is unsettled: linear-probe, kNN, and fine-tuning rank methods differently {2021_He}.

## Synthesis

### What SSL is, and why it matters

SSL learns general-purpose representations without human labels by defining a supervised-style objective whose targets are derived from the input. Its promise is to convert abundant unlabeled data into transferable features, and it now provides the pretraining backbone for vision and vision-language systems. It sits within the broader unsupervised/semi-supervised landscape {2021_Qi}, but is distinguished by learning representations rather than densities or clusters for their own sake.

### Historical evolution

The field passed through recognizable phases. **Pretext tasks (covariant era):** early methods trained encoders to solve hand-designed puzzles — predicting spatial context (Doersch et al., not consulted this session), inpainting missing regions {2016_Pathak}, plus rotation and jigsaw. These learn features that *co-vary* with the transformation. **The invariance shift:** PIRL explicitly reframed the goal as *invariance* to transformations rather than covariance {2019_Misra}, and DeepCluster concurrently learned invariant features via clustering {2019_Caron}. **The joint-embedding explosion (2020–2021):** invariance-based methods proliferated into distinct families — contrastive (SimCLR {2020_Chen_SimCLR}, MoCo {2020_He}), distillation (BYOL {2020_Grill}), clustering (SwAV, via [[cluster_based_ssl]]), decorrelation (VICReg {2022_Bardes}), and teacher-student ViT (DINO {2021_Caron_DINO}). **Masked image modeling:** with ViTs, BERT-style masked prediction returned reconstruction to prominence — MAE {2021_He}, BEiT {2022_Bao}, SimMIM {2021_Xie}, and the distillation/MIM hybrid iBOT {2022_Zhou_iBOT}. **Predictive architectures:** methods that predict in latent space rather than pixel/token space, e.g. I-JEPA {2023_Assran} and data2vec, close the loop back toward joint embeddings. See [[pretext_tasks]], [[masked_image_modeling]], [[joint_embedding_predictive_architecture]].

### Method families and architecture taxonomy

Two orthogonal taxonomies organize the field. By *objective*, the VICReg paper's four-way split is widely reused — contrastive, clustering, distillation, and information-maximization {2022_Bardes} — corresponding to [[contrastive_ssl]], [[cluster_based_ssl]], [[teacher_student_ssl]], and [[feature_decorrelation_ssl]], with [[multimodal_ssl]] (e.g. CLIP {2021_Radford}) cutting across them; [[non_contrastive_ssl]] names the negative-free superset (distillation + decorrelation + clustering) or, narrowly, the BYOL/SimSiam branch. By *architecture*, I-JEPA distinguishes Joint-Embedding Architectures (invariance), Generative Architectures (reconstruction, e.g. MAE), and Joint-Embedding Predictive Architectures (latent prediction) {2023_Assran}. The boundaries are soft: SwAV "contrasts cluster assignments," DINO's centering/sharpening can be swapped for SwAV's Sinkhorn, and BYOL is claimed by both the non-contrastive and teacher-student literatures.

### Collapse: the unifying hazard

Any method that learns only from positive pairs risks *collapse* — mapping all inputs to one point (complete/uniform collapse) or to a low-rank subspace (dimensional/partial collapse). Each negative-free family has a characteristic anti-collapse device: contrastive negatives (which prevent complete but not dimensional collapse), decorrelation/whitening and variance floors ([[feature_decorrelation_ssl]]), balanced cluster assignment ([[cluster_based_ssl]]), and predictor + stop-gradient + momentum asymmetry ([[non_contrastive_ssl]], [[teacher_student_ssl]]). A recurring theoretical result is that these devices are interconvertible — asymmetry implicitly performs decorrelation, and all act as spectral filters that keep the effective rank high. See [[dimensional_collapse]].

### Augmentation, invariance, and equivariance

For joint-embedding methods, augmentation is the engine: it defines the positive pairs and thus the learned invariances. SimCLR showed composition of augmentations (especially color distortion, to defeat the color-histogram shortcut) is essential {2020_Chen_SimCLR}; theory shows augmentation decouples signal from per-view noise {2021_Wen}, and there is a mutual-information "sweet spot" between views {2020_Tian}. This dependence is a double-edged sword: strong augmentation can destroy task-relevant information (e.g. color under grayscale), which motivates a reconsideration of *equivariance*-based SSL — rotation/jigsaw prediction — reframed via an "explaining-away" information-theoretic account that also recasts instance discrimination and masked modeling as fine-grained equivariance tasks {2024_Wang_ESSL}. See [[image_augmentation_methods]].

### Unifying theories

A striking recent theme is that superficially different methods reduce to shared mathematics. Contrastive and non-contrastive (decorrelation) losses are dual — sample-contrastive vs dimension-contrastive, equal up to normalization {2023_Garrido}; contrastive, clustering, and decorrelation objectives recover global/local spectral embeddings {2022_Balestriero}; contrastive learning is spectral clustering on an augmentation/similarity graph {2022_HaoChen}{2023_Tan}; and I-Con expresses clustering, dimensionality reduction, and contrastive learning as one KL-divergence objective over neighborhood distributions {2025_Alshammari}. See [[spectral_methods_in_ssl]], [[mutual_information_maximization]].

### Why SSL transfers, and how it is evaluated

Theory relates downstream performance to the pretraining objective under assumptions of varying strength: contrastive risk bounds via latent classes {2019_Arora}, reconstruction-SSL bounds via conditional independence {2021_Lee}, and multi-view-redundancy guarantees {2021_Tosh}. Evaluation typically uses linear probing, kNN, full fine-tuning, and transfer to detection/segmentation, but these disagree — MAE excels at fine-tuning yet lags at linear probing, reflecting nonlinear but not linearly-separable features {2021_He}; representations are conventionally taken *before* the projection head, which the [[projection_head]] discards information into for uniformity {2023_Ma}. See [[representation_evaluation]], [[transfer_learning]].

### SSL versus supervised learning

On CNN backbones, SSL and supervised models behave surprisingly alike — and both unlike humans: they share a texture bias and pick one arbitrary class under heavy noise, with SimCLR closest to human robustness among 2020-era methods {2020_Geirhos}. This equivalence is an ImageNet/ResNet-era finding and may not hold for ViT-scale masked or distillation methods, which is itself an open direction.

## References

{2016_Pathak} 2016_Pathak_Context_Encoders_Feature_Learning_by_Inpainting
{2019_Arora} 2019_Arora_A_Theoretical_Analysis_of_Contrastive_Unsupervised_Representation_Learning
{2019_Caron} 2019_Caron_Deep_Clustering_for_Unsupervised_Learning_of_Visual_Features
{2019_Misra} 2019_Misra_Self-Supervised_Learning_of_Pretext-Invariant_Representations
{2020_Chen_SimCLR} 2020_Chen_A_Simple_Framework_for_Contrastive_Learning_of_Visual_Representations
{2020_Geirhos} 2020_Geirhos_On_the_Surprising_Similarities_between_Supervised_and_Self-Supervised_Models
{2020_Grill} 2020_Grill_Bootstrap_Your_Own_Latent_A_New_Approach_to_Self-Supervised_Learning
{2020_He} 2020_He_Momentum_Contrast_for_Unsupervised_Visual_Representation_Learning
{2020_Tian} 2020_Tian_What_Makes_for_Good_Views_for_Contrastive_Learning
{2021_Caron_DINO} 2021_Caron_Emerging_Properties_in_Self-Supervised_Vision_Transformers
{2021_He} 2021_He_Masked_Autoencoders_Are_Scalable_Vision_Learners
{2021_Lee} 2021_Lee_Predicting_What_You_Already_Know_Helps_Provable_Self-Supervised_Learning
{2021_Qi} 2021_Qi_Small_Data_Challenges_in_Big_Data_Era_A_Survey_of_Recent_Progress_on_Unsupervised_and_Semi-Supervised_Methods
{2021_Radford} 2021_Radford_Learning_Transferable_Visual_Models_From_Natural_Language_Supervision
{2021_Tosh} 2021_Tosh_Contrastive_Learning_Multi-View_Redundancy_and_Linear_Models
{2021_Wen} 2021_Wen_Toward_Understanding_the_Feature_Learning_Process_of_Self-supervised_Contrastive_Learning
{2021_Xie} 2021_Xie_SimMIM_A_Simple_Framework_for_Masked_Image_Modeling
{2022_Balestriero} 2022_Balestriero_Contrastive_and_Non-Contrastive_Self-Supervised_Learning_Recover_Global_and_Local_Spectral_Embedding_Methods
{2022_Bao} 2022_Bao_BEiT_BERT_Pre-Training_of_Image_Transformers
{2022_Bardes} 2022_Bardes_VICReg_Variance-Invariance-Covariance_Regularization_for_Self-Supervised_Learning
{2022_HaoChen} 2022_HaoChen_Provable_Guarantees_for_Self-Supervised_Deep_Learning_with_Spectral_Contrastive_Loss
{2022_Zhou_iBOT} 2022_Zhou_iBOT_Image_BERT_Pre-training_with_Online_Tokenizer
{2023_Assran} 2023_Assran_Self-Supervised_Learning_from_Images_with_a_Joint-Embedding_Predictive_Architecture
{2023_Garrido} 2023_Garrido_On_the_Duality_between_Contrastive_and_Non-Contrastive_Self-Supervised_Learning
{2023_Ma} 2023_Ma_Deciphering_the_Projection_Head_Representation_Evaluation_Self-Supervised_Learning
{2023_Tan} 2023_Tan_Contrastive_Learning_is_Spectral_Clustering_on_Similarity_Graph
{2024_Wang_ESSL} 2024_Wang_Understanding_the_Role_of_Equivariance_in_Self-Supervised_Learning
{2025_Alshammari} 2025_Alshammari_I-Con_A_Unifying_Framework_for_Representation_Learning
