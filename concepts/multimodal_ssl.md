---
title: "Multimodal Self-Supervised Learning"
tags: ["self-supervised-learning", "multimodal", "vision-language", "clip", "cross-modal", "modality-gap"]
last_updated: "2026/08/21"
---

## Summary

Multimodal SSL learns representations from the natural correspondence between paired modalities — image–text, audio–video — using one modality as free supervision for another, without human labels. CLIP is the defining method: contrastive alignment of image and text embeddings over web-scale pairs, yielding strong zero-shot transfer. The paradigm reuses the machinery of [[contrastive_ssl]] (cross-modal InfoNCE) and [[cluster_based_ssl]] (cross-modal clustering) and inherits their contrastive-vs-non-contrastive debate. It also raises phenomena unique to the multimodal setting — most notably the *modality gap* — and theory on why multiple modalities help.

## Related Concepts
[[self_supervised_learning]]
[[contrastive_ssl]]
[[cluster_based_ssl]]
[[teacher_student_ssl]]
[[non_contrastive_ssl]]
[[modality_gap]]
[[mutual_information_maximization]]
[[image_augmentation_methods]]
[[vision_transformers]]

## Open Questions

Whether contrastive or non-contrastive objectives are better for multimodal learning is unsettled: nCLIP wins on dense/semantic tasks and small pretraining data but CLIP scales better, motivating blends {2022_Zhou_nCLIP}. The modality gap is a robust but poorly-understood phenomenon — it arises from an initialization "cone effect" plus contrastive optimization, a nonzero gap is empirically optimal, and its size depends on temperature, yet its causal role in downstream performance is unclear {2022_Liang}. Theory guarantees more modalities help only above a sample-size threshold, below which the function-complexity penalty dominates {2021_Huang}, but tight practical predictions are lacking. How to exploit *unpaired* multimodal data, and whether "modality-agnostic" frameworks truly escape modality-specific inductive biases {2022_Baevski}, remain open.

## Synthesis

### CLIP and cross-modal contrastive learning

CLIP {2021_Radford} trains an image encoder and a text encoder so that matched image–text pairs have high cosine similarity and mismatched pairs low, using a symmetric cross-entropy over the batch's $N \times N$ similarity matrix — exactly InfoNCE applied across modalities, with matched pairs on the diagonal as positives and all off-diagonal pairs as negatives. Trained on 400M web pairs with a bag-of-words text target for efficiency, it enables zero-shot classification by comparing an image embedding to embeddings of candidate captions, with ViT encoders outperforming ResNet. CLIP is thus a cross-modal instance of [[contrastive_ssl]], and inherits its properties (temperature sensitivity, reliance on large negative sets/batches, spectral-clustering interpretation — the latter extended to CLIP explicitly in {2023_Tan}, cross-linked from [[contrastive_ssl]]). In the classic multimodal taxonomy {2017_Baltrusaitis}, CLIP realizes a *coordinated representation* (each modality encoded separately, then aligned) via *co-learning*, as opposed to a *joint representation* that fuses modalities in one encoder.

### Cross-modal clustering

Before and alongside CLIP, audio–video self-supervision used clustering to generate cross-modal targets: XDC {2020_Alwassel} uses one modality's cluster assignments as labels for the other; SeLaVi {2021_Asano} applies optimal-transport self-labelling treating audio and video as two views of one signal; Deep Multimodal Clustering {2019_Hu} clusters each modality's encodings with a differentiable soft assignment and a cross-modal triplet objective. These overlap fully with [[cluster_based_ssl]] and show the same collapse-avoidance-via-balanced-assignment logic transferred to the multimodal case.

### Non-contrastive multimodal learning

Whether multimodal SSL *must* be contrastive is questioned by nCLIP {2022_Zhou_nCLIP}, a non-contrastive CLIP that replaces the contrastive loss with softmax cross-entropy between paired image/text distributions plus two entropy regularizers (sharp per-sample distributions, smooth mean distribution) to avoid collapse — the same teacher-student/clustering anti-collapse logic used by DINO and SwAV. Alone nCLIP underperforms CLIP on zero-shot classification but beats it on segmentation and semantic-textual-similarity and on smaller pretraining sets; the blended xCLIP (weighted CLIP + nCLIP) dominates both, echoing the joint contrastive+non-contrastive remedies seen in unimodal SSL. This ties the multimodal debate directly to [[non_contrastive_ssl]] and [[teacher_student_ssl]].

### The modality gap and why modalities help

A distinctive multimodal phenomenon is the *modality gap*: in CLIP, image embeddings and text embeddings occupy separate narrow "cones" on the hypersphere {2022_Liang}. This originates from a general "cone effect" (even randomly-initialized deep encoders map data into a small cone; different initializations give different cones) amplified by contrastive optimization; strikingly, a nonzero gap is empirically *optimal* for the loss, its size decreases with temperature, and deeper networks shrink it. On the theory side, {2021_Huang} proves that using more modalities yields better latent-representation quality and test accuracy — but only once sample size is large enough to overcome the greater function-class complexity, so multimodal can *underperform* unimodal in low-data regimes. See [[modality_gap]].

### Scope and coverage note

Multimodal SSL blends into [[teacher_student_ssl]] via modality-agnostic latent-prediction methods like data2vec, whose universality claim is qualified by its continued use of modality-specific masking and feature extractors {2022_Baevski}. This node draws on the vision–language and audio–video literature; further library works specifically on *unpaired* multimodal data and its guarantees (e.g., Nakada et al. 2023; Liang et al. 2023) were not consulted this session and would strengthen the treatment of the unpaired-data question raised in Open Questions.

## References

{2017_Baltrusaitis} 2017_Baltrusaitis_Multimodal_Machine_Learning_A_Survey_and_Taxonomy
{2019_Hu} 2019_Hu_Deep_Multimodal_Clustering_for_Unsupervised_Audiovisual_Learning
{2020_Alwassel} 2020_Alwassel_Self-Supervised_Learning_by_Cross-Modal_Audio-Video_Clustering
{2021_Asano} 2021_Asano_Labelling_Unlabelled_Videos_from_Scratch_with_Multi-Modal_Self-Supervision
{2021_Huang} 2021_Huang_What_Makes_Multi-modal_Learning_Better_than_Single_Provably
{2021_Radford} 2021_Radford_Learning_Transferable_Visual_Models_From_Natural_Language_Supervision
{2022_Baevski} 2022_Baevski_data2vec_A_General_Framework_for_Self-Supervised_Learning_in_Speech_Vision_and_Language
{2022_Liang} 2022_Liang_Mind_the_Gap_Understanding_the_Modality_Gap_in_Multi-modal_Contrastive_Representation_Learning
{2022_Zhou_nCLIP} 2022_Zhou_Non-Contrastive_Learning_Meets_Language-Image_Pre-Training
{2023_Tan} 2023_Tan_Contrastive_Learning_is_Spectral_Clustering_on_Similarity_Graph
